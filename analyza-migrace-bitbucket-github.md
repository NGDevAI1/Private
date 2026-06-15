# Analýza migrace z Bitbucket / Atlassian na GitHub

Datum zpracování: 2026-06-15

## Shrnutí pro rozhodnutí

Přechod z Bitbucketu na GitHub dává největší smysl, pokud chcete sjednotit vývojářskou platformu kolem GitHubu, využít širší ekosystém integrací, GitHub Actions, GitHub Advanced Security, GitHub Copilot, Codespaces a standardní pull request workflow známé většině vývojářů. GitHub je dnes silná platforma nejen pro správu repozitářů, ale i pro CI/CD, bezpečnost, automatizaci, interní open-source model, code review, balíčky, dokumentaci vývoje a AI asistenci.

Migrace ale nemusí znamenat úplné opuštění Atlassianu. U firem, které mají Jira, timesheeting, Confluence, zákaznický support nebo schvalovací procesy hluboce navázané na Atlassian, je často nejlepší varianta:

- migrovat zdrojové kódy, pull requesty a CI/CD do GitHubu,
- ponechat Jira jako hlavní systém pro projektové řízení, timesheety a reporting,
- propojit GitHub s Jira pomocí oficiální integrace,
- až po stabilizaci zvážit, zda GitHub Issues / Projects dokážou nahradit část Jira agendy.

Doporučení: nezačínat „big bang“ migrací celé organizace. Nejprve udělat pilot na 1–3 reprezentativních repozitářích, ověřit migraci historie, pull requestů, pipeline, oprávnění, Jira vazeb a nákladů. Teprve potom připravit vlnovou migraci týmů.

## Výchozí situace

Předpokládaný současný stav:

- Atlassian účet / organizace.
- Bitbucket jako Git hosting.
- Jira jako hlavní evidence práce, issue tracking, sprinty a roadmapy.
- Timesheet nebo worklog řešení navázané na Jira.
- Možné další Atlassian nástroje: Confluence, Jira Service Management, Compass, Access, Marketplace aplikace.
- CI/CD pravděpodobně přes Bitbucket Pipelines, Jenkins, Bamboo nebo jiné externí nástroje.

## Co lze migrovat na GitHub

| Oblast | Lze migrovat? | Doporučený přístup | Poznámky |
|---|---:|---|---|
| Git repozitáře | Ano | GitHub Enterprise Importer, `git clone --mirror`, migrační skripty | Zachová se historie commitů, branche a tagy. |
| Pull requesty z Bitbucketu | Částečně / ano podle typu Bitbucketu | GitHub Enterprise Importer | Je nutné ověřit komentáře, review metadata, odkazy a stav PR. |
| Issues v Bitbucketu | Částečně | Export + import přes API / skripty | Pokud je hlavní evidence v Jira, obvykle se nemigrují do GitHub Issues. |
| Jira issues | Ano, ale ne 1:1 | Ponechat Jira a integrovat s GitHubem; případně export/import do GitHub Issues | Jira má bohatší workflow, pole, sprinty, timesheety a reporting. |
| Jira vazby na commity / PR | Částečně | GitHub for Jira integrace + zachování ticket klíčů v branchích/commitech/PR | Historické vazby je nutné otestovat; často vyžadují skripty nebo zůstanou jako odkazy. |
| Timesheet / worklog | Obvykle ne přímo | Ponechat v Jira / Tempo / Marketplace aplikaci | GitHub nemá plnohodnotnou nativní náhradu timesheetingu. |
| Bitbucket Pipelines | Ne přímo | Přepsat na GitHub Actions | Syntaxe a model jsou odlišné, ale většina workflow je převoditelná. |
| Deploymenty | Ano, ale vyžaduje úpravy | GitHub Actions environments, secrets, OIDC, branch protection | Nutná revize secretů, schvalování a auditů. |
| Oprávnění | Částečně | Namapovat workspace/project/repo oprávnění na GitHub org/team/repo model | Vhodné udělat nový permission model, ne slepou kopii. |
| Uživatelé a skupiny | Ano | SSO / SCIM / Enterprise Managed Users podle edice | Vyžaduje identity plán. |
| Webhooky a integrace | Částečně | Inventura + nové webhooky / GitHub Apps | Některé Atlassian Marketplace aplikace nemají GitHub ekvivalent. |
| Wiki a dokumentace | Částečně | GitHub Wiki, Markdown v repozitáři, nebo ponechat Confluence | Confluence obvykle zůstává lepší pro firemní znalostní bázi. |
| Packages / artefakty | Ano | GitHub Packages, container registry, externí registry | Je potřeba zvážit limity, billing a retention. |
| Bezpečnostní skeny | Ano | GitHub Advanced Security, Dependabot, secret scanning, CodeQL | Pokročilé funkce mohou být za příplatek podle licence. |


