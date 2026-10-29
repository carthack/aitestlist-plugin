---
name: exec-test
description: Telecharge une file d'execution approuvee d'AI TestList (outils MCP) et execute les tests dans un navigateur (Playwright ou Claude dans Chrome) depuis Claude Code. Skill core du greffon aitest.
user-invocable: false
---

# Execute Tests

Skill core pour executer les tests AI TestList dans un navigateur, depuis Claude Code.

## IMPORTANT: Status Output (Live Progress)

**Tu DOIS afficher des messages de status a chaque etape.**
Ces messages sont visibles en temps reel dans le terminal Claude Code.
Ils donnent un effet professionnel et montrent la progression au client.

**Format obligatoire — afficher en texte brut (PAS dans un bloc de code).**

**Verification du navigateur:**
```
🎭 Checking browser...
✅ Playwright ready
```

**Mode d'execution:**
```
👤 Execution mode: interactive_browser_minimal (1280x720)
```

**Download queue:**
```
📥 Downloading execution queue #42...
✅ Queue #42 loaded
   → Project: "Mon Application" (id: 5)
   → Tests: 3 tests, 24 tasks total
   → Auto-fix: OFF
```

**Rules:**
```
📜 Loading execution rules...
   ┌─────────────────────────────────────────────────┐
   │  GLOBAL RULES                                   │
   │  [security] Always check CSRF tokens on forms   │
   │  [general] Test in both FR and EN               │
   │                                                 │
   │  PROJECT RULES                                  │
   │  [general] Login page is at /login              │
   │  [a11y] All forms must be keyboard navigable    │
   └─────────────────────────────────────────────────┘
```

**Avant chaque test:**
```
═══════════════════════════════════════════════════
  📋 Test 1/3: Authentication - Login Page
  Tasks: 8
═══════════════════════════════════════════════════
```

**Pour chaque tache — afficher les actions du navigateur en temps reel:**
```
  🔄 [1/8] Login with valid credentials...
     → Creating test email: create_test_email
     → Got: login_test_queue42@...
     → Navigate to /login
     → Snapshot: found email field, password field, Login button
     → Fill email: login_test_queue42@...
     → Fill password: ********
     → Click "Login" button
     → Snapshot: URL=/dashboard, "Welcome" text found
     → ✅ PASSED
     → 📤 Result pushed live
```

En cas d'echec:
```
  🔄 [3/8] Login with empty fields...
     → Navigate to /login
     → Snapshot: found form fields
     → Leave fields empty
     → Click "Login" button
     → Expected: "Required field" message
     → Got: Form submitted without validation
     → ❌ FAILED — No client-side validation
     → 📤 Result pushed live
```

En cas d'erreur:
```
  🔄 [5/8] Login with expired session...
     → Navigate to /dashboard
     → ⚠️ ERROR — Timeout after 10s
     → 📤 Result pushed live
```

Auto-fix:
```
  🔧 Auto-fix triggered for task [N]...
     → Analyzing: website/templates/auth/login.html
     → Found: missing 'required' on email input (line 23)
     → Applying fix...
     → Re-testing...
     → ✅ FIXED — Validation now works
```

**Apres chaque test:**
```
  ── Test complete: 7/8 passed, 1 failed ──
```

**Finalize:**
```
📤 Finalizing execution queue #42...
✅ Queue finalized
```

**Rapport final:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  📊 EXECUTION REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Tests:    3
  Tasks:    24 total
  ✅ Passed: 20
  ❌ Failed: 3
  ⚠️  Errors: 1

  Success rate: 83%

  ❌ Failed tasks:
     → [Login] Login with empty fields — Missing validation
     → [Login] SQL injection — Server returned 500
     → [Register] Duplicate email — No error message

  ⚠️  Errors:
     → [Profile] Update avatar — Timeout after 10s

  📤 Results sent to AI TestList
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Regle d'or : le guide du serveur fait foi

**Avant d'executer, appeler `test_execution_guide`** et le suivre exactement : deroulement,
statuts a reporter (`succes` / `echec` / `erreur`, regle « Suggestion »), marqueurs
`[CREATE_TEST_EMAIL:...]` et `[PAYMENT_TEST]`, elevation de permissions en base, correction
automatique, erreurs fatales. Ce skill ne garde que ce qui est propre a Claude Code :
le navigateur, l'affichage, la delegation aux autres instructions.

Les outils d'AI TestList s'appellent `mcp__plugin_aitest_aitestlist__<outil>`. N'appelle jamais
l'API toi-meme (curl, Python) : elle refuse tout appel qui ne vient pas du serveur MCP.
`USER_LANG` vient du preflight (`get_account`) : commentaires et rapport final dans cette langue.

## Etape 1: Verifications prealables

### Etape 1A: Navigateur

Deux choix dans Claude Code :
- **Playwright** (recommande) : outils `mcp__plugin_playwright_playwright__browser_*`.
  Verifier avec un appel simple (`browser_tabs` ou `browser_snapshot`).
- **Claude dans Chrome** : outils `mcp__claude-in-chrome__*`. Pilote le Chrome de l'utilisateur
  (ses vraies sessions et ses temoins de connexion) : attention a ses comptes existants.

Aucun des deux → afficher ❌ « Aucun navigateur disponible. Installez le greffon Playwright :
`/plugin install playwright@claude-plugins-official` » → **STOP**.

### Etape 1B: Mode d'execution

`get_exec_mode` → `exec_mode` et `payment_testing` (garder pour exec-payment).

