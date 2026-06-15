# MS SQL Server – Log Shipping: read-only replika pro reporting

> **Prostředí:** MS SQL Server 2022 Standard Edition (16.0.4245.2) na primárním i sekundárním serveru  
> **Cíl:** Sekundární databáze readonly s minutovým zpožděním, automatická propagace DDL i DML, bez dopadu na primární server.

---

## Přehled replikovaných databází

Celkem **13 databází**, celková velikost **~35 GB** – viz tabulka:

| Databáze | Velikost (MB) | Poznámka |
|---|---|---|
| ReportServer | 14 608 | SSRS katalog |
| DN.LOG | 7 784 | |
| DN.ITM | 7 671 | |
| DN.GDA | 1 053 | |
| DN.OPR | 946 | |
| DN.CFG | 646 | |
| DN.EVL | 528 | |
| DN.STK | 364 | |
| NG.Portal | 362 | |
| DN.TMP | 298 | Zvážit, zda je reporting na TMP potřeba |
| DN.TRX | 290 | |
| ReportServerTempDB | 208 | SSRS temp – lze vynechat, recreatuje se automaticky |
| DN.ITF | 144 | |

> ⚠️ **Poznámka k Express edici:** SQL Server Express má limit **10 GB na databázi**.  
> Z výše uvedených databází by na Express nevešla žádná z velkých DB (ReportServer 14 GB, DN.LOG 7,8 GB, DN.ITM 7,7 GB, …).  
> **Express edice je pro tento scénář nevhodná** – sekundární server musí být Standard nebo vyšší.

---

## Postup nastavení Log Shipping (Standard → Standard)

Log Shipping přehrává transakční log byte-per-byte, čímž zajišťuje automatickou propagaci veškerých změn včetně DDL (ALTER TABLE, CREATE TABLE, DROP TABLE, …). Primární server není při záloze logu nijak omezen.

### Schéma fungování

```
[Primary Server – Standard]
        │
        │  každé 1–2 min: T-Log backup → sdílená složka
        ▼
[\\PrimaryServer\LogShip\]
        │
        │  každé 1–2 min: kopírování + restore
        ▼
[Secondary Server – Standard – STANDBY mode]
        │
        ├─ mezi restore operacemi: ✅ READ-ONLY dotazy (reporting)
        └─ během restore operace: ⚠️ DB krátce nedostupná (sekundy)
```

---

### Krok 1 – Příprava primárního serveru

1. Ověř Recovery Model pro každou replikovanou databázi – musí být `FULL`:
   ```sql
   -- Spusť pro každou DB zvlášť (nebo hromadně):
   ALTER DATABASE [DN.LOG]  SET RECOVERY FULL;
   ALTER DATABASE [DN.ITM]  SET RECOVERY FULL;
   ALTER DATABASE [DN.GDA]  SET RECOVERY FULL;
   -- … opakuj pro všechny DB
   ```

