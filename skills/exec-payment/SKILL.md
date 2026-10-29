---
name: exec-payment
description: Execute les taches de test de paiement (Stripe Elements, Stripe Checkout, PayPal sandbox) dans le navigateur. Verifie payment_testing (get_exec_mode) avant execution. Utilise par exec-test quand une tache contient [PAYMENT_TEST].
user-invocable: false
---

# Execute Payment Tests

Instructions pour les taches dont la description contient `[PAYMENT_TEST]`.
Les regles (verification de `payment_testing`, refus des cles `live`, cartes de test) sont dans
`test_execution_guide` (section « Payment tasks ») : les suivre. Ce skill ne garde que ce qui
est propre a Claude Code : remplir les champs Stripe avec les outils du navigateur.

## Verification avant execution

`payment_testing` vient de `mcp__plugin_aitest_aitestlist__get_exec_mode` (deja lu au debut de l'execution).
- `enabled` faux → `echec` ; `stripe_mode` ou `paypal_mode` = `live` → `erreur`. Messages exacts : voir le guide.

## Stripe Elements (iframes) avec Playwright

Chaque champ de carte est souvent dans son propre iframe
(`iframe[name*="__privateStripeFrame"]` ou `iframe[title*="Secure card"]`).

1. `browser_snapshot` : reperer les iframes et les champs (`cardnumber`, `exp-date`, `cvc`).
   Les champs des iframes apparaissent dans le snapshot avec leurs refs : `browser_type` sur ces refs.
2. Si un champ n'est pas accessible par ref, utiliser `browser_run_code` / `browser_run_code_unsafe` :
   ```javascript
   const card = page.frameLocator('iframe[name*="__privateStripeFrame"]').first();
   await card.locator('input[name="cardnumber"]').fill('4242 4242 4242 4242');
   ```
3. Revenir a la page principale pour cliquer sur le bouton de paiement.

Avec Claude dans Chrome : `find` pour reperer les champs, puis `form_input` ou `computer` (clic + saisie).

## Stripe Checkout et PayPal

- Stripe Checkout : la page passe sur `checkout.stripe.com` ; remplir les champs directement.
- PayPal : fenetre ou redirection vers `sandbox.paypal.com` ; `browser_tabs` pour changer d'onglet ;
  identifiants sandbox fournis dans la tache.

## Jamais

- Une vraie carte, des cles `live`, un numero de carte dans un commentaire ou un journal.

En cas de refus d'acces a AI TestList (« Not connected », 401) : dire a l'utilisateur
« tapez `/mcp`, choisissez `aitestlist`, puis Authenticate » (forfait payant requis), puis STOP.