## Je GitHub Enterprise Importer zpoplatněn?

GitHub Enterprise Importer není samostatně zpoplatněný migrační produkt s vlastní cenou za repozitář nebo za migraci. Prakticky je dostupný jako součást GitHub Enterprise prostředí a tooling okolo migrace do GitHub Enterprise Cloud. Náklad tedy typicky nevzniká za samotné použití Importeru, ale za:

- cílové GitHub licence, například GitHub Enterprise Cloud,
- případné doplňky jako GitHub Advanced Security, Copilot, Codespaces nebo GitHub Actions nad rámec zahrnutých limitů,
- interní práci při přípravě migrace, přemapování oprávnění, úpravě CI/CD a validaci,
- případné externí konzultanty nebo GitHub Professional Services u složitější migrace.

Důležité: i když Importer samotný není samostatná licenční položka, nemusí být dostupný nebo vhodný pro každý typ GitHub plánu. Pro menší migraci bez potřeby zachovat pull request metadata lze použít i jednodušší Git mirror migraci. Pro firemní migraci s pull requesty, komentáři a více repozitáři je GitHub Enterprise Importer preferovaná cesta.

## Migrační varianty

### Varianta A: GitHub pouze pro kód, Atlassian zůstává pro řízení práce

Nejrealističtější varianta pro firmy s existující Jira/timesheet závislostí.

**Co se změní:**

- Repozitáře budou v GitHubu.
- Pull requesty, code review a CI/CD budou v GitHubu.
- Jira zůstane zdrojem pravdy pro úkoly, sprinty, worklogy a timesheety.
- GitHub a Jira se propojí přes oficiální integraci.

**Výhody:**

- Nižší riziko migrace.
- Není nutné měnit projektové řízení a timesheeting.
- Vývojáři získají GitHub Actions, Copilot, security tooling a lepší ekosystém.
- Lze migrovat postupně po týmech.

**Nevýhody:**

- Zůstávají dvě platformy.
- Nutná správa integrace GitHub ↔ Jira.
- Část nákladů Atlassianu zůstane.

### Varianta B: Plný přechod vývoje i issue trackingu do GitHubu

Vhodné jen pokud Jira používáte jednoduše a nepotřebujete pokročilé workflow, worklogy, timesheety, SLA, reporting ani rozsáhlé custom fieldy.

**Co se změní:**

- Jira issues se převedou do GitHub Issues.
- Plánování se přesune do GitHub Projects.
- Timesheeting se musí nahradit externím nástrojem nebo GitHub Marketplace řešením.

**Výhody:**

- Jedna platforma pro vývoj.
- Méně kontextového přepínání.
- Jednodušší onboarding vývojářů.
- Nižší závislost na Atlassianu.

**Nevýhody:**

- GitHub Projects nejsou plná náhrada Jira pro komplexní agile řízení.
- Chybí nativní plnohodnotný timesheeting.
- Riziko ztráty custom polí, workflow, reportingu a historických vazeb.
- Vyšší náklady na změnové řízení a školení netechnických rolí.

