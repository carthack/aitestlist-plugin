# AI TestList pour Claude Code

*English below.*

AI TestList : créer, exécuter et rapporter des tests avec votre assistant IA.

- **`@test-creator`** analyse le code de votre projet et soumet des tests à AI TestList (vous les approuvez sur le site).
- **`@test-executor`** exécute une file d'exécution approuvée dans un navigateur et envoie chaque résultat en direct.
- **`@test-reporter`** analyse les échecs d'un projet et produit un rapport d'erreurs PDF.
- **`/aitest:status`** vérifie la connexion, le forfait et le navigateur.

## Installation

**Forfait payant AI TestList requis.**

1. Ajoutez le marché, puis installez le greffon :

   ```
   /plugin marketplace add carthack/aitestlist-plugin
   /plugin install aitest@aitestlist-plugin
   ```

2. Connectez votre compte : tapez `/mcp`, choisissez `aitestlist`, puis **Authenticate**.
   AI TestList ouvre la page « Autoriser » : acceptez. Aucune clé à copier.
3. Pour exécuter des tests, il faut un navigateur : le greffon Playwright
   (`/plugin install playwright@claude-plugins-official`) ou Claude dans Chrome.

Si un agent signale un refus d'accès, refaites `/mcp` → `aitestlist` → **Authenticate**.

## Mises à jour

```
claude plugin update aitest@aitestlist-plugin
```

---

# AI TestList for Claude Code

AI TestList: create, run and report tests with your AI assistant.

- **`@test-creator`** analyses your project's code and submits tests to AI TestList (you approve them on the site).
- **`@test-executor`** runs an approved execution queue in a browser and reports each result live.
- **`@test-reporter`** analyses a project's failures and produces a PDF error report.
- **`/aitest:status`** checks the connection, the plan and the browser.

## Install

**A paid AI TestList plan is required.**

1. Add the marketplace, then install the plugin:
   `/plugin marketplace add carthack/aitestlist-plugin` then `/plugin install aitest@aitestlist-plugin`
2. Connect your account: type `/mcp`, choose `aitestlist`, then **Authenticate**, and accept on the
   AI TestList "Authorize" page. No key to copy.
3. To run tests you need a browser: the Playwright plugin
   (`/plugin install playwright@claude-plugins-official`) or Claude in Chrome.

If an agent reports that access is refused, run `/mcp` → `aitestlist` → **Authenticate** again.

Update: `claude plugin update aitest@aitestlist-plugin`.

---

© Solutions Codexalia Inc. — [aitestlist.com](https://aitestlist.com)