2. Vytvoř **sdílenou zálohovací složku** přístupnou z obou serverů:
   - Doporučená cesta: `\\PrimaryServer\LogShip\`
   - SQL Server service account primárního serveru potřebuje **write** přístup
   - SQL Server service account sekundárního serveru potřebuje **read** přístup

3. Ověř, že **SQL Server Agent je spuštěn** na obou serverech (nutné pro scheduling jobů).

---

### Krok 2 – Inicializace sekundárních databází

Pro každou databázi:

1. Proveď **Full Backup** na primárním serveru:
   ```sql
   BACKUP DATABASE [DN.LOG]
   TO DISK = '\\PrimaryServer\LogShip\DN.LOG_init.bak'
   WITH COMPRESSION, STATS = 10;
   ```

2. Přesuň zálohu na sekundární server (nebo přistupuj přes UNC cestu).

3. Obnov databázi na sekundárním serveru v **STANDBY** režimu:
   ```sql
   RESTORE DATABASE [DN.LOG]
   FROM DISK = 'D:\Restore\DN.LOG_init.bak'
   WITH
       STANDBY = 'D:\Standby\DN.LOG_undo.bak',
       MOVE 'DN.LOG'     TO 'D:\Data\DN.LOG.mdf',
       MOVE 'DN.LOG_log' TO 'D:\Log\DN.LOG_log.ldf',
       STATS = 10;
   ```
   > `STANDBY =` cesta k undo souboru – musí existovat adresář, soubor SQL Server vytvoří sám.  
   > Tento soubor **nesmaz** – používá se při každém restore k tomu, aby DB zůstala čitelná.

4. Opakuj pro každou databázi.

---

### Krok 3 – Konfigurace Log Shipping přes SSMS

Pro každou databázi zvlášť (nebo skriptem – viz Krok 4):

1. V SSMS klikni pravým na primární DB → **Properties → Transaction Log Shipping**
2. Zaškrtni **„Enable this as a primary database in a log shipping configuration"**
3. Záložka **Backup Settings:**
   - Backup folder: `\\PrimaryServer\LogShip\`
   - Backup file name prefix: název DB (např. `DN_LOG_`)
   - Schedule: každé **1 minutu** (nebo 2 minuty)
   - Retention: `1440` minut (24 h)
4. Klikni **Add…** pro přidání sekundárního serveru:
   - Secondary server instance: `SecondaryServer\Instance`
   - Secondary database: stejný název jako primární (např. `DN.LOG`)
   - Initialize: **„No, the secondary database is initialized"** ← protože jsme restore provedli ručně v Kroku 2
   - **Restore Mode: STANDBY** ← klíčové nastavení
   - Standby file: `D:\Standby\DN.LOG_undo.bak` ← stejná cesta jako při ručním restore
   - Copy schedule: každé **1 minutu**
   - Restore schedule: každé **1 minutu**
   - Disconnect users: **Yes** (přeruší aktivní reporting dotazy na dobu restore)
5. Klikni **OK** → SSMS vytvoří SQL Agent joby na obou serverech automaticky

---

### Krok 4 – Skript pro hromadné nastavení (volitelné)

Pro automatizaci lze použít systémovou uloženou proceduru místo GUI:

```sql
-- Příklad pro jednu DB – opakuj pro každou databázi
EXEC msdb.dbo.sp_add_log_shipping_primary_database
    @database = N'DN.LOG',
    @backup_directory = N'\\PrimaryServer\LogShip\',
    @backup_share = N'\\PrimaryServer\LogShip\',
    @backup_job_name = N'LSBackup_DN.LOG',
    @backup_retention_period = 1440,
    @backup_threshold = 60,
    @threshold_alert_enabled = 1,
    @history_retention_period = 1440;

EXEC msdb.dbo.sp_add_log_shipping_secondary_database
    @secondary_database = N'DN.LOG',
    @primary_server = N'PrimaryServer',
    @primary_database = N'DN.LOG',
    @restore_delay = 0,
    @restore_mode = 1,           -- 1 = STANDBY (0 = NORECOVERY)
    @disconnect_users = 1,
    @restore_threshold = 45,
    @threshold_alert_enabled = 1,
    @history_retention_period = 1440,
    @standby_file_name = N'D:\Standby\DN.LOG_undo.bak';
```

---

### Krok 5 – Ověření a monitoring

```sql
-- Stav na primárním serveru (čas poslední zálohy):
SELECT primary_database, last_backup_date, last_backup_file
FROM msdb.dbo.log_shipping_monitor_primary;

-- Stav na sekundárním serveru (čas posledního restore):
SELECT secondary_database, last_copied_date, last_restored_date, last_restored_latency
FROM msdb.dbo.log_shipping_monitor_secondary;
```

- `last_restored_latency` = aktuální zpoždění v minutách
- Nastavit **alert** v SQL Agent jobu, pokud latence překročí např. 15 minut

---

### Provozní omezení

| Omezení | Dopad |
|---|---|
| Krátká nedostupnost sekundární DB při restore (~1–5 s) | Reporting aplikace by měla mít retry logiku nebo tolerovat krátký výpadek |
| Latence dat 1–5 minut | Přijatelné pro reporting |
| Log Shipping není HA (ruční failover) | Sekundární DB slouží pouze pro reporting, ne jako záloha pro failover |
| Undo soubor pro každou DB (~velikost uncommitted transakcí) | Monitorovat místo na disku sekundárního serveru |
| Při výpadku síťové sdílené složky přestane kopírování | Monitoring SQL Agent jobů je nutný |

---

## Proč jiné metody replikace nejsou vhodné

| Metoda | Důvod nevhodnosti |
|---|---|
| **Always On FCI** | Pasivní uzel nespouští SQL Server – nelze z něj číst |
| **Always On AG Basic (Standard)** | Readable secondary explicitně nepodporována v Basic AG |
| **Always On AG Full** | Vyžaduje Enterprise Edition – výrazně vyšší náklady na licence |
| **Transactional Replication** | Nevhodné pro DDL – změny schématu je nutné přidávat do publikace ručně; nelze replikovat databáze SSRS a systémové tabulky snadno |
| **Snapshot Replication** | Latence v řádu hodin; při obnově snapshotu krátká nedostupnost |
| **Express Edition subscriber** | Limit 10 GB/DB – žádná z replikovaných DB se nevejde |