### Varianta C: Hybrid dlouhodobě

GitHub slouží jako vývojářská platforma, Atlassian jako business/project platforma.

**Typické rozdělení:**

- GitHub: kód, pull requesty, CI/CD, security, releases, packages, Copilot.
- Jira: product backlog, sprinty, timesheety, reporting, business workflow.
- Confluence: dokumentace, rozhodnutí, procesy.

Tato varianta je nejčastější u organizací, které už mají významné investice do Atlassianu.

## Co migrace přinese

### 1. Lepší vývojářská zkušenost

GitHub je pro mnoho vývojářů standardní prostředí. Přechod může zlepšit onboarding, code review, práci s pull requesty, dostupnost nástrojů a spolupráci napříč týmy.

Přínosy:

- známé prostředí pro nové vývojáře,
- silné pull request workflow,
- CODEOWNERS a automatické review assignmenty,
- branch protection rules a required checks,
- lepší integrace s IDE a CLI,
- jednodušší práce s open-source závislostmi a komunitním ekosystémem.

### 2. GitHub Actions jako jednotná automatizační platforma

GitHub Actions může nahradit Bitbucket Pipelines, část Jenkins/Bamboo workflow nebo interní skripty.

Nové možnosti:

- CI/CD definované přímo v repozitáři,
- reusable workflows,
- self-hosted runners,
- GitHub-hosted runners,
- environments se schvalováním deploymentů,
- OIDC federace do cloudu bez dlouhodobých cloud secretů,
- automatizace releasů, verzování, labelování, dependency update workflow.

Komplikace:

- Bitbucket Pipelines nelze převést mechanicky 1:1,
- bude nutné přemapovat secrets, proměnné, cache, artefakty a deployment environments,
- placené minuty a storage mohou zvýšit náklady,
- self-hosted runners vyžadují provozní odpovědnost.

### 3. Bezpečnost a compliance

GitHub může výrazně zlepšit bezpečnostní workflow, zejména s GitHub Advanced Security.

Možnosti:

- secret scanning,
- push protection,
- CodeQL code scanning,
- dependency review,
- Dependabot alerts a pull requesty,
- security overview napříč organizací,
- audit logy,
- SAML SSO, SCIM a enterprise policies,
- rulesets a repository policy enforcement.

Komplikace:

- GitHub Advanced Security může být významný licenční náklad,
- bude nutné nastavit baseline, triage proces a odpovědnosti,
- staré repozitáře mohou po zapnutí skenů vygenerovat mnoho nálezů,
- některé funkce závisí na edici GitHubu.

### 4. GitHub Copilot a AI workflow

Aktuální upřesnění: počítáme s 10 vývojáři, všichni GitHub Copilot používají a někteří mají individuální Copilot Pro. To znamená, že Copilot už není jen potenciální nová funkcionalita po migraci, ale existující firemní náklad a nástroj, který je vhodné převést z individuálního režimu do centrální správy.

Přechod na GitHub může otevřít cestu k širšímu využití GitHub Copilot Business nebo Enterprise.

Přínosy:

- AI asistence při psaní kódu,
- rychlejší práce s testy, refaktoringem a dokumentací,
- Copilot Chat v IDE a na GitHubu,
- vysvětlování kódu a pull requestů,
- potenciálně lepší onboarding do cizího kódu.

Komplikace:

- samostatný licenční náklad,
- nutné nastavit zásady používání AI,
- právní a bezpečnostní posouzení,
- měření reálného dopadu na produktivitu.


#### Dopad pro 10 vývojářů

Pokud dnes vývojáři používají individuální Copilot účty, firma pravděpodobně nemá jednotnou kontrolu nad licencemi, offboardingem, policy nastavením a billingem. Přechod na Copilot Business obvykle nebude levnější než individuální Copilot Pro, ale řeší firemní správu.

