# Nowis IA – Phase 1

## Changements récents (octobre 2025)

- Correction complète des erreurs et avertissements ESLint (backend et frontend).
- Passage des scripts backend en modules ES (import/export).
- Ajout des flags React Router v7 pour anticiper la migration et supprimer les warnings.
- Tous les tests et builds sont au vert (Jest, Vitest, lint).
- Phase 4 (version gamma) : ajout d’un comité d’experts IA (fiscaliste, comptable, planificateur, avocat) avec écran « Conseillers IA » et orchestrateur backend.
- Instructions de lancement et de build validées sur Windows (PowerShell).

## Prochaines étapes

1. **Déploiement**
	- Backend : déployer sur le serveur cible, configurer `.env` et la base de données.
	- Frontend : builder puis uploader le dossier `dist` sur l’hébergement statique ou cloud.
	- Checklist détaillée dans `docs/deployment-checklist.md`.

2. **Recette et validation**
	- Tester l’application en conditions réelles (import/export CSV, gestion des rôles, navigation corporate).
	- Vérifier la sécurité (rate limiting, politique de mot de passe, accès API).
	- Plan de recette complet dans `docs/recette-plan.md`.

3. **Maintenance**
	- Surveiller les logs et les erreurs.
	- Mettre à jour la documentation si de nouvelles fonctionnalités sont ajoutées.
	- Préparer la migration Vite 7 (voir `docs/vite7-migration.md`).

Ce monorepo contient l'architecture de base pour la Phase 1 : API Node/Express + Prisma et frontend React/Vite.

## Structure

- `backend/` : API Express + Prisma (Postgres requis dès la phase 3).
- `frontend/` : UI React 18 + Vite, MUI, TanStack Query.
- `docs/` : ADR, guides phase 1-3, wireframes texte.

## Lancement rapide

```bash
# Backend
cd backend
npm install
npm run prisma:generate
npm run migrate:dev
npm run seed
npm run dev

# Frontend
cd ../frontend
npm install
npm run dev
```

Le frontend proxifie les requêtes `/api` vers `http://localhost:4000`.

## Qualité

- Lint : `npm run lint`
- Tests : `npm run test`
- Build : `npm run build`

## Variables d'environnement

Copier `.env.example` en `.env` dans `backend/` et ajuster `DATABASE_URL`, `JWT_SECRET`, `PORT`.

## Flux CSV et exports fiscaux

- Import revenus : `POST /api/revenues/import` avec un fichier CSV (`text/csv`). Colonnes attendues : `propertyId` ou `propertyName`, `label`, `amount`, `frequency`, `startDate`, `endDate`. Les lignes invalides sont ignorées et retournées dans `errors`.
- Export dépenses fiscales : `GET /api/expenses/export/fiscal?year=2024&format=csv` retourne le rapport structuré par immeuble/catégorie. Sans `format`, la réponse JSON inclut les totaux pour alimenter le frontend.
- Dans l'UI, l'écran **Dépenses** permet de lancer l'export CSV (sélection de l'année, feedback de succès/erreur). L'écran **Revenus** propose le dialogue d'import CSV avec validation et tableau récapitulatif.
- Détails complets (exemples, colonnes CSV) : `docs/financial-exports.md`.

## Immeubles (Phase 1)

- Gestion des immeubles (CRUD) avec calculs de synthèse reliés au dashboard.
- Dialogues **Unités** et **Hypothèques** depuis l'écran Immeubles pour enrichir la fiche d'un actif.
- Unités : libellé, superficie (pi²) et loyer attendu; la synthèse se met à jour immédiatement.
- Hypothèques : capital, taux annuel, durée, amortissement, fréquence et date de départ avec validations métier.
- Tableau portefeuille enrichi avec *Unités*, *Loyer potentiel*, *Dette en cours* et *Ratio LTV*.
- Exports CSV/PDF du bilan immobilier alimentés par `/api/summary/export/*`.
- Détails API/UI supplémentaires dans `docs/properties.md`.

## Corporate (Phase 2)

