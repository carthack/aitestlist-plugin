---
name: error-report
description: Analyse les taches echouees d'un projet AI TestList (outils MCP failed_tasks et create_error_report) et genere un rapport d'erreurs detaille. Le rapport PDF devient disponible sur le site. Appele par test-reporter apres execution ou manuellement.
user-invocable: false
---

# Error Analysis Report

Skill pour analyser les taches echouees et generer un rapport d'erreurs PDF.
Prechage dans l'agent `test-reporter` via le champ `skills:`.
Utilise apres une execution ou invoque directement via `@test-reporter`.

## Outils

Les outils s'appellent `mcp__plugin_aitest_aitestlist__<outil>`. N'appelle jamais l'API toi-meme
(curl, Python). `USER_LANG` vient du preflight (`get_account`). Resume des regles cote serveur :
`test_execution_guide`, section « After the execution ».

| Pour | Outil |
|------|-------|
| Lister les projets | `list_projects` |
| Taches echouees d'un projet | `failed_tasks(project_id)` → `task_id`, `title`, `test_name`, `description_raw`, `category`, `comment` |
| Creer le rapport PDF | `create_error_report(project_id, diagnoses, task_index)` |

Parametres de `create_error_report` :
```json
{
  "project_id": 9,
  "task_index": {
    "123": "Page d'accueil du portail affilies",
    "456": "Validation formulaire inscription"
  },
  "diagnoses": {
    "123": {
      "error": "Description courte de l'erreur probable",
      "cause": "Explication de la cause racine",
      "solution": "La meilleure solution en une phrase",
      "steps": ["Etape 1", "Etape 2"]
    }
  }
}
```

**IMPORTANT** : `task_index` est un dictionnaire `{task_id: titre}` pour toutes les taches
echouees (lexique en fin de rapport PDF). Le construire avec `task_id` et `title` de `failed_tasks`.

Si la connexion est refusee (« Not connected », 401) : dire a l'utilisateur
« tapez `/mcp`, choisissez `aitestlist`, puis Authenticate », puis STOP.

## Workflow

1. **Choisir le projet** - `list_projects` et laisser l'utilisateur choisir (ou recevoir le project_id du demandeur)
2. **Recuperer les taches echouees** - `failed_tasks(project_id)`
3. **Analyser TOUTES les taches en un seul bloc** - Voir section "Analyse batch" ci-dessous
4. **Envoyer les diagnostics** - `create_error_report` (un seul appel)
5. **Confirmer** - Informer l'utilisateur que le rapport est disponible

## IMPORTANT: Analyse batch (pas une par une!)

**PERFORMANCE CRITIQUE** — Analyser toutes les taches echouees en UN SEUL passage.

**NE PAS FAIRE:** Boucler sur chaque tache individuellement avec un prompt par tache.
Cela genere N analyses et prend 30+ minutes pour 30 taches.

**FAIRE:** Lire toutes les taches echouees retournees par `failed_tasks`, puis produire
tous les diagnostics dans une seule reponse. Le JSON de diagnostics complet
doit etre construit en un seul bloc puis envoye en un seul appel de `create_error_report`.

### Etapes concretes:

1. `failed_tasks(project_id)` → recevoir la liste complete
2. Lire TOUTES les taches d'un coup (elles sont dans la reponse JSON)
3. Produire le JSON `diagnoses` complet avec tous les task_id en UNE SEULE reponse
4. `create_error_report` avec le JSON complet → UN SEUL appel

Total: 2 appels d'outils (`failed_tasks` + `create_error_report`). Pas de boucle.

## Champs du diagnostic (par tache)

- **error**: Description concise de l'erreur probable (1-2 phrases)
- **cause**: Explication de la cause racine (2-3 phrases)
- **solution**: LA meilleure solution en une phrase claire (resume de haut niveau, PAS la meme chose que step 1)
- **steps**: Etapes concretes pour implementer la solution. IMPORTANT sur le nombre d'etapes:
  - Bug simple (1 import manquant, 1 config) → 1 etape
  - Correction moderee (2 fichiers a modifier) → 2 etapes
  - Correction complexe (architecture, multi-fichiers) → 3-4 etapes
  - Refactoring majeur → 5 etapes max
  - **NE PAS toujours mettre 3 etapes.** Varier selon la complexite reelle du probleme.

## Methodologie d'analyse

Pour chaque tache dans le batch:
1. **Lire la description** - Comprendre les preconditions, les etapes et le resultat attendu
2. **Analyser le commentaire** - Le testeur a souvent laisse des indices
3. **Considerer la categorie** - Un echec en securite n'a pas les memes causes qu'en UI
4. **Etre specifique** - Eviter les diagnostics generiques comme "verifier le code"
5. **Proposer UNE solution actionnable** - La meilleure, avec des etapes concretes pour l'implementer
6. **Regrouper les erreurs similaires** - Si plusieurs taches echouent pour la meme raison, le mentionner
7. **Utiliser les IDs numeriques** - Dans les textes de diagnostic, referencer les taches par leur ID (ex: "resout taches 1973, 1989") plutot que par leur titre complet. Le lexique task_index en fin de rapport fournit la correspondance ID → titre

### Exemple de bon diagnostic

```json
{
  "error": "Le formulaire de login accepte des identifiants vides - la validation cote client est contournee quand JavaScript est desactive",
  "cause": "La validation est uniquement cote client (JavaScript). Il n'y a pas de validation cote serveur dans le controleur /auth/login.",
  "solution": "Ajouter une validation serveur dans le formulaire WTForms et la route Flask",
  "steps": [
    "Ouvrir website/codeDir/forms/auth.py",
    "Ajouter DataRequired() sur les champs email et password du LoginForm",
    "Dans la route /auth/login, verifier form.validate_on_submit() avant traitement"
  ]
}
```

### Exemple de MAUVAIS diagnostic (a eviter)

```json
{
  "error": "Le test a echoue",
  "cause": "Il y a un bug dans le code",
  "solution": "Corriger le bug",
  "steps": ["Verifier le code"]
}
```

## Langue du rapport

Les diagnostics doivent etre rediges dans la langue de l'utilisateur (`USER_LANG` du preflight):
- Si `USER_LANG` = `fr` → diagnostics en francais
- Si `USER_LANG` = `en` → diagnostics en anglais

## Appel automatique par test-reporter

Quand appele par l'agent `test-reporter` apres une execution:
- Le project_id est fourni automatiquement
- Pas besoin de demander a l'utilisateur de choisir
- Le rapport est genere et envoye sans interaction
- En mode teams, le reporter a deja les taches echouees en memoire — utiliser
  `failed_tasks` pour avoir le format complet si necessaire

## Apres envoi

Informer l'utilisateur:
1. Nombre de taches analysees
2. Le rapport PDF a ete genere et stocke sur AI TestList
3. Pour le telecharger: **https://aitestlist.com/reports** → selectionner le projet → section "Rapports disponibles"