| Copilot scénář pro 10 vývojářů | Orientační měsíční náklad | Dopad |
|---|---:|---|
| Individuální Copilot Pro pro všechny | 100 USD | Nejnižší cena, ale bez centrální firemní správy. |
| Mix individuálních Copilot účtů, někteří Pro | cca 10–390 USD podle mixu Free / Pro / Pro+ | Nejednotný billing, různé limity a horší kontrola; minimum předpokládá alespoň jeden placený Pro účet. |
| Copilot Business pro všech 10 vývojářů | cca 190 USD | Doporučená firemní varianta: centrální seats, policy a správa. |
| Copilot Enterprise pro všech 10 vývojářů | cca 390 USD | Vhodné hlavně při Enterprise adopci a vyšších compliance / AI požadavcích. |

Doporučení: protože Copilot už používají všichni vývojáři, nedává smysl dělat pouze adopční pilot. Lepší je udělat krátký administrativní pilot převodu 1–2 uživatelů na Copilot Business, ověřit billing, policy, IDE přístup a následně převést všech 10 vývojářů. Individuální Pro předplatná je vhodné zrušit nebo nechat doběhnout podle billing cyklu, aby nevznikalo dvojí placení.

### 5. Silnější ekosystém integrací

GitHub Marketplace a GitHub Apps pokrývají bezpečnost, projektové řízení, deploymenty, monitoring, komunikaci, artefakty i compliance.

Přínosy:

- větší výběr nástrojů,
- jednodušší integrace s cloud providery,
- široká podpora SaaS nástrojů,
- standardní webhook/API model.

Komplikace:

- je nutná inventura současných Atlassian Marketplace aplikací,
- ne každá aplikace má přímý GitHub ekvivalent,
- přibývá správa oprávnění pro GitHub Apps.

## Jira, timesheet a Atlassian účet

### Jira

Jira lze po migraci ponechat a propojit s GitHubem. To je nejbezpečnější varianta, pokud Jira používáte pro:

- sprinty,
- backlog refinement,
- epics,
- custom workflow,
- custom fields,
- reporting,
- release planning,
- worklogy,
- integrace na finance, support nebo zákaznické procesy.

GitHub Issues a Projects jsou vhodné pro jednodušší technické backlogy, open-source styl práce a lightweight plánování. Pro komplexní enterprise projektové řízení Jira obvykle zůstává silnější.

### Timesheet

GitHub nemá nativní plnohodnotný timesheeting odpovídající běžným Jira doplňkům typu Tempo Timesheets nebo podobným nástrojům. Pokud dnes vykazování práce běží nad Jira worklogy, doporučení je timesheeting nemigrovat v první vlně.

Možnosti:

1. **Ponechat timesheet v Jira** – nejmenší riziko.
2. **Napojit externí time tracking na GitHub Issues/PR** – vhodné jen po ověření funkcí a reportingu.
3. **Vlastní reporting nad GitHub API** – možné, ale obvykle dražší a méně komfortní.

### Atlassian účet a Access

Pokud používáte Atlassian Access, SSO, SCIM nebo centrální správu identit, bude potřeba srovnat s GitHub Enterprise funkcemi:

- GitHub Enterprise Managed Users,
- SAML SSO,
- SCIM provisioning,
- enterprise policies,
- audit log,
- team synchronization podle identity providera.

Přechod identity modelu je kritická část migrace. Doporučuje se řešit dříve než samotný přesun repozitářů.

## Technické komplikace a rizika