- Modèles Prisma pour sociétés, actionnaires, classes d'actions, transactions, états financiers et résolutions.
- Routes Express `/api/companies` fournissant un CRUD complet (actionnaires liés, classes, transactions, états, résolutions).
- Validation Zod pour sécuriser les payloads corporate (dates, décimales, énumérations).
- Services de synthèse corporate intégrés au dashboard (indicateurs consolidés, dernier état et dernière résolution).
- Tests Jest/Supertest couvrant les principaux flux corporatifs et les exports.

## Frontend Phase 2

- Écran **Companies** : gestion complète (CRUD sociétés, actionnaires, classes, transactions, états financiers et résolutions).
- États financiers avec lignes détaillées, affichage extensible, création/édition dans des dialogues dédiés.
- Liens profonds depuis le dashboard (query params `companyId`, `statementId`, `resolutionId`) avec surbrillance et toasts.
- Actions rapides : copie de liens partageables, export JSON de chaque état, toasts de réussite/erreur uniformes.
- Dashboard consolidé (React Query + MUI) avec navigation directe vers les entités corporate.


## Exemples d’exports

### CSV (extrait)

```
"Immeuble","Unités","Loyer potentiel","Dette en cours","Ratio LTV (%)","Revenus","Dépenses","Service de la dette","Intérêts","Capital","Cashflow net","CCA","Équité"
"Immeuble Centre-Ville","10","15000.00","900000.00","64.29","180000.00","24000.00","54000.00","36000.00","18000.00","102000.00","12000.00","480000.00"
"TOTAL","10","15000.00","900000.00","64.29","180000.00","24000.00","54000.00","36000.00","18000.00","102000.00","12000.00","480000.00"

"Synthèse corporate"
"Indicateur","Valeur"
"Sociétés actives","1"
"Actionnaires suivis","2"
"Classes d'actions","1"
"Transactions d'actions","2"
"Valeur des transactions (FMV)","100000.00"
"Contrepartie reçue","100000.00"
"États financiers","1"
"Résolutions consignées","1"
"Actifs consolidés","125000.00"
"Capitaux propres consolidés","90000.00"
"Bénéfice net consolidé","50000.00"
"Dernier état financier","Compagnie Familiale Tremblay · 2024-12-31 · ANNUAL"
"Dernière résolution","Compagnie Familiale Tremblay · 2025-03-15 · ANNUAL_MEETING"

"États financiers (synthèse)"
"Société","Type","Période début","Période fin","Audité","Actifs","Passifs","Capitaux propres","Revenus","Dépenses","Bénéfice net","Métadonnées"
"Compagnie Familiale Tremblay","ANNUAL","2024-01-01","2024-12-31","Oui","125000.00","35000.00","90000.00","200000.00","150000.00","50000.00","Structure corporative beta pour la phase 2."

"États financiers (détail des lignes)"
"Société","Type","Période fin","Catégorie","Libellé","Montant","Métadonnées"
"Compagnie Familiale Tremblay","ANNUAL","2024-12-31","ACTIF","Trésorerie","45000.00",""
"Compagnie Familiale Tremblay","ANNUAL","2024-12-31","PASSIF","Dette bancaire","35000.00",""

"Résolutions corporatives"
"Société","Type","Date","Titre","Contenu","Métadonnées"
"Compagnie Familiale Tremblay","ANNUAL_MEETING","2025-03-15","Approbation des états financiers","Les administrateurs approuvent les états financiers annuels.",""
```

### PDF

Le PDF reprend les mêmes sections, avec un rendu tabulaire et des styles harmonisés (titres, sous-titres, couleurs, alignement des montants).

## Cas d’usage corporate

- **CRUD sociétés** : création, édition, suppression de sociétés avec validation des champs (nom, NEQ, province, année fiscale).
- **Actionnaires** : ajout/suppression, gestion des rôles et pourcentages de vote, synchronisation avec les transactions d’actions.
- **Classes d’actions** : création, édition, gestion des droits de vote et de croissance.
- **Transactions d’actions** : enregistrement des mouvements (achat, vente, transfert), calculs de FMV et contrepartie.
- **États financiers** : création, édition, ajout de lignes détaillées, affichage extensible, export JSON individuel.
- **Résolutions** : création, édition, suppression, affichage et export.
- **Navigation profonde** : accès direct à une société, un état ou une résolution via l’URL (`companyId`, `statementId`, `resolutionId`), surbrillance et toasts.

