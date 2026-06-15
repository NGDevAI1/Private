# Stručné shrnutí migrace z Bitbucket / Atlassian na GitHub

## Doporučení

- GitHub dává smysl jako hlavní vývojářská platforma pro kód, pull requesty, CI/CD, bezpečnost a AI nástroje.
- Jira může zůstat zachovaná, pokud aktuálně dobře funguje pro řízení práce, timesheety a reporting.
- Nejpraktičtější první krok je pilotní migrace několika repozitářů a jednoho typického buildu.

## Možné varianty

### Varianta A: GitHub pro kód, Jira zůstává

- Git repozitáře se přesunou do GitHubu.
- Pull requesty, code review, Actions, security a Copilot budou v GitHubu.
- Jira zůstane pro backlog, sprinty, úkoly, reporting a timesheet.
- Nejnižší organizační riziko.

### Varianta B: Kompletní přechod do GitHubu

- Kód, issues, projekty i automatizace se přesunou do GitHubu.
- Atlassian stack se výrazně omezí nebo zruší.
- Větší úspora nástrojů, ale vyšší změna pro týmy.
- Nutné ověřit náhradu Jira procesů a timesheetů.

### Varianta C: Dlouhodobý hybrid

- Část týmů nebo projektů běží v GitHubu, část dál v Atlassianu.
- Vhodné pro postupný přechod.
- Riziko dvojí správy, nejednotných procesů a horšího reportingu.

## Co GitHub přinese

- Silnější pull request workflow.
- Lepší integrace s IDE, CLI a vývojářským ekosystémem.
- GitHub Actions pro CI/CD přímo v repozitáři.
- Branch protection, CODEOWNERS a required checks.
- GitHub Advanced Security: secret scanning, CodeQL, dependency review.
- GitHub Copilot a AI workflow přímo nad repozitářem.
- Reusable workflows a lepší automatizace releasů.

## GitHub Actions a buildy

- GitHub Actions může nahradit Bitbucket Pipelines, část Jenkins/Bamboo workflow nebo TeamCity.
- Pro .NET lze řešit restore, build, test, publish, ZIP artefakty i PowerShell skripty.
- Čisté .NET / .NET Core buildy mohou běžet na GitHub-hosted runnerech.
- Legacy .NET Framework, interní nástroje, licence nebo přístup do interní sítě mohou vyžadovat self-hosted Windows runner.

## Self-hosted Windows runner jako možnost

- Vhodný pro buildy závislé na Windows, Visual Studio Build Tools, interní síti nebo lokálních licencích.
- Přidává se přes **Settings → Actions → Runners → New self-hosted runner**.
- Ve workflow se používá například přes `runs-on: [self-hosted, windows]`.
- Měl by běžet na odděleném stroji nebo VM.
- Vyžaduje provozní odpovědnost, aktualizace a bezpečnostní omezení.

## Jira, timesheet a Atlassian

- Jira nemusí být nahrazena hned.
- GitHub má issues a projects, ale nemusí pokrýt všechny firemní procesy.
- Oficiální integrace GitHub - Jira je zcela zdarma.
- Timesheet je potřeba posoudit samostatně.
- Atlassian Access / SSO / správa uživatelů může zůstat relevantní, pokud Jira zůstane.

## Hlavní rizika

- Migrace issue historie a vazeb z Jira / Bitbucketu.
- Převod pipeline není mechanický 1:1.
- Přemapování secrets, proměnných, cache, artefaktů a prostředí.
- Náklady na GitHub licence, Actions minuty, storage a případně Advanced Security.
- Změna návyků vývojářů a nutnost sjednotit procesy.
- Provozní odpovědnost u self-hosted runnerů.

## Doporučený postup

1. Zmapovat repozitáře, pipeline, build závislosti a Jira workflow.
2. Vybrat cílový model: GitHub-only, GitHub + Jira, nebo hybrid.
3. Udělat pilot na několika repozitářích.
4. Převést typický build do GitHub Actions.
5. Ověřit permissions, branch protection, secrets a security nástroje.
6. Vyhodnotit náklady a dopad na tým.
7. Migrovat po vlnách.

## Co spočítat před rozhodnutím

- Počet vývojářů a typ GitHub licencí.
- Potřebu GitHub Advanced Security.
- Očekávaný počet Actions minut.
- Velikost artefaktů a dobu jejich uchování.
- Náklady na provoz self-hosted runnerů.
- Náklady na ponechání nebo zrušení části Atlassian stacku.

## Doporučený cílový stav

- GitHub jako hlavní platforma pro repozitáře, pull requesty, CI/CD, bezpečnost a Copilot.
- Jira ponechat tam, kde přináší hodnotu pro plánování, reporting a timesheet.
- GitHub Actions používat jako standardní automatizační platformu.
- Self-hosted Windows runnery použít pouze tam, kde jsou skutečně potřeba.
- Migraci provést postupně přes pilot a migrační vlny.