| Riziko | Dopad | Doporučené opatření |
|---|---|---|
| Neúplná migrace PR komentářů a metadat | Ztráta historického kontextu | Pilotní migrace a porovnání vzorku PR. |
| Ztráta vazeb Bitbucket ↔ Jira | Horší dohledatelnost historie | Zachovat Jira klíče v názvech branchí, commitech a PR; ověřit GitHub for Jira. |
| Přepis CI/CD pipeline | Vyšší pracnost | Migrovat pipeline po kategoriích, vytvořit šablony GitHub Actions. |
| Rozdílný permission model | Bezpečnostní nebo provozní chyby | Navrhnout cílový model organizací, týmů, rolí a rulesetů. |
| Secrets v CI/CD | Riziko úniku nebo výpadku deploymentů | Udělat inventuru secretů, rotaci a přechod na OIDC, kde je možné. |
| Náklady na GitHub Advanced Security / Copilot | Výrazné zvýšení TCO | Licencovat jen relevantní uživatele / aktivní commitery, vyjednat enterprise cenu. |
| Vendor lock-in | Strategické riziko | Používat standardní Git, otevřené workflow a exportovatelné konfigurace. |
| Změna návyků týmů | Dočasné zpomalení | Školení, pilot, interní dokumentace, ambasadoři v týmech. |
| Compliance a audit | Riziko nesouladu | Zapojit security/legal/IT před migrací, ověřit audit logy a retention. |
| Marketplace aplikace | Chybějící náhrady | Inventura aplikací a rozhodnutí: nahradit, ponechat, zrušit. |

## Doporučený postup migrace

### Fáze 1: Discovery

- Sepsat všechny Bitbucket workspace/projekty/repozitáře.
- Rozdělit repozitáře podle kritičnosti, aktivity a vlastníků.
- Zjistit CI/CD nástroje a pipeline pro každý repozitář.
- Zmapovat Jira projekty, workflow, custom fields a timesheet proces.
- Zmapovat uživatele, skupiny, oprávnění, externisty a bot účty.
- Sepsat Marketplace aplikace, webhooky, deployment integrace a registry.

### Fáze 2: Cílová architektura

- Rozhodnout, zda použít GitHub Team, Enterprise Cloud nebo Enterprise Managed Users.
- Navrhnout GitHub organizace, týmy a oprávnění.
- Definovat branching model, CODEOWNERS, rulesets a branch protections.
- Definovat GitHub Actions standardy.
- Rozhodnout, co zůstane v Jira a co se přesune do GitHubu.
- Navrhnout security baseline: Dependabot, secret scanning, CodeQL, policies.

### Fáze 3: Pilot

- Vybrat 1–3 reprezentativní repozitáře.
- Provést testovací migraci.
- Přepsat pipeline do GitHub Actions.
- Ověřit Jira integraci.
- Ověřit oprávnění a audit.
- Ověřit práci vývojářů: clone, PR, review, merge, release, deployment.
- Vyhodnotit náklady na Actions, storage, Copilot a Advanced Security.

### Fáze 4: Migrační vlny

- Migrovat týmy po vlnách.
- Pro každou vlnu mít checklist, rollback plán a freeze okno.
- Zamknout nebo archivovat původní Bitbucket repozitáře po cutoveru.
- Nastavit redirect informaci v README původních repozitářů, pokud zůstávají dostupné.
- Monitorovat incidenty a support dotazy.

### Fáze 5: Optimalizace

- Odstranit duplicitní pipeline a integrace.
- Optimalizovat GitHub Actions minuty a cache.
- Zapnout širší bezpečnostní pravidla.
- Vyhodnotit adopci Copilotu.
- Rozhodnout, zda část Jira projektů přesunout do GitHub Projects.
- Snížit nebo zrušit nepotřebné Atlassian licence.

## Nákladová tabulka

Ceny níže jsou orientační veřejné list ceny v USD bez DPH, bez individuálních enterprise slev a bez přesného počtu uživatelů. Před rozhodnutím je nutné ověřit aktuální ceník u GitHubu a Atlassianu a dosadit reálné počty uživatelů, aktivních commiterů, CI minut a storage.