## Système de rôles et permissions

### Modèles
- **Role** : définit un type de rôle (ex : ADMIN, COLLAB, INVITE)
- **UserRole** : relie un utilisateur à un rôle, éventuellement pour une entreprise donnée

### Endpoints API
- `/api/roles` : CRUD des rôles
- `/api/userRoles` : CRUD des assignations de rôles aux utilisateurs

### Middleware de contrôle d’accès
- `requireRole('ADMIN')` : protège une route, accès réservé aux utilisateurs ayant le rôle ADMIN
- Exemple : la création d’entreprise (`POST /api/companies`) est réservée aux ADMIN

### Utilisation
Pour protéger une route :
```ts
import { requireRole } from '../middlewares/requireRole';

router.post('/chemin', requireRole('ADMIN'), (req, res) => { ... });
```

### Attribution des rôles
- Les rôles sont attribués via l’API `/api/userRoles` (POST, PUT, DELETE)
- Un utilisateur peut avoir plusieurs rôles (global ou par entreprise)

### Vérification côté backend
- Le middleware vérifie le rôle avant d’exécuter la logique métier
- Si l’utilisateur n’a pas le rôle requis, la réponse est 403 Forbidden

### Exemples de rôles
- `ADMIN` : gestion complète
- `COLLAB` : accès collaboratif
- `INVITE` : accès limité

### Seed et tests
- Des exemples de seed et de tests sont disponibles dans `backend/src/scripts/seed.ts` et les fichiers de test.

## Phase 3 – Multi-utilisateurs, fiscalité et sécurité
- **Phase 4 – IA multi-expert**
	- Nouvel écran « Conseillers IA » (frontend) proposant un dialogue guidé.
	- Service backend simulant quatre modules d’expertise et un coordinateur pour fusionner les conseils.
	- Indicateurs cliquables avec explications détaillées et pistes d’action générées dynamiquement.
	- Sélection du moteur via `ADVISOR_ENGINE` (`heuristic` par défaut, `gpt` active OpenAI). Pour `gpt`, configurez `OPENAI_API_KEY` et (optionnellement) `OPENAI_MODEL` — par défaut `gpt-4.1`. Si vous définissez `OPENAI_BASE_URL`, utilisez `https://api.openai.com/v1` (ou laissez vide — le backend corrige aussi automatiquement l’absence de `/v1`). Sur le frontend, `VITE_ADVISOR_ENGINE` relaie la préférence.


- **Gestion de l'équipe** : écran `Team` (frontend) affichant les utilisateurs et leurs rôles, avec invitation d'un nouvel utilisateur via mot de passe provisoire conforme aux règles de sécurité.
- **Reporting avancé** : nouvelle route `GET /api/reports/overview` et écran `Reports` fournissant un tableau de bord consolidé (répartition des rôles, top sociétés par équité, activités récentes).
- **Renforcement sécurité** : politique de mot de passe renforcée (12 caractères, casse mixte, chiffres et symboles) et rate limiting global sur les routes `/api`.
- **API utilisateurs** : routes `/api/users`, `/api/roles`, `/api/userRoles` protégées par le rôle `ADMIN` pour piloter les permissions.
- **Credentials de démonstration** :
	- `demo@nowis.local / Demo#2024!Nowis`
	- `admin@nowis.local / Admin#2024!Nowis`
	- `collab@nowis.local / Collab#2024!Nowis`
	- `invite@nowis.local / Invite#2024!Nowis`
- **Fiscalité** : nouveaux modules `/corporate/*` et `/tax/*` pour les dividendes, retours de capital, impôt corporatif/personnel, exports CSV (T5, RL-3, T4, RL-1) et rapport annuel PDF. Voir `docs/phase3-fiscalite.md`.
