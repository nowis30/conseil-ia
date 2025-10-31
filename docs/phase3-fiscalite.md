# Phase 3 — Fiscalité et rapport annuel

Cette phase ajoute la couche fiscale (impôt corporatif/personnel, dividendes, retours de capital, prêts d'actionnaires) et les exports requis (CSV et PDF). Ce guide décrit la mise en place et les vérifications attendues.

## 1. Pré-requis

- **Base de données** : Postgres (local ou Render). Exemple local : `postgresql://postgres:postgres@localhost:5432/nowis_local?schema=public`.
- **Variables d'environnement** :
  - `DATABASE_URL` : DSN Postgres.
  - `JWT_SECRET` : clé de signature JWT (ex. `openssl rand -base64 32`).
  - `NODE_ENV=production` en déploiement (Render).
  - `PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1` si Puppeteer n'est pas utilisé en build.

## 2. Migration & seed

```powershell
# depuis backend/
npx prisma migrate deploy
npm run seed
```

La migration `20251024114500_phase3_fiscal_module` crée les tables fiscales et l'enum `DividendType`. Le script `npm run seed` injecte :

- Dividende admissible 25 000 $.
- Retour de capital 10 000 $ (ajuste le PBR).
- Prêt actionnaire 50 000 $ + remboursement échelonné.
- Déclarations d'impôt corporatif/personnel (2024) avec soldes RDTOH/GRIP/CDA.

## 3. Endpoints ajoutés

### Dividendes & ROC (`/corporate`)

| Méthode | Route | Description |
| --- | --- | --- |
| `POST` | `/corporate/dividends` | Déclarer un dividende (admissible / non). |
| `GET` | `/corporate/dividends?year=2024` | Lister les dividendes déclarés (filtre optionnel par année). |
| `POST` | `/corporate/returns-of-capital` | Enregistrer un retour de capital (ajuste le PBR). |
| `GET` | `/corporate/returns-of-capital?year=2024` | Lister les retours de capital. |

### Fiscalité (`/tax`)

| Méthode | Route | Description |
| --- | --- | --- |
| `POST` | `/tax/corporate-tax` | Calcul / mise à jour de l'impôt corporatif (RDTOH, GRIP, CDA). |
| `POST` | `/tax/personal-tax` | Calcul d'impôt personnel (Québec + Fédéral). |
| `GET` | `/tax/exports/t5?year=2024` | Export CSV T5 (dividendes). |
| `GET` | `/tax/exports/rl3?year=2024` | Export CSV RL-3. |
| `GET` | `/tax/exports/t4?year=2024` | Export CSV T4. |
| `GET` | `/tax/exports/rl1?year=2024` | Export CSV RL-1. |
| `GET` | `/tax/annual-report?year=2024` | JSON consolidé (résumé + fiscalité + prêts). |
| `GET` | `/tax/annual-report/pdf?year=2024` | Rapport annuel PDF (États, fiscalité, résolutions). |

> Toutes les routes exigent l'authentification + rôle `ADMIN`.

## 4. Tests recommandés

1. **Backend**
   - `npm run lint`
   - `npm run build`
   - `npm run test` (une fois les tests mis à jour pour la phase 3).

2. **API**
   - Postman/Thunder Client : vérifier les réponses attendues (ex. `GET /tax/annual-report`).
   - Valider que les exports CSV contiennent les lignes seedées.

3. **Frontend** (à implémenter)
   - Écran dividendes / retour de capital : création, liste.
   - Module fiscal : génération des rapports, téléchargement CSV/PDF.
   - Surveillance des erreurs (notifications utilisateur).

## 5. Déploiement Render

1. Nettoyer le cache puis redeployer (`Clear build cache & Redeploy`).
2. Vérifier `Build Command` et `Start Command` :
   - Build : `npm ci --include=dev && npm run render-build`
   - Start : `npm run start:render`
3. Supprimer toute variable `PORT` côté Render (Render la gère automatiquement).
4. Vérifier l'endpoint `https://backend-nowis-1.onrender.com/health` et les logs (Prisma migrations).

## 6. Étapes frontend (prochain sprint)

- Ajouter les appels API correspondants (`/corporate/*`, `/tax/*`).
- Mettre à jour le state management (Zustand) pour intégrer les nouveaux modules.
- Ajouter les écrans / composants : saisie de dividendes, prêts, exports, rapport PDF.
- Assurer la gestion des authorisations (ADMIN only).
- Rédiger les tests E2E/UI pour les nouvelles fonctionnalités.

## 7. Checklist finale

- [ ] Migration déployée & seed exécuté.
- [ ] Routes testées (200 OK, payloads corrects).
- [ ] Rapport annuel PDF généré sans erreur (Puppeteer accessible sur Render).
- [ ] Documentation à jour (README, guides internes).
- [ ] Frontend planifié (tickets créés) ou implémenté.
- [ ] Déploiement Render/Vercel vérifié (auth, API, exports).

---

Suivez cette feuille de route pour valider et livrer la phase 3. Une fois la checklist complétée, la fonctionnalité est prête à être remise au CPA pour revue.
