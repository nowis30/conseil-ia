# Engine contracts backlog

Ce document suit les travaux techniques a planifier autour des moteurs relies aux contrats exposes dans `backend/src/server/engines/contracts.ts`.

## Advisor engine
- [ ] Harmoniser les questions avec les nouvelles donnees profil (`profileSummaryService`) et mettre a jour les tests d integration `src/__tests__/advisorsRoute.test.ts`.
- [ ] Externaliser les poids heuristiques dans un fichier de configuration charge par le moteur pour faciliter la calibration sans redeploiement.
- [ ] Ajouter la persistance optionnelle des echanges (table `AdvisorConversation`) et brancher le coordinator pour rejouer un scenario.
- [ ] Implementer la bascule automatique entre mode heuristique et GPT suivant les flags d environnement et une verification de quotas.

## Tax engine
- [ ] Mutualiser le schema d entree corporate/personal (`TaxEngineContract`) et dedier des validateurs Zod pour securiser l API `/tax/*`.
- [ ] Etendre `corporateTaxEngine` aux autres provinces (profils de taux et calculs GRIP/RDTOH specifiques) et couvrir par des snapshots Jest.
- [ ] Introduire le calcul des acomptes provisionnels et consolider la sortie dans `annualReportService`.
- [ ] Migrer les resultats Prisma vers des vues intermediaires (DTO) pour eviter les conversions `toNumber` repetees.

## Valuation engine
- [ ] Brancher `ValuationEngineContract.compute` sur un entrepot de donnees supportant plusieurs periodes (stocker `valuationDate` timezone aware).
- [ ] Ajouter la simulation de sensibilite (variation +/- 5 % des valeurs) et exposer la matrice via l API PDF.
- [ ] Unifier la generation CSV/PDF (`pdfService`) avec les DTO du moteur et couvrir par des tests de regression.
- [ ] Prevoir un cache Redis court terme pour les valuations recalculees dans la meme requete.

## Wealth engine
- [x] Alimenter le snapshot automatique (`persistSnapshot`) via un job planifie et historiser dans `familyWealthSnapshot`.
- [ ] Supporter des hypotheses mensuelles dans les scenarios (au lieu d une granularite annuelle) et mettre a jour `runFamilyWealthScenario`.
- [ ] Ajouter un export JSON des stress tests pour integrer les deltas dans le dashboard IA.
- [ ] Couvrir les fonctions principales par des tests unitaires (mock Prisma) pour fiabiliser les projections.

## Leveraged finance engine
- [ ] Brancher le moteur sur une source de taux externe (table `InterestRateSnapshot`) avant normalisation.
- [ ] Permettre la simulation multi-pret dans `calculateLeveragedBuyback` et enrichir le DTO avec un calendrier de flux.
- [ ] Generer des pieces PDF en s appuyant sur `buildLeveragedBuybackResolution` avec essais snapshot.
- [ ] Stocker les hypotheses et resultats dans `leveragedBuybackScenario` via un service dedie pour isoler Prisma du routeur.

## Succession engine
- [x] Implementer le calcul complet de gel (valeurs preferentielles, impots latents, redemptions) dans un module `successionCalculator` consomme par le contrat.
- [ ] Enrichir `FreezeSimulationResult` avec un suivi annuel et le brancher au PDF plan de succession.
- [x] Ajouter la validation schema (Zod) pour les entrees `FreezeSimulationInput` et securiser l API `/freeze`.
- [ ] Preparer des tests de scenario fin-a-fin incluant fiducies et beneficiaries (mock Prisma).
- [x] Fournir un rapport de progression (étapes guidées) pour alimenter la discussion IA et rassurer l'utilisateur.

## Profile engine
- [ ] Exposer `ProfileEngineContract.buildSummary` via un service dedie afin de mutualiser l usage entre `/profile/summary` et `/profile/dashboard`.
- [ ] Etendre `deriveProfileInsights` avec une librairie de regles modulaires (fichier JSON ou base de donnees) chargee au demarrage.
- [x] Ajouter la projection sur 12 mois (cashflow vs objectifs) dans le payload dashboard pour alimenter la future IA proactive.
 - [x] Instrumenter le moteur (logs structures + metrics Prometheus) pour suivre les temps de calcul et les alertes (succession exposé via /api/metrics).

## Registry et orchestration
- [x] Introduire un module `backend/src/server/engines/registry.ts` exposant `getEngineRegistry` et `getEngine` pour aggreger les moteurs.
- [ ] Charger la version des moteurs depuis `package.json` ou configuration centrale.
- [ ] Ajouter des hooks d audit (logs + metrics) lors des invocations de chaque moteur (succession instrumente, etendre aux autres).
- [ ] Permettre l enregistrement dynamique de moteurs tiers (extension du registry via configuration).
