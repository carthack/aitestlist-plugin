# Greffon aitest - Architecture

> AI TestList : creer, executer et rapporter des tests avec votre assistant IA.
> Version : 2.0.0 | Auteur : Solutions Codexalia Inc.

## Vue d'ensemble

Le greffon `aitest` (marche `aitestlist-plugin`, depot `carthack/aitestlist-plugin`) branche Claude Code
sur le **serveur MCP d'AI TestList** (`https://aitestlist.com/mcp`). Il permet de :

- **Creer** des tests a partir de l'analyse du code d'un projet
- **Executer** une file d'execution approuvee dans un navigateur (Playwright ou Claude dans Chrome)
- **Reporter** chaque resultat en direct sur AI TestList
- **Analyser** les echecs et produire un rapport d'erreurs PDF

```
Utilisateur
    |-- @test-creator   → agent (analyse du projet + creation des tests)
    |-- @test-executor  → agent (execution dans le navigateur)
    |-- @test-reporter  → agent (rapport d'erreurs PDF ; hub de reporting en mode teams)
    |-- /aitest:status  → skill (diagnostic)
    v
Outils MCP mcp__plugin_aitest_aitestlist__*   (.mcp.json)
    v
https://aitestlist.com/mcp  --(jeton OAuth de l'usager)-->  API d'AI TestList
```

### Ce qui vit ou

| Ou | Contenu |
|----|---------|
| Serveur MCP (depot AITestList, `mcp_server/`) | Les 21 outils et les deux guides `test_writing_guide` / `test_execution_guide` : format des tests, ordre des taches, statuts, marqueurs courriel et paiement, elevation en base, correction automatique. **Source de verite.** |
| Ce greffon | Ce qui est propre a Claude Code : agents et sous-agents (mode teams), affichage de progression, outils du navigateur (Playwright, Claude dans Chrome), lecture du code du projet (Read/Grep/Glob), requetes en base par Bash, detection des systemes de paiement dans le code. |

Les skills ne recopient pas les guides : ils disent d'appeler le guide, puis ajoutent ce qui est propre a Claude Code.

## Connexion

- Aucune cle a copier. Une fois le greffon installe : `/mcp` → `aitestlist` → « Authenticate ».
  Le site ouvre la page « Autoriser » (OAuth). **Forfait payant requis.**
- Les agents n'appellent jamais l'API directement (curl, Python) : elle refuse tout appel qui ne vient pas du serveur MCP.
- En cas de refus d'acces (« Not connected », 401) : refaire `/mcp` → `aitestlist` → « Authenticate ».

## Structure

```
aitestlist-plugin/
|-- .claude-plugin/
|   |-- plugin.json          # Greffon aitest (version, auteur)
|   |-- marketplace.json     # Marche aitestlist-plugin
|-- .mcp.json                # Serveur MCP aitestlist (http, https://aitestlist.com/mcp)
|-- agents/
|   |-- test-creator.md
|   |-- test-executor.md
|   |-- test-reporter.md
|-- skills/
|   |-- status/              # Diagnostic (seul skill visible dans le menu /)
|   |-- preflight/           # get_account : connexion, forfait, langue
|   |-- create-test/         # Creation de tests (renvoie a test_writing_guide)
|   |-- create-payment/      # Detection Stripe/PayPal dans le code
|   |-- exec-test/           # Execution dans le navigateur (renvoie a test_execution_guide)
|   |-- exec-payment/        # Champs Stripe (iframes) avec les outils du navigateur
|   |-- exec-email/          # Aide-memoire des outils de courriel de test
|   |-- exec-db-elevation/   # Elevation temporaire en base par Bash
|   |-- report-live/         # report_result tache par tache
|   |-- error-report/        # Analyse des echecs + create_error_report
|   |-- pdf/ docx/ xlsx/ pptx/   # Skills utilitaires de documents (non precharges)
|-- docs/ARCHITECTURE.md
|-- README.md
```

## Agents

| Agent | Modele | Skills precharges | Outils AI TestList |
|-------|--------|-------------------|--------------------|
| test-creator | Opus | preflight, create-test, create-payment | get_account, list_projects, list_categories, submit_test, pending_tests, test_writing_guide |
| test-executor | Opus | preflight, exec-test, exec-payment, exec-email, exec-db-elevation, report-live | get_account, get_exec_mode, list_execution_queues, get_execution_queue, report_result, report_results, finalize_execution, queue_status, create_test_email, list_emails, read_email, wait_for_email, delete_test_email, test_execution_guide |
| test-reporter | Sonnet | preflight, report-live, error-report | get_account, list_projects, report_result, queue_status, failed_tasks, create_error_report |

`test-executor` a aussi les outils Playwright (`mcp__plugin_playwright_playwright__browser_*`) et
Claude dans Chrome (`mcp__claude-in-chrome__*`), plus Bash, Read, Edit, Grep, Glob, Task et SendMessage.
Les agents declarent leurs skills dans le champ `skills:` : le contenu est injecte au demarrage, aucun skill n'en appelle un autre.

## Deroulement

### Creer des tests (`@test-creator`)

1. `get_account` (langue), `test_writing_guide`
2. Analyse du code du projet (toujours a neuf)
3. `list_categories(lang)`, `list_projects` si un projet est nomme
4. `submit_test` (un par test) → file d'import, approbation par l'usager sur https://aitestlist.com/import

### Executer (`@test-executor`)

1. `get_account`, `test_execution_guide`, verification du navigateur, `get_exec_mode`
2. `list_execution_queues` / `get_execution_queue(queue_id)`
3. Serveur cible injoignable → `report_results` (tout en erreur, ferme la file) → STOP
4. Chaque tache : execution dans le navigateur, puis `report_result` immediat
5. `finalize_execution(queue_id)`, rapport final

Cycle de vie d'une file : `approved` → `running` (1er `report_result`) → `executed` (`finalize_execution` ou `report_results`).

### Mode teams

Active par `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` dans `~/.claude/settings.json`. Le leader `test-executor`
lance `test-reporter` et N sous-agents d'execution ; les sous-agents envoient leurs resultats au reporter
(SendMessage), qui appelle `report_result` ; le leader finalise.

### Rapport d'erreurs (`@test-reporter`)

`list_projects` → `failed_tasks(project_id)` → analyse de tous les echecs en un passage →
un seul `create_error_report(project_id, diagnoses, task_index)` → PDF sur https://aitestlist.com/reports

## Modes du navigateur (`get_exec_mode`)

| Mode | Fenetre | Usage |
|------|---------|-------|
| `batch` | invisible | `claude -p` |
| `interactive_headless` | invisible | Terminal seulement |
| `interactive_browser_minimal` | 1280x720 | Defaut |
| `interactive_browser_fullscreen` | maximisee | Tests visuels |

Reglage dans AI TestList > Parametres > Execution.

## Installation

Voir `README.md`.

> Derniere mise a jour : 2026-10-29
