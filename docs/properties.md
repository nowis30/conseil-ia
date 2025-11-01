# Immeubles – Phase 1

Cette note couvre l'étendue livrée pour la gestion des immeubles dans la Phase 1 : modèles de données, routes backend et comportement de l'interface.

## Modèle de données

- `Property`
  - `id`, `name`, `address`, `acquisitionDate`, `purchasePrice`, `currentValue`, `notes`
- `PropertyUnit`
  - `id`, `propertyId`, `label`, `squareFeet`, `rentExpected`
- `PropertyMortgage`
  - `id`, `propertyId`, `lender`, `principal`, `rateAnnual`, `termMonths`, `amortizationMonths`, `startDate`, `paymentFrequency`, `paymentAmount`
  - `paymentAmount` est calculé automatiquement à l'enregistrement (formule d'annuité) et reflète le paiement périodique planifié.

Toutes les entités sont rattachées au propriétaire connecté via les vérifications Prisma.

## API backend

Toutes les routes sont protégées par l'auth middleware et vivent sous `/api`.

### Immeubles (CRUD de base)

- `GET /properties` : liste paginée complète (Phase 1 : liste complète).
- `POST /properties` : création d'un immeuble.
- `PUT /properties/:propertyId` : mise à jour.
- `DELETE /properties/:propertyId` : suppression Cascade (hypothèques + unités).

### Unités

- `GET /properties/:propertyId/units`
- `POST /properties/:propertyId/units`
- `PUT /properties/:propertyId/units/:unitId`
- `DELETE /properties/:propertyId/units/:unitId`

`POST/PUT` acceptent :

```json
{
  "label": "Appartement 101",
  "squareFeet": 850,
  "rentExpected": 1350
}
```

`label` est requis. `squareFeet` et `rentExpected` sont optionnels (`null`).

### Hypothèques

- `GET /properties/:propertyId/mortgages`
- `POST /properties/:propertyId/mortgages`
- `PUT /properties/:propertyId/mortgages/:mortgageId`
- `DELETE /properties/:propertyId/mortgages/:mortgageId`

`POST/PUT` attendent :

```json
{
  "lender": "Banque ABC",
  "principal": 250000,
  "rateAnnual": 0.045,
  "termMonths": 60,
  "amortizationMonths": 300,
  "startDate": "2024-01-01",
  "paymentFrequency": 12
}
```

Les champs numériques doivent être positifs. `termMonths`, `amortizationMonths` et `paymentFrequency` sont des entiers (> 0). Le backend calcule et persiste automatiquement `paymentAmount`; il ne faut plus l'envoyer depuis le frontend.

Endpoints Phase 2 :

- `POST /properties/:propertyId/mortgages/preview` : renvoie une simulation complète (paiement périodique, échéancier, agrégats annuels) sans créer d'enregistrement.
- `GET /properties/:propertyId/mortgages/:mortgageId/amortization` : retourne l'hypothèque sérialisée et son échéancier calculé (pour analyses/export).

## Interface Immeubles

Écran : `frontend/src/screens/Properties.tsx`.

- Tableau principal : liste les immeubles avec les totaux (dette, cashflow, etc.) dérivés du hook `useSummary`.
- Colonnes ajoutées : **Unités**, **Loyer potentiel**, **Dette en cours** et **Ratio LTV** (calculé à partir de la synthèse backend).
- Actions par ligne :
  - ✎ modifier l'immeuble
  - 🏢 ouvrir le dialogue **Unités**
  - 🏦 ouvrir le dialogue **Hypothèques**
  - 🗑️ supprimer l'immeuble
  - ⚙️ ouvrir les paramètres CCA
- Dialogue **Unités** :
  - Table récapitulative des unités existantes (formatage devise/superficie).
  - Formulaire d'ajout/modification avec validations côté client (libellé requis, numéros ≥ 0).
  - Suppression avec confirmation.
- Dialogue **Hypothèques** :
  - Table récapitulative des hypothèques.
  - Formulaire avec validations strictes (positifs, entiers pour durée/amortissement/fréquence, taux en pourcentage) et calcul automatique du paiement périodique.
  - Aperçu dynamique (paiement, échéancier sur 12 premières périodes, agrégats annuels, solde après le terme).
  - Messages d'erreur backend surfacent via `Alert`.

Toutes les mutations invalident `summary` et leurs caches respectifs pour rafraîchir l'écran principal.

## Tests

- `backend/src/__tests__/propertiesNestedRoutes.test.ts` couvre les scénarios CRUD + contrôles d'accès pour unités et hypothèques.
- Lancer `npm test` dans `backend/` pour vérifier.
- `backend/src/__tests__/summaryRoute.test.ts` atteste que `/api/summary` expose les agrégats (unités, loyer potentiel, dette, LTV, taux moyen).

## Guide QA manuel

1. **Préparer les données** : lancer `npm run seed` puis se connecter avec l'utilisateur `demo@nowis.local`.
2. **Immeubles** : créer un immeuble (libellé obligatoire), modifier la valeur actuelle, supprimer, vérifier la mise à jour du tableau et des totaux.
3. **Unités** :
  - Ouvrir le dialogue via l'icône 🏢.
  - Ajouter une unité avec superficie et loyer, confirmer l'apparition dans la liste et la mise à jour de la synthèse.
  - Modifier l'unité avec une valeur invalide (ex : `-10`) pour valider le message d'erreur, puis corriger et sauvegarder.
  - Supprimer l'unité après confirmation et vérifier le retrait.
4. **Hypothèques** :
  - Ouvrir le dialogue via l'icône 🏦.
  - Créer une hypothèque complète (taux saisi en pourcentage), vérifier l'affichage formaté (devise, date, %).
  - Tester la validation en saisissant une fréquence non entière, confirmer l'erreur, puis corriger.
  - Supprimer l'hypothèque et s'assurer que la synthèse se met à jour.
5. **CCA** : ouvrir le dialogue ⚙️, éditer les champs puis sauvegarder pour vérifier la cohérence des notifications.
6. **Tests automatiques** : exécuter `npm test` dans `backend/` et vérifier que la suite passe.

## Points à venir (Phase 2+)

- Gestion des pièces jointes (actes/fichiers hypothèques).
- Indicateurs dédiés par unité (taux d'inoccupation, loyer perçu vs attendu).

## Import fiscal locatif (T776/TP‑128) → Lignes Immeubles (auto)

Depuis l’ingestion IA des rapports d’impôt personnels, l’extracteur détecte les feuillets locatifs (T776/TP‑128) et crée automatiquement des éléments liés à l’immeuble correspondant.

Comportement:

- Association d’immeuble: tentative de match par nom/adresse; si aucun match fiable et qu’une adresse ou un nom est présent, l’immeuble est auto‑créé puis lié à l’état locatif.
- Création idempotente de 3 lignes agrégées par [immeuble, année, formulaire]:
  - Revenu ANNUEL: « T776 AAAA – Loyers bruts »
  - Revenu ANNUEL: « T776 AAAA – Autres revenus locatifs »
  - Dépense ANNUELLE: « T776 AAAA – Dépenses totales (agrégées) »
  (ou « TP128 AAAA – … » pour Québec)
- Idempotence: une ligne n’est créée que si aucune ligne avec le même `label` et la même `startDate` (01‑01‑AAAA) n’existe déjà pour l’immeuble.

Notes:

- Ces lignes servent de base; les catégories fines (intérêts, taxes, entretien, assurances, etc.) pourront être ventilées dans une phase suivante.
- Le ré‑import du même document ne dupliquera pas ces lignes (clé: label + startDate + propertyId).
