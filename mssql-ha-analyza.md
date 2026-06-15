# MS SQL Server – Analýza HA řešení pro read-only repliku

> **Kontext:** MS SQL Server 2022 Standard Edition (16.0.4245.2)  
> **Požadavek:** Sekundární databáze online, readonly, automatická propagace změn schématu, bez dopadu na primární DB.

---

## Porovnání řešení

| Vlastnost | Always On FCI | Always On AG (Basic) | Log Shipping (STANDBY) |
|---|---|---|---|
| **Dostupná ve Standard** | ✅ Ano | ✅ Ano (Basic AG) | ✅ Ano |
| **Sekundární DB readonly** | ❌ **NE** | ❌ **NE (Basic AG)** | ✅ Ano |
| **Automatická propagace DDL** | ✅ Ano | ✅ Ano | ✅ Ano |
| **Latence** | ms (failover) | ms | minuty |
| **Primární DB neovlivněna** | ✅ Ano | ✅ Ano | ✅ Ano |
| **Licenční náklady navíc** | ⚠️ Windows Cluster + oba uzly | ⚠️ Oba uzly Standard | ⚠️ Druhý server Standard |
| **Sdílené úložiště (SAN/NAS)** | ✅ Vyžaduje | ❌ Nevyžaduje | ❌ Nevyžaduje |
| **Složitost nasazení** | Vysoká | Střední | Nízká |

---

## Always On Failover Cluster Instance (FCI) – proč NEVYHOVUJE

### Co FCI je

Always On FCI je **clustering na úrovni instance SQL Serveru**, nikoliv databáze.  
Oba uzly clusteru sdílejí **jedno společné úložiště** (SAN, iSCSI, Azure Shared Disk).  
V daný okamžik je aktivní vždy jen **jeden uzel** – druhý je pasivní.

### Klíčový problém: sekundární uzel NENÍ pro čtení

```
[Node 1 – aktivní]  ←→  [Shared Storage]  ←→  [Node 2 – pasivní]
     ↑ SQL běží zde                                  ↑ SQL neběží vůbec
```

- Pasivní uzel FCI **nespouští SQL Server** – čeká pouze na failover
- **Nelze z něj číst**, nelze na něj posílat reporting dotazy
- Po failoveru se role obrátí, ale opět pouze jeden uzel je aktivní

### Závěr pro FCI

> ❌ **Always On FCI nevyhovuje požadavku na read-only repliku pro reporting.**  
> FCI řeší pouze vysokou dostupnost (HA) formou failoveru, nikoliv čitelnou kopii.

---

## Always On Availability Groups (AG) – Basic vs Full

### Basic AG (Standard Edition)

SQL Server 2016+ Standard Edition podporuje **Basic Availability Groups** s těmito omezeními:

| Omezení Basic AG | Popis |
|---|---|
| Max. 2 repliky | 1 primární + 1 sekundární |
| Sekundární replica readonly | ❌ **Readable secondary NENÍ podporována** |
| Max. 1 databáze na AG | Nelze sdružit více DB |
| Bez distribuovaných AG | Pouze lokální cluster |

> ❌ **Basic AG na Standard Edition neumožňuje čtení ze sekundární repliky.**

### Full AG (Enterprise Edition)

- ✅ Readable secondary je plně podporována
- ✅ Lze nastavit `READ_ONLY_ROUTING`
- ✅ Latence v řádu milisekund (synchronní) nebo sekund (asynchronní)
- ❌ **Vyžaduje Enterprise Edition** – výrazně vyšší licenční náklady

---

## Doporučené řešení: Log Shipping v STANDBY režimu

Jediné řešení dostupné na **Standard Edition**, které splňuje všechny požadavky:

### Proč Log Shipping

| Požadavek | Log Shipping STANDBY |
|---|---|
| Primární DB neovlivněna | ✅ Záloha logu je offline operace |
| Sekundární readonly | ✅ STANDBY = DB je čitelná mezi restore |
| Automatická propagace DDL | ✅ Log obsahuje vše byte-per-byte |
| Latence v minutách | ✅ Konfigurovatelné (1–5 min) |
| Standard Edition | ✅ Plně podporováno |

### Schéma fungování

```
[Primary DB – FULL Recovery]
        │
        │  každé 1–2 min: T-Log backup
        ▼
[Sdílená složka \\server\logship\]
        │
        │  každé 1–2 min: kopírování + restore
        ▼
[Secondary DB – STANDBY mode]
        │
        ├─ mezi restore: ✅ READ-ONLY dotazy
        └─ během restore: ⚠️ krátce nedostupná (sekundy)
```

### Postup nastavení (shrnutí)

1. Nastav Recovery Model primární DB na `FULL`
2. Vytvoř sdílenou zálohovací složku přístupnou oběma serverům
3. Proveď Full Backup a obnov sekundární DB s `STANDBY = 'cesta\undo.bak'`
4. V SSMS → Properties primární DB → **Transaction Log Shipping**
   - Zapni primární roli
   - Nastav backup job (interval 1–2 min)
   - Přidej secondary server, zvol **Restore Mode: STANDBY**
   - Nastav copy + restore job (interval 1–2 min)
5. Ověř pomocí `msdb.dbo.log_shipping_monitor_secondary`

### Omezení Log Shipping

| Omezení | Dopad |
|---|---|
| Krátká nedostupnost při restore (sekundy) | Reporting aplikace potřebuje retry logiku |
| Latence min. 1–5 minut | Přijatelné pro reporting |
| Ruční failover (není automatický) | Log Shipping není HA – pouze DR/reporting |
| Undo soubor musí mít místo | Monitorovat velikost souboru |

---

## Souhrn doporučení

```
Standard Edition + požadavek na readonly repliku pro reporting
                         │
                         ▼
              ✅ Log Shipping (STANDBY mode)
                         │
              Pokud potřebuješ automatický failover:
                         │
                         ▼
              ❌ Basic AG (readonly secondary není k dispozici)
                         │
              Pokud potřebuješ readonly secondary + HA:
                         │
                         ▼
              → Upgrade na Enterprise Edition (Always On AG full)
                nebo Azure SQL (Hyperscale / Business Critical)
```

> **Doporučení:** Pro SQL Server 2022 Standard s požadavkem na čitelnou kopii pro reporting  
> je **Log Shipping v STANDBY režimu** jedinou nativní a ekonomicky dostupnou volbou.
