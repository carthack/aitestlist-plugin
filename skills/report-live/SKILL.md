---
name: report-live
description: Pousse les resultats de test a AI TestList en temps reel, tache par tache, avec l'outil MCP report_result. Le client voit les statuts se mettre a jour en direct. Utilise par exec-test apres chaque tache ou par l'agent test-reporter en mode teams.
user-invocable: false
---

# Report Live

Envoi en temps reel des resultats de test a AI TestList, juste apres chaque tache.
Les regles completes (statuts, commentaires, quand utiliser `report_results` en lot) sont dans
`test_execution_guide` (section « Status codes to report ») : les suivre.

## Outil

`mcp__plugin_aitest_aitestlist__report_result` avec :

| Parametre | Exemple | Note |
|-----------|---------|------|
| `queue_id` | `42` | File d'execution |
| `task_id` | `123` | Tache de `get_execution_queue` |
| `status` | `succes` | `succes`, `echec` ou `erreur` |
| `comment` | `null` | Dans `USER_LANG` ; `null` si la tache passe simplement |
| `duration_ms` | `342` | Facultatif |

Le premier appel fait passer la file a `running`. La reponse donne l'avancement.

## Workflow

1. Recevoir : task_id, status, comment, duration_ms, queue_id
2. Valider le statut (`succes` / `echec` / `erreur`)
3. Appeler `report_result` immediatement
4. Si l'appel echoue (delai, 500) : reessayer 1 fois apres 2 secondes
5. Si le second essai echoue : garder le resultat en memoire, continuer
6. Confirmer au demandeur

Si la connexion est refusee (« Not connected », 401) : STOP, et dire a l'utilisateur
« tapez `/mcp`, choisissez `aitestlist`, puis Authenticate ».

## Utilisation dans Claude Code

**Mode sequentiel :** l'agent `test-executor` appelle `report_result` apres chaque tache.

**Mode teams :** les sous-agents d'execution envoient leurs resultats a l'agent `test-reporter`
par SendMessage ; le reporter appelle `report_result` pour chacun.

## Effet cote client

La page d'execution d'AI TestList se rafraichit regulierement : la ligne de la tache passe
de gris a vert (succes), rouge (echec) ou orange (erreur), et le compteur avance.
