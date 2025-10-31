# Module Effet de levier intelligent

## Objectifs
- Simuler l\'emprunt (hypothèque, marge de crédit, prêt corporatif) et son impact sur le cashflow, la fiscalité et la valeur nette.
- Comparer le coût réel de la dette (intérêts après impôt) avec le rendement attendu des placements (bourse, immeubles, comptes enregistrés).
- Fournir des recommandations chiffrées et contextualisées à l\'IA conversationnelle (`CoordinationAI`).

## Flux fonctionnel
1. **Saisie** (UI ou requête IA): "Emprunter 60 000 $ à 5 % amorti sur 5 ans pour investir dans VFV".
2. **Analyse AI**:
   - Détecte `montant=60000`, `taux=5%`, `source=maison`, `durée=5 ans`, `objectif=placement boursier`, `cible=VFV`.
   - Valide la capacité d\'endettement via `ProfileSummaryService` (ratio service de la dette, LTV, DSCR).
3. **LeverageEngine**:
   - Calcule intérêts annuels, capital remboursé, déductibilité.
   - Appelle `InvestmentEngine` pour obtenir rendement attendu (moyenne historique, volatilité).
   - Calcule rendement net après impôt: `(Rendement brut - Coût net de la dette) * (1 - taux d\'imposition applicable)`.
4. **Fiscalité** (`PersonalTaxEngine` ou `CorpTaxEngine`): ajuste l\'impôt en fonction de la déductibilité des intérêts et des gains en capital.
5. **Résultat AI**: répond avec cashflow annuel, valeur nette projetée, recommandation.

## Structure de données (proposition Prisma)
```prisma
model LeverageScenario {
  id                Int       @id @default(autoincrement())
  userId            Int
  companyId         Int?      // si scénario corporatif
  label             String
  sourceType        LeverageSourceType // HOME_EQUITY | DUPLEX | HELOC | CORPORATE_LOAN
  principal         Decimal
  rateAnnual        Decimal
  termMonths        Int       // durée du prêt
  amortizationMonths Int?
  startDate         DateTime
  interestDeductible Boolean @default(false)
  investmentVehicle LeverageInvestmentVehicle // ETF | DIRECT_STOCK | REALESTATE | BUSINESS
  expectedReturnAnnual Decimal
  expectedVolatility Decimal?
  planHorizonYears  Int
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt

  user    User      @relation(fields: [userId], references: [id])
  company Company?  @relation(fields: [companyId], references: [id])
}
```

### Enums
```prisma
enum LeverageSourceType {
  HOME_EQUITY
  RENTAL_PROPERTY
  HELOC
  CORPORATE_LOAN
}

enum LeverageInvestmentVehicle {
  ETF
  STOCK
  REALESTATE
  BUSINESS
  FUND
}
```

## Services à créer
- `src/server/services/leverageService.ts`
  - `evaluateScenario(input)` → renvoie cashflow, IRR, valeur nette projetée, ratio dette/revenu.
  - `listScenarios(userId)` → historique, comparaisons.
- `src/server/routes/leverage.ts`
  - POST `/api/leverage/simulate`
  - GET `/api/leverage/scenarios`
- `src/server/routes/ai.ts`
   - POST `/api/ai/leverage` → orchestrateur `CoordinationAI` (résumé narratif + persist optionnelle)
- `frontend/src/api/leverage.ts`
   - `useLeverageConversation` + `useLeverageScenarios` (React Query)
- `frontend/src/screens/Leverage.tsx`
   - Formulaire interactif, narration IA et historique des scénarios

### Orchestrateur IA (Phase 3)

- `src/server/services/ai/coordinationAI.ts`
   - `runLeverageConversation(input)` appelle `evaluateScenario`, optionnellement `saveScenario`, puis fournit un récit (`narrative`) et des `highlights` prêts à être relayés à la couche conversationnelle.
   - Utilise un formatage déterministe (`Intl.NumberFormat`) pour rester aligné avec la voix fr-CA.
- Tests associés : `src/__tests__/coordinationAI.test.ts` (sans persistance et avec sauvegarde).

### Contrat API (draft)

**POST** `/api/leverage/simulate`
- **Auth**: Bearer (JWT rendu par `/api/auth/login`).
- **Payload** (`application/json`):
   ```json
   {
      "label": "Investir VFV",
      "sourceType": "HOME_EQUITY",
      "principal": 60000,
      "annualRate": 0.05,
      "termMonths": 60,
      "amortizationMonths": 300,
      "startDate": "2025-11-01",
      "investmentVehicle": "ETF",
      "expectedReturnAnnual": 0.07,
      "expectedVolatility": 0.18,
      "planHorizonYears": 10,
      "interestDeductible": true,
      "marginalTaxRate": 0.47,
      "companyId": null,
      "save": true
   }
   ```
