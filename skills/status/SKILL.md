---
name: status
description: Verifie la connexion a AI TestList (serveur MCP), le forfait, le navigateur disponible (Playwright ou Claude dans Chrome) et la configuration d'execution. Affiche un tableau de diagnostic.
argument-hint:
disable-model-invocation: true
---

# Status Check

Diagnostic de la connexion AI TestList et des dependances. Les outils d'AI TestList s'appellent
`mcp__plugin_aitest_aitestlist__<outil>`. N'appelle jamais l'API toi-meme (curl, Python) : passe par les outils.

## Etape 1: Connexion et forfait

Appeler `get_account`.

- Reponse → Connexion = OK ; noter `user`, `plan_code`, `language`, `ai_integration_enabled`, `exec_test_enabled`.
- Outil absent, « Not connected », 401 ou acces refuse → Connexion = FAIL, avec la consigne :
  « tapez `/mcp`, choisissez `aitestlist`, puis Authenticate » (forfait payant requis). Passer a l'etape 3.

## Etape 2: Mode d'execution et paiements

Appeler `get_exec_mode`. Afficher `exec_mode` et `payment_testing` (`enabled`, `stripe_mode`, `paypal_mode`).

## Etape 3: Navigateur

Verifier qu'un outil de navigateur est disponible :
- Playwright : un appel simple `mcp__plugin_playwright_playwright__browser_snapshot` (ou `browser_tabs`).
- Sinon Claude dans Chrome : `mcp__claude-in-chrome__tabs_context_mcp`.

Aucun des deux → Navigateur = MISSING (« Installez le greffon Playwright : `/plugin install playwright@claude-plugins-official` »).

## Etape 4: Mode teams

```bash
cat ~/.claude/settings.json 2>/dev/null
```

Chercher `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` dans la cle `env`.

## Affichage

```
=== AI TestList Status ===

| Composant       | Statut | Detail                           |
|-----------------|--------|----------------------------------|
| Connexion       | OK     | Stephane (aitestlist.com/mcp)    |
| Forfait         | OK     | pro — IA oui, execution oui      |
| Langue          | OK     | fr                               |
| Navigateur      | OK     | Playwright                       |
| Mode exec       | OK     | interactive_browser_minimal      |
| Tests paiement  | OFF    | Desactives                       |
| Mode teams      | OFF    | Non configure                    |
```

Icones de statut :
- OK = vert
- FAIL/MISSING = rouge
- OFF = gris (pas une erreur, juste desactive)
