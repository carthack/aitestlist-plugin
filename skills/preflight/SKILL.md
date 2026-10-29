---
name: preflight
description: Verification prealable centralisee pour les agents du greffon aitest. Verifie la connexion au serveur MCP d'AI TestList (get_account), le forfait et la langue de l'utilisateur. Precharge dans les agents via le champ skills.
user-invocable: false
---

# Preflight

Verification prealable centralisee. Les agents du greffon l'executent en premiere etape.

## Connexion : rien a configurer

Le greffon branche Claude Code sur le serveur MCP d'AI TestList (`https://aitestlist.com/mcp`).
Les outils s'appellent `mcp__plugin_aitest_aitestlist__<outil>` (ex. `mcp__plugin_aitest_aitestlist__get_account`).

- **Aucune cle a copier.** L'utilisateur se connecte une fois avec son compte AI TestList :
  dans Claude Code, `/mcp`, choisir `aitestlist`, puis « Authenticate ». Le site ouvre une page « Autoriser ».
- N'appelle **jamais** l'API d'AI TestList toi-meme (curl, Python, fetch) : elle refuse tout appel
  qui ne vient pas du serveur MCP. Passe toujours par les outils.

## Etape unique : get_account

Appeler `get_account` (aucun parametre). Il retourne `user`, `email`, `plan_code`,
`ai_integration_enabled`, `exec_test_enabled` et `language` (`fr` ou `en`).

**Cas d'erreur — STOP, ne pas contourner :**
- Outil absent, « Not connected », erreur 401 ou acces refuse →
  « Connexion a AI TestList requise : tapez `/mcp`, choisissez `aitestlist`, puis Authenticate. »
  La page « Autoriser » exige un forfait payant ; si elle refuse, l'utilisateur doit changer de forfait sur aitestlist.com.
- `ai_integration_enabled` faux → « Votre forfait AI TestList n'inclut pas l'integration IA. »
- Pour l'execution seulement : `exec_test_enabled` faux → « Votre forfait n'inclut pas l'execution des tests. »

## Resultat

| Variable | Exemple | Source |
|----------|---------|--------|
| `USER_LANG` | `fr` | `language` de `get_account` (defaut `fr`) |
| `PLAN` | `pro` | `plan_code` de `get_account` |

Tout ce que l'agent ecrit pour AI TestList (tests, commentaires, rapports) est dans `USER_LANG`,
quelle que soit la langue de la conversation.

## Usage

Ce skill est precharge dans les agents via le champ `skills:` de leur frontmatter.
Les agents n'ont pas besoin de « l'appeler » : ils executent l'etape directement.