| Položka | Jednotka | Orientační cena | Kdy vzniká | Poznámka |
|---|---:|---:|---|---|
| GitHub Free | uživatel / měsíc | 0 USD | Malé týmy / veřejné a omezené privátní použití | Nevhodné pro enterprise řízení, SSO a compliance. |
| GitHub Team | uživatel / měsíc | cca 4 USD | Menší organizace bez enterprise požadavků | Levnější varianta, ale méně enterprise funkcí. |
| GitHub Enterprise Cloud | uživatel / měsíc | cca 21 USD | Organizace vyžadující SSO, audit, enterprise policy | Pravděpodobná cílová varianta pro firmu. |
| GitHub Advanced Security | aktivní committer / měsíc | cca 49 USD | Pokud chcete CodeQL, secret scanning, dependency review ve větším rozsahu | Významná nákladová položka; licencovat cíleně. |
| GitHub Copilot Business | uživatel / měsíc | cca 19 USD | AI asistence pro vývojáře | Pro 10 vývojářů cca 190 USD/měsíc; doporučené pro centrální firemní správu. |
| GitHub Copilot Enterprise | uživatel / měsíc | cca 39 USD | Pokročilejší enterprise AI scénáře | Pro 10 vývojářů cca 390 USD/měsíc; vhodné až při jasné potřebě enterprise AI funkcí. |
| GitHub Actions | minuta / storage | podle OS a spotřeby | CI/CD po překročení zahrnutých limitů | Linux bývá nejlevnější, macOS výrazně dražší. |
| GitHub Codespaces | hodiny + storage | podle velikosti stroje | Cloud dev prostředí | Volitelné, vhodné pro onboarding nebo specifické týmy. |
| GitHub Packages / artefakty | storage + přenosy | podle spotřeby | Registry, artefakty, kontejnery | Zvážit proti stávajícím registrům. |
| Bitbucket Standard | uživatel / měsíc | cca 3–4 USD | Současný nebo alternativní stav | Levnější Git hosting. |
| Bitbucket Premium | uživatel / měsíc | cca 6–8 USD | Současný stav s pokročilejšími funkcemi | Některé bezpečnostní funkce mohou být v ceně. |
| Jira Software Standard | uživatel / měsíc | cca 7–9 USD | Pokud Jira zůstane | Pravděpodobně zůstane kvůli backlogu a timesheetům. |
| Jira Software Premium | uživatel / měsíc | cca 14–16 USD | Pokročilé roadmapy, SLA, admin, sandbox | Časté u větších organizací. |
| Confluence Standard/Premium | uživatel / měsíc | cca 6–12 USD | Pokud používáte dokumentaci v Atlassianu | Migrace do GitHubu nemusí dávat smysl. |
| Tempo / timesheet doplněk | uživatel / měsíc | dle vendora | Pokud vykazování zůstane v Jira | Nutné zahrnout, pokud je součást procesu. |
| Migrační práce interní | člověkodny | dle sazby | Discovery, pilot, pipeline, školení | Největší skrytý náklad. |
| GitHub Enterprise Importer | migrace / repo | 0 USD jako samostatná položka | Pokud máte vhodný GitHub Enterprise cílový plán | Neplatí se typicky za Importer samotný; platí se licence, práce a případné služby. |
| Externí konzultace | projekt / denní sazba | dle dodavatele | Komplexní enterprise migrace | Volitelné; může být největší jednorázový náklad, pokud chcete řízenou migraci s garancemi. |
| Školení a adopce | tým / uživatel | dle rozsahu | Změna workflow | Snižuje riziko odporu a výpadků produktivity. |

### Modelové scénáře nákladů

| Scénář | Co obsahuje | Hrubý měsíční profil nákladů | Vhodné pro |
|---|---|---|---|
| Minimální GitHub | GitHub Team, Jira zůstává, bez Copilotu a GHAS | Nízký | Malé týmy, nízké compliance požadavky. |
| Enterprise hybrid | GitHub Enterprise Cloud + Jira + vybrané Actions | Střední | Firmy, které chtějí GitHub pro vývoj a Jira pro řízení práce. |
| Security-first | Enterprise Cloud + GHAS pro aktivní commitery + Jira | Vyšší | Regulované prostředí, bezpečnostní důraz. |
| AI-first | GitHub Team nebo Enterprise Cloud + Copilot Business + Jira | Vyšší než individuální Pro, ale řízené | Vhodné pro aktuální stav, kdy Copilot používá všech 10 vývojářů. |
| Full GitHub | Enterprise Cloud + GitHub Issues/Projects + Actions + volitelně GHAS/Copilot | Proměnlivý | Firmy ochotné opustit Jira workflow a timesheeting nahradit jinak. |