| Mode | Playwright | Comportement |
|---|---|---|
| `batch` | headless | Non-interactif (`claude -p`). Pas de navigateur visible. |
| `interactive_headless` | headless | Session interactive, terminal visible, pas de navigateur. |
| `interactive_browser_minimal` | visible, 1280x720 | Fenetre compacte (defaut). `browser_resize(1280, 720)`. |
| `interactive_browser_fullscreen` | visible, maximise | Fenetre maximisee via CDP apres la premiere navigation. |

**Mode plein ecran :** apres la premiere navigation, avec `browser_run_code` (ou `browser_run_code_unsafe`
selon la version de Playwright) :
```javascript
const cdp = await page.context().newCDPSession(page);
const { windowId } = await cdp.send('Browser.getWindowForTarget');
await cdp.send('Browser.setWindowBounds', { windowId, bounds: { windowState: 'maximized' } });
```

## Etape 2: Telecharger la file d'execution

`list_execution_queues` si l'utilisateur n'a pas donne de numero, puis `get_execution_queue(queue_id)`.
Contenu et erreurs possibles : voir le guide. File vide (`tests` vide) → le dire a l'utilisateur.

## Etape 3: Regles et correction automatique

Afficher `global_rules` et `project_rules` (voir l'affichage « Rules » plus haut) : ce sont des
directives a respecter pendant toute l'execution. Lire `auto_fix_enabled`.

## Etape 3b: Delegation aux instructions specialisees

Avant chaque tache (instructions prechargees dans l'agent via `skills:`) :

| Condition | Instructions |
|-----------|-------------|
| Description contient `[PAYMENT_TEST]` | exec-payment |
| Description contient `[CREATE_TEST_EMAIL:...]` | exec-email |
| Echec du a une restriction de forfait ou de role | exec-db-elevation |

## Etape 3c: Serveur cible joignable

Naviguer vers `target_url`.
```
🌐 Checking target server: http://localhost:8005
✅ Target server reachable
```
Connexion refusee ou delai depasse :
```
🌐 Checking target server: http://localhost:8005
❌ Target server unreachable: http://localhost:8005
   All tasks will be marked as error.
```
→ toutes les taches en `erreur` en UN appel `report_results(queue_id, results)` (ferme la file ;
ne PAS appeler `finalize_execution`), rapport final, **STOP**.

## Etape 4: Executer les tests

Pour chaque test, puis chaque tache par `position` : suivre « Executing a task » du guide,
puis **`report_result` immediatement** (instructions report-live).

### Outils Playwright

| Action | Outil |
|---|---|
| Naviguer | `browser_navigate(url)` |
| Etat de la page + refs | `browser_snapshot()` |
| Cliquer | `browser_click(ref, element)` |
| Remplir un champ | `browser_type(ref, text)` ou `browser_fill_form(fields)` |
| Choisir une option | `browser_select_option(ref, values)` |
| Touche clavier | `browser_press_key(key)` |
| Boite de dialogue du navigateur | `browser_handle_dialog(accept)` |
| Attendre | `browser_wait_for(text)` ou `browser_wait_for(time)` |
| Evaluer du JS | `browser_evaluate(function)` |
| Capture d'ecran | `browser_take_screenshot()` |
| Onglets / fenetres | `browser_tabs` |
| Console, requetes reseau | `browser_console_messages`, `browser_network_requests` |

**IMPORTANT :** les refs changent apres chaque action — reprendre un `browser_snapshot()` avant d'interagir.

### Equivalents Claude dans Chrome

`navigate`, `read_page` / `get_page_text` (etat de la page), `find` (reperer un element),
`computer` (clic, saisie, capture), `form_input`, `javascript_tool`, `read_console_messages`,
`read_network_requests`, `tabs_context_mcp` / `tabs_create_mcp`.

### Correction automatique et verification apres action

Seulement si `auto_fix_enabled` est vrai (regles dans le guide). Dans Claude Code : lire et corriger
le code du projet avec Read, Grep, Edit ; reporter l'echec AVANT de corriger, re-tester, puis
reporter le nouveau resultat (« Fixed: ... » ou « Fix attempted but failed »).

Apres une tache qui modifie l'etat (`[SETUP]`, correction, insertion en base) : ouvrir les pages
touchees, `browser_snapshot`, verifier l'absence d'erreur (500, UndefinedError). Au plus 3
tentatives de correction par page, puis continuer. Cette verification ne change PAS le statut de la tache.

## Etape 5: Finaliser

Quand toutes les taches sont reportees : `finalize_execution(queue_id)`.
Ne jamais appeler `report_results` apres des resultats envoyes en direct.

## Etape 6: Rapport final (dans USER_LANG)

```
=== RESUME ===
Tests: 5
Taches: 45 total, 40 succes, 3 echecs, 1 erreur, 1 suggestion
Taux de reussite: 89%
Resultats envoyes a AI TestList.
```

**IMPORTANT:** Le rapport est la DERNIERE chose affichee. Rien apres.

## Gestion d'erreurs

### Erreurs fatales — ARRET IMMEDIAT

Ne JAMAIS contourner (pas d'appel direct a l'API, pas d'URL devinee).

- **Aucun navigateur** → ❌ + consigne d'installation de l'etape 1A → **STOP**
- **Connexion a AI TestList refusee ou expiree** (« Not connected », 401, acces refuse) →
  ❌ « Tapez `/mcp`, choisissez `aitestlist`, puis Authenticate » → **STOP**
- **Forfait sans execution** (`exec_test_enabled` faux) → ❌ → **STOP**
- **File introuvable** → ❌ « File #ID introuvable. » → **STOP**
- **File pas approuvee** → ❌ « Approuvez-la dans AI TestList. » → **STOP**
- **Serveur cible injoignable** → etape 3c → **STOP**

### Erreurs de tache — continuer

Element introuvable, delai depasse sur une action, erreur JavaScript dans la page →
`erreur` + description, puis tache suivante.
