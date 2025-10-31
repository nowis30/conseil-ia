# Plan de recette fonctionnelle

## Préparation
- Déployer l'API backend et l'UI frontend sur l'environnement de recette.
- Initialiser la base avec `npm run migrate:deploy` et `npm run seed` si besoin.
- Vérifier que les comptes démo fonctionnent :
  - `demo@nowis.local / Demo#2024!Nowis`
  - `admin@nowis.local / Admin#2024!Nowis`
  - `collab@nowis.local / Collab#2024!Nowis`
  - `invite@nowis.local / Invite#2024!Nowis`

## Scénarios prioritaires
1. **Authentification & rôles**
   - Connexion en ADMIN, COLLAB et INVITE.
   - Vérifier l’accès aux écrans restreints (`/companies`, `/reports`, gestions rôles).
   - Tester les erreurs 403 pour les rôles insuffisants.

2. **Import revenus CSV**
   - Importer un fichier valide (voir `docs/financial-exports.md`).
   - Vérifier le feedback UI et la création des lignes.
   - Importer un fichier avec erreurs pour vérifier la remontée des lignes invalides.

3. **Export dépenses fiscales**
   - Télécharger le JSON (`format` absent) et le CSV (`format=csv`).
   - Valider la cohérence des totaux et l’intégration dans l’UI.

4. **Synthèse immobilière**
   - Créer/éditer/supprimer une unité et une hypothèque sur un immeuble.
   - Vérifier la mise à jour temps réel du tableau de bord et du ratio LTV.
   - Télécharger les pièces jointes et vérifier le nettoyage côté backend.

5. **Synthèse corporate**
   - Créer une société avec actionnaires, classes, transactions, états financiers, résolutions.
   - Tester les liens profonds depuis le dashboard.
   - Exporter un état financier en JSON et vérifier les toasts de feedback.

6. **Rapports & PDF**
   - Vérifier `GET /api/reports/overview` depuis le dashboard.
   - Télécharger le PDF `nowis-bilan.pdf` et vérifier la structure des sections.

7. **Sécurité**
   - Tester le rate limiting (300 requêtes/15 min) sur `/api/*`.
   - Vérifier la politique de mot de passe renforcée (12 caractères, symboles, chiffres, casse).
   - Contrôler les en-têtes de sécurité (CSP, HSTS) via les devtools navigateur.

## Validation technique
- Contrôler les logs backend et frontend (console, APM si présent).
- Vérifier l’absence d’erreurs JS côté navigateur.
- Lancer `npm run lint`, `npm test`, `npm run build` en recette si compilation locale.

## Go/NoGo
- Documenter chaque scénario dans un rapport de recette.
- Valider la disponibilité du plan de rollback avant Go Live.
