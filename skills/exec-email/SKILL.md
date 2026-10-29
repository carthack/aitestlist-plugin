---
name: exec-email
description: Gere les adresses de courriel de test (creation, attente, lecture, nettoyage) avec les outils MCP d'AI TestList. Utilise par exec-test quand une tache contient [CREATE_TEST_EMAIL:...].
user-invocable: false
---

# Execute Email Testing

Instructions pour les taches qui contiennent `[CREATE_TEST_EMAIL:{context}]`.
La marche a suivre complete est dans `test_execution_guide` (section « Test email markers ») :
la suivre. Ce skill ne garde que l'aide-memoire des outils.

**Ne jamais utiliser une autre boite courriel ni inventer une adresse.** Tout passe par les outils
`mcp__plugin_aitest_aitestlist__*` (jamais d'appel direct a l'API ni au fournisseur de courriel).

## Outils

| Pour | Outil et parametres |
|------|---------------------|
| Creer l'adresse (1re apparition du contexte) | `create_test_email(prefix="{context}_queue{queue_id}")` → `alias` |
| Attendre un courriel | `wait_for_email(alias, subject_contains="...", timeout=30)` (60 s maximum) → `found`, `email` |
| Lister les courriels recus | `list_emails(alias, limit=10, after=...)` |
| Lire un courriel (et ses liens) | `read_email(message_id)` → `subject`, `html`, `text`, `links` |
| Nettoyer (OBLIGATOIRE) | `delete_test_email(alias)` |

## Contexte → adresse

1. Premiere fois qu'un contexte apparait : `create_test_email`, puis retenir `context → alias`.
2. Meme contexte plus loin dans la file : reutiliser la MEME adresse.
3. `found: false` apres `wait_for_email` → la tache est en `echec`.
4. Ouvrir le lien de confirmation ou de reinitialisation (`links` de `read_email`) dans le navigateur.
5. `delete_test_email` pour chaque adresse creee, au `[TEARDOWN]` ou a la fin, meme apres des echecs.

En cas de refus d'acces a AI TestList (« Not connected », 401) : dire a l'utilisateur
« tapez `/mcp`, choisissez `aitestlist`, puis Authenticate » (forfait payant requis), puis STOP.