- **Réponse** `200 OK`:
   ```json
   {
      "summary": {
         "annualDebtService": 4209.05,
         "annualInterestCost": 2971.90,
         "afterTaxDebtCost": 1575.11,
         "expectedInvestmentReturn": 4200,
         "netExpectedDelta": 2624.89,
         "cashflowImpact": -9.05,
         "breakEvenReturn": 0.0263,
         "details": {
            "monthlyPayment": 350.75,
            "principalRepaidYearOne": 1237.14
         }
      },
      "savedScenarioId": 42
   }
   ```
- **Validation**: gérée par `zod`, messages d'erreur localisés en français.
- **Logging**: `logger.info` avec `userId`, `principal`, `annualRate`, `expectedReturnAnnual`, `sourceType`.
- **Postman**: `docs/api/leverage.postman_collection.json` (variables `baseUrl`, `token`).

**POST** `/api/ai/leverage`
- **Payload**: identique à `/api/leverage/simulate` avec `save` optionnel.
- **Réponse** `200 OK`:
   ```json
   {
      "summary": { /* mêmes métriques que simulate */ },
      "narrative": "Le scenario projette un flux...",
      "highlights": [
         "Service de la dette: ...",
         "Gain net attendu..."
      ],
      "savedScenarioId": 42
   }
   ```
- **Tests**: `src/__tests__/coordinationRoutes.test.ts` + `src/__tests__/coordinationAI.test.ts`.
- **UI**: `frontend/src/screens/Leverage.tsx` (+ navigation via `/leverage`).

**GET** `/api/leverage` (Protection JWT identique)
- **Réponse**: liste des enregistrements `LeverageScenario` triés par `createdAt desc`.
- **TODO**: exposer champ `notes`, surfaces calculs dérivés (cashflow projeté) pour l'UI.

## Intégrations existantes
- `ProfileSummaryService`: accès aux revenus, dettes, actifs pour ratio et validation.
- `InvestmentService`: calcul du rendement moyen (utiliser historiques existants ou seed).
- `PersonalTaxEngine` (à développer) : détermination du taux marginal réel et économie d\'impôt sur les intérêts.

## Étapes d\'implémentation
1. **Phase 1 – Base de données & service**
   - Ajouter modèle Prisma + migration.
   - Créer `leverageService.evaluateScenario` (calculs simples : intérêts, rendement net, cashflow).
   - Ajouter tests unitaires (nouveau fichier `src/__tests__/leverageService.test.ts`).
2. **Phase 2 – API REST**
   - Route `/api/leverage/simulate` qui valide l\'input (Zod) et renvoie un rapport JSON.
   - Autorisation : utilisateur authentifié.
   - Logging pino (info sur montant, ratio).
   - Documenter la réponse type et ajouter exemple Postman (✅ `docs/api/leverage.postman_collection.json`).
   - Exposer `/api/ai/leverage` pour la narration IA (✅ tests backend).
   - Intégrer l'écran frontend (`/leverage`) avec narration et historique (✅ React Query + MUI).
3. **Phase 3 – IA & orchestrateur**
   - Ajouter un handler dans `CoordinationAI` (à créer) qui mappe les entrées conversationnelles vers `leverageService`.
   - Format de réponse IA (`advisor`): narrative + chiffres.
4. **Phase 4 – UI & Tests end-to-end**
   - Écran React Native: formulaire minimal + résumé.
   - Commande dans `smoke.ts` (? option future) pour valider le simulateur via API.
5. **Phase 5 – Optimisations**
   - Monte Carlo / scénarios multiples.
   - Comparaison "rembourser dette vs investir".

## Questions / prochaines décisions
- **Taux d\'imposition effectif**: utiliser le taux marginal ou recombiner via simulation fiscale complète ?
- **Historique des rendements**: se baser sur des séries stockées (seed) ou requêtes externes ?
- **Gestion du risque**: inclure un stress test (±2 % sur le rendement, +1 % sur le taux d\'intérêt).
- **Planification IA**: doit-on stocker les requêtes conversationnelles pour audit ? (champ `prompt`, `interprétation`).

## Livrables immédiats
- Migration Prisma + modèle `LeverageScenario`.
- Service `leverageService.evaluateScenario` avec tests.
- Route `/api/leverage/simulate` documentée dans Swagger/Postman.
- Exemple de réponse pour l\'IA dans `docs/ai/responses.md`.