## Co je potřeba spočítat před finálním rozhodnutím

1. Počet všech vývojářů.
2. Počet aktivních commiterů za posledních 90 dní.
3. Počet uživatelů Jira a timesheetingu, kteří nejsou vývojáři.
4. Počet repozitářů a jejich velikost.
5. Počet pipeline běhů, minut a artefaktů měsíčně.
6. Počet self-hosted runnerů nebo požadované GitHub-hosted minuty.
7. Počet secrets a deployment environmentů.
8. Počet Atlassian Marketplace aplikací.
9. Náklady na interní práci migrace.
10. Náklady na školení a support po migraci.

## Doporučený cílový stav

Pro popsanou situaci je nejvhodnější cílový stav:

- GitHub Enterprise Cloud jako hlavní platforma pro repozitáře, pull requesty, code review, CI/CD a security.
- Jira ponechat jako systém pro projektové řízení, timesheety a business reporting.
- GitHub for Jira integrace jako povinná součást workflow.
- GitHub Actions postupně zavádět místo Bitbucket Pipelines.
- GitHub Advanced Security zavést nejprve pilotně pro kritické repozitáře.
- Copilot převést z individuálních účtů do Copilot Business pro všech 10 vývojářů; pilotovat už jen administrativní převod, policy a billing.
- Confluence ponechat, pokud je využívaná jako znalostní báze.

## Rozhodovací kritéria

Migrace dává smysl, pokud platí většina bodů:

- vývojáři preferují GitHub nebo s ním mají zkušenost,
- chcete zlepšit CI/CD a automatizaci,
- chcete využívat Copilot a GitHub-native AI workflow,
- chcete zlepšit security scanning a dependency management,
- Bitbucket je využíván hlavně jako Git hosting, ne jako hluboce integrovaná platforma,
- jste ochotni investovat do migrace pipeline a oprávnění.

Migraci je vhodné odložit nebo omezit, pokud:

- hlavní bolest není ve vývojářské platformě,
- Jira/Bitbucket/Atlassian integrace jsou silně customizované,
- nemáte kapacitu přepsat CI/CD,
- není jasný vlastník identity, oprávnění a security baseline,
- rozdíl v licenčních nákladech není obhajitelný přínosy.

## Doporučení pro první krok

1. Udělat inventuru Atlassian a Bitbucket prostředí.
2. Vybrat pilotní repozitáře.
3. Založit testovací GitHub organizaci nebo enterprise sandbox.
4. Ověřit migraci pomocí GitHub Enterprise Importer, pokud chcete přenést i PR metadata; pro čistý kód stačí mirror migrace.
5. Převést jednu reálnou pipeline na GitHub Actions.
6. Propojit pilot s Jira.
7. Spočítat reálné měsíční náklady podle spotřeby.
8. Vyhodnotit pilot s vývojáři, IT, security a product/project managementem.

## Zdroje k ověření

- GitHub Pricing: https://github.com/pricing
- GitHub Enterprise Importer dokumentace: https://docs.github.com/en/migrations/using-github-enterprise-importer
- Bitbucket migrations with GitHub Enterprise Importer: https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-from-bitbucket-to-github-enterprise-cloud
- GitHub Actions billing: https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions
- GitHub Advanced Security: https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security
- GitHub for Jira integrace: https://github.com/integrations/jira
- Atlassian Bitbucket pricing: https://bitbucket.org/product/pricing
- Atlassian Jira pricing: https://www.atlassian.com/software/jira/pricing
- Atlassian Marketplace: https://marketplace.atlassian.com/
