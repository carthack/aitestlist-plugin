---
name: create-payment
description: Genere des taches de test pour les systemes de paiement (Stripe, PayPal). Detecte les fournisseurs dans le code du projet et cree un test de facturation dedie. Utilise par create-test quand un systeme de paiement est detecte.
user-invocable: false
---

# Create Payment Tests

Instructions pour un projet qui utilise un systeme de paiement.
Le contenu du test (marqueur `[PAYMENT_TEST]`, cartes de test Stripe, scenarios proposes,
exemple de description) est dans `test_writing_guide` (section « Payment tests ») : le suivre.
Ce skill garde seulement la detection dans le code, faite avec les outils de Claude Code.

## Detection dans le code (Grep, Glob)

**Stripe :**
- `import stripe` ou `stripe.` dans le code Python/JS
- Routes `/pricing`, `/billing`, `/checkout`, `/subscribe`
- Gabarits avec `stripe-elements`, `card-element`, `stripe.js`

**PayPal :**
- `paypalrestsdk` ou `paypal` dans le code Python
- SDK PayPal JS (`paypal-js`, `paypal.Buttons`)
- Routes contenant `paypal`

## Resultat

Un test dedie « Paiement et facturation » (dans `USER_LANG`), soumis avec `submit_test` comme les
autres, en utilisant les categories obtenues par `list_categories`.
Chaque tache de paiement commence par `[PAYMENT_TEST]` sur sa propre ligne.

En cas de refus d'acces a AI TestList (« Not connected », 401) : dire a l'utilisateur
« tapez `/mcp`, choisissez `aitestlist`, puis Authenticate » (forfait payant requis), puis STOP.
