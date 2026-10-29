---
name: create-test
description: Cree et soumet des tests a AI TestList avec les outils MCP (list_categories, list_projects, submit_test). Genere des taches de test intelligentes avec les bonnes categories. Skill core du greffon aitest.
user-invocable: false
---

# Create Test

Skill core pour creer et soumettre des tests a AI TestList.

Les outils s'appellent `mcp__plugin_aitest_aitestlist__<outil>`. N'appelle jamais l'API toi-meme
(curl, Python) : elle refuse tout appel qui ne vient pas du serveur MCP.

## Regle d'or : le guide du serveur fait foi

**Avant d'ecrire le moindre test, appeler `test_writing_guide`** et le suivre exactement :
format de `submit_test`, structure des descriptions (Preconditions / Etapes / Attendu),
types de taches (`[SETUP]`, `[TEST]`, `[VERIFY]`, `[TEARDOWN]`), ordre des taches,
`[TEARDOWN]` obligatoire, donnees de test, marqueur `[CREATE_TEST_EMAIL:{context}]`,
tests de paiement (`[PAYMENT_TEST]`), regles de qualite et exemples.
Ne pas s'en remettre a une version memorisee : le guide peut changer.

## IMPORTANT: Status Output (Live Progress)

**Tu DOIS afficher des messages de status a chaque etape.**
Ces messages sont visibles en temps reel dans le terminal Claude Code.

**Format obligatoire — afficher en texte brut (PAS dans un bloc de code):**

**Chargement des categories:**
```
📋 Loading test categories (FR)...
   → 47 categories in 8 groups loaded
```

**Chargement des projets:**
```
📂 Loading projects...
   → Found 5 projects
   → Target: "Mon Application" (id: 12)
```

**Generation des taches — lister CHAQUE tache:**
```
🧠 Generating test tasks for "[description]"...
   → [1/8]  [SETUP] Inscrire un compte de test
   → [2/8]  Connexion avec identifiants valides
   → [3/8]  Connexion avec mauvais mot de passe
   → [4/8]  Connexion avec email inexistant
   → [5/8]  Connexion avec champs vides
   → [6/8]  Verifier le lien "Mot de passe oublie"
   → [7/8]  Verifier la protection CSRF
   → [8/8]  [TEARDOWN] Nettoyer les donnees de test
```

**Soumission:**
```
📤 Submitting 8 tasks to AI TestList...
✅ Test "Test page de connexion" queued for import!
   → 8 tasks
   → Categories: Techniques > Securite > Authentification, Comportementales > Fonctionnalite > Workflow
   → Import queue: https://aitestlist.com/import
   → Approve the import to create the test
```

## Workflow

`USER_LANG` vient du preflight (`get_account`).

1. **Guide** — `test_writing_guide`
2. **Categories** — `list_categories(lang=USER_LANG)` ; utiliser uniquement ces chemins (« A > B > C »)
3. **Projet** — `list_projects` si l'utilisateur a nomme un projet ; `project_id` seulement pour un projet
   dont il est proprietaire, sinon l'omettre (il choisira a l'approbation)
4. **Analyser** — scanner le code du projet courant avec Read, Glob, Grep (toujours a neuf, pas de cache)
5. **Specialites** — paiement detecte (Stripe, PayPal) : suivre aussi les instructions create-payment
6. **Generer** — les taches **dans `USER_LANG`**, pas dans la langue de la conversation
7. **Soumettre** — `submit_test(name, tasks, project_id)` : un appel par test (un test par page ou fonctionnalite)
8. **Confirmer** — voir « Apres soumission »

## Erreurs

- `Project not found or access denied` : retirer `project_id` et soumettre de nouveau.
- `LIMIT_REACHED` : limite du forfait atteinte ; le dire a l'utilisateur, ne pas reessayer.
- « Not connected », 401 ou acces refuse : dire a l'utilisateur
  « tapez `/mcp`, choisissez `aitestlist`, puis Authenticate » (forfait payant requis), puis STOP.

## Apres soumission

Informer l'utilisateur :
1. Nombre de taches creees
2. Categories utilisees
3. Le test attend son approbation dans la file d'import : **https://aitestlist.com/import**
   (`pending_tests` liste ce qui attend encore)
