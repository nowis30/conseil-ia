# Checklist de déploiement Nowis IA

## Pré requis
- Node.js 20+ et npm 10+ installés sur la machine de build
- Accès à la base de données (SQLite locale ou URL distante)
- Variables d'environnement produites : `DATABASE_URL`, `JWT_SECRET` (ne définissez pas `PORT` sur Render)

## Suivi d'avancement
- [x] CI & lint backend stables (GitHub Actions, npm run lint/test sur main)
- [x] Sanity check production (script `tmp/manualSanityCheck` exécuté sur build dist)
- [x] Revue des secrets et variables d'environnement (section README mise à jour)
- [x] Automatiser `prisma migrate deploy` lors du démarrage Render (`start:render` lance `dist/scripts/startRender.js`, validé en prod Render)
- [x] Définir les smoke tests post-déploiement (script ou checklist dédiée)
- [x] Documenter le plan de rollback et la stratégie de monitoring

## Backend (Render)
1. Copier `.env.example` en `.env` et renseigner les secrets.
2. Installer les dépendances :
   ```powershell
   cd backend
   npm install
   ```
3. Générer Prisma et appliquer les migrations :
   ```powershell
   npm run prisma:generate
   npm run migrate:deploy
   ```
4. (Optionnel) Recréer le jeu de données de démo :
   ```powershell
   npm run seed
   ```
5. Vérifier qualité et tests :
   ```powershell
   npm run lint
   npm test
   ```
6. Construire les sources TypeScript :
   ```powershell
   npm run build
   ```
7. Render (hébergement conseillé)
    - Créer un dépôt GitHub pour `backend` puis pousser `main`.
    - Sur https://dashboard.render.com/ > New > Web Service > connecter le dépôt.
    - Node Version: auto (engines `>=18 <=22` dans package.json).
   - Build Command: `npm ci --include=dev && npm run render-build` (assure l’installation des devDependencies pour compiler TypeScript).
    - Start Command: `npm run start:render` (exécute `prisma migrate deploy` puis démarre l'API).
    - Variables d’environnement à ajouter:
       - `DATABASE_URL`: recommandé = Postgres Render (Free). Sinon démo: `file:./prisma/dev.db` (non persistant en prod).
       - `JWT_SECRET`: chaîne forte de 32+ caractères.
       - `NODE_ENV`: `production`.
       - `POST_TO_LEDGER_DEFAULT`: `true` pour publier par défaut les écritures comptables lors des imports/ingestions IA (recommandé pour tester en ligne la propagation). Mettre `false` pour ne publier que sur demande.
       - (optionnel) `OPENAI_API_KEY`: requis pour l’extraction IA des PDF (import fiscal).
       - (optionnel) `REDIS_URL`: pour activer la persistance des événements (ex: `redis://default:<pwd>@<host>:6379`). Sans cette variable, le bus d’événements reste en mémoire.
   - Ne PAS définir `PORT`. Render injecte automatiquement `PORT` et attend que l’application écoute dessus, sur l’hôte `0.0.0.0`.
   - Si Render n’installe pas les devDependencies: ajoutez `NPM_CONFIG_PRODUCTION=false` dans l’environnement (ou utilisez la commande de build ci-dessus qui force `--include=dev`).
    - Déployer et vérifier `/health` renvoie `{ status: 'ok' }`.

### Obtenir et configurer l'URL Postgres (DATABASE_URL)

- Si la base existe déjà sur Render:
   1. Dashboard Render > Databases > sélectionne ta base.
   2. Onglet Connections.
   3. Copie « Internal Database URL » (recommandé pour un service Render dans la même région) ou « External Database URL » (inclut `sslmode=require`).
   4. Dans ton service backend > Settings > Environment, ajoute/édite `DATABASE_URL` avec cette valeur.

- Si tu n'as pas encore de base:
   1. Dashboard Render > New > PostgreSQL > plan Free > choisis un nom et la même région que le backend.
   2. Une fois créée, va dans l’onglet Connections.
   3. Copie l'« Internal Database URL » (ou l'« External ») et mets-la dans `DATABASE_URL` du service backend.
   4. Redéploie (Clear build cache & Redeploy).

Notes:
- Préfère l’Internal Database URL entre services Render (latence et sécurité). L’External inclut déjà `?sslmode=require`.
- Prisma appliquera automatiquement les migrations au démarrage via `npm run start:render`.

## Frontend (Vercel)
1. Installer les dépendances :
   ```powershell
   cd ../frontend
   npm install
   ```
2. Vérifier lint et tests :
   ```powershell
   npm run lint
   npm test
   ```
3. Construire :
   ```powershell
   npm run build
   ```
4. Vercel
   - `vercel.json` (déjà en place): framework Vite, output `dist`, build `npm run build`.
   - Déployer via GitHub (auto). 
   - Variables d’environnement: `VITE_API_URL` = URL Render du backend.
   - SPA fallback automatique avec Vercel + Vite, sinon ajouter réécritures si besoin.

## Recette fonctionnelle
- Tester l'authentification avec les comptes démo.
- Vérifier l'import CSV revenus et l'export fiscal (JSON + CSV) depuis l'UI.
- Valider le dashboard corporate (CRUD sociétés, actionnaires, états financiers, résolutions, transactions).
- Contrôler les modules Immeubles (unités, hypothèques, amortissement) et le téléchargement des pièces jointes.
- Confirmer la génération du PDF de synthèse (`/api/summary/export/pdf`).

## Post-déploiement
- Mettre en place la surveillance des logs et des alertes (erreurs 4xx/5xx, temps de réponse).
- Planifier les sauvegardes régulières de la base.
- Documenter la procédure de rollback.
- Surveiller les mises à jour critiques (ex : migration Vite >= 7 pour corriger la vulnérabilité esbuild).

### Rollback (Render)

1. Depuis `Événements`, cliquez sur le déploiement sain précédent puis « Retour en arrière ».
2. Si le cache semble corrompu : `Clear build cache & Redeploy` sur le commit stable.
3. Pour un retour express : `Deploy latest` sur la version GitHub connue (tag/commit). Gardez Prisma en lecture seule pour éviter les migrations hasardeuses.
4. Après rollback, vérifiez :
    - `/health` et smoke tests distants (`npm run smoke` avec `SMOKE_BASE_URL`).
    - Migration DB : `Prisma migrate status` (Render shell) doit rester « Already in sync ».
5. Documentez l’incident dans `docs/incidents/` (à créer si nécessaire) : date, commit fautif, plan d’action.

### Monitoring & alertes minimales

- Journaux Render : onglet « Journaux » pour suivre erreurs 4xx/5xx, stack traces Prisma. Ajouter un filtre `level:error`.
- Métriques : surveiller `CPU`, `RAM` et `Requests/min` dans l’onglet Métrique. Anticiper la montée de charge.
- Alertes simples : configurer une automation (Zapier/IFTTT) qui interroge `https://backend-nowis-1.onrender.com/health` toutes les 5 minutes et ping Slack/Email en cas d’échec.
- Sauvegardes : activer snapshots automatiques Postgres chez Render ou exporter quotidiennement via `pg_dump` (jobs Render Cron ou GitHub Actions).
- Sécurité : vérifier chaque trimestre les clés sensibles (`JWT_SECRET`, `OPENAI_API_KEY`). Forcer la rotation si fuite suspectée.

- Checklist après chaque déploiement :
   1. `npm run smoke` (Render).
   2. S’assurer que les jobs planifiés (wealth snapshot) tournent (`logs` → `WealthSnapshotJob`).
   3. Vérifier les nouvelles routes dans les journaux (absence d’erreurs 500).

### Smoke tests (local & Render)

- Local (démarrage et vérification end-to-end automatiques) :
   ```powershell
   npm run build
   node .\dist\tmp\manualSanityCheck.js
   ```
   Ce script démarre le serveur local, crée un utilisateur de test, appelle les endpoints clés (/health, /api/health, /api/advisors/health, /api/summary, /api/profile/summary, /api/advisors/questions), puis arrête le serveur.

- Remote (Render) — endpoints publics + privés si token disponible :
   ```powershell
   # publics
   $env:SMOKE_BASE_URL = "https://<votre-service>.onrender.com"; npm run smoke

   # avec endpoints protégés
   # Option A (sans Shell Render) : laissez le script obtenir un token via /api/auth/login
   $env:SMOKE_BASE_URL = "https://<votre-service>.onrender.com"
   $env:SMOKE_EMAIL = "render-smoke@nowis.local"
   $env:SMOKE_PASSWORD = "Aa12345678!Smoke"  # 12+ chars avec Maj/Min/Chiffre/Spécial
   npm run smoke

   # Option B (avec Shell Render) : générez un JWT manuel et fournissez-le
   $env:SMOKE_BASE_URL = "https://<votre-service>.onrender.com"; $env:SMOKE_TOKEN = "<jwt>"; npm run smoke
   ```
   Le script `npm run smoke` vérifie toujours /health, /api/health, /api/advisors/health. Il teste aussi /api/summary et /api/profile/summary uniquement si `SMOKE_TOKEN` est fourni.

- Obtenir un JWT temporaire pour Render (à exécuter via Shell Render ou SSH) :
   ```bash
   npm run build
   node ./dist/tmp/manualSanityCheck.js --token-only --print-token
   ```
   La commande crée/rafraîchit l’utilisateur `manual-sanity@nowis.local` puis affiche un token JWT valable 2h. Copiez ce token dans `SMOKE_TOKEN` pour tester les endpoints privés.

#### Journal des smoke tests Render

| Date | URL cible | Token | Résultat | Notes |
| --- | --- | --- | --- | --- |
| 27/10/2025 | https://backend-nowis-1.onrender.com | login auto (SMOKE_EMAIL) | OK 5/5 | /health, /api/health, /api/advisors/health, /api/summary, /api/profile/summary |

### Notes importantes Render
- Évitez SQLite en production: utilisez une base Postgres Render (Free) pour la persistance.
- Le stockage local (`uploads/`) est éphémère. Pour des pièces jointes persistantes, utiliser un stockage objet (S3/Cloudflare R2) et configurer les variables correspondantes.
- Puppeteer (PDF) peut nécessiter des flags Chromium selon l’environnement. Si nécessaire, ajoutez `PUPPETEER_EXECUTABLE_PATH` ou désactivez la génération PDF en environnement Free.
 - Port binding: assurez-vous que votre serveur écoute sur `0.0.0.0` et sur `process.env.PORT`. N’utilisez pas un port codé en dur (3000/4000) ni `localhost`.

### Paramètres clés et observabilité

- Import d’impôt (IA):
   - Flags utilisables en query string lors de l’ingestion `/api/ai/ingest`:
      - `autoCreate=true|false` (création automatique des revenus détectés)
      - `postToLedger=true|false` (publier ou non les écritures; défaut contrôlé par `POST_TO_LEDGER_DEFAULT`)
   - Variables d’environnement:
      - `POST_TO_LEDGER_DEFAULT`: défaut global si le paramètre n’est pas fourni en requête
      - `OPENAI_API_KEY`: obligatoire pour l’analyse des PDF

- Diagnostics UI (frontend):
   - Écran « Revenus personnels » > bouton « Pourquoi ce total ? » interroge `/why/personal-income` et affiche:
      - Revenus saisis, agrégats fiscaux (si calculés), et écritures postées (journal + lignes)
   - Panneau « Événements récents » interroge `/events/recent` (jusqu’à 20 par défaut)
   - Panneau « Graphe de dépendances » interroge `/graph/nodes` et permet un recalcul `/graph/recalc` depuis `Tax` ou `Compta`

- Bus d’événements persistant (optionnel):
   - Définissez `REDIS_URL` pour activer Redis; le backend utilise ioredis dynamiquement si disponible
   - Avantages: historique persistant, consommation différée, possibilité de DLQ/retries
   - Sans `REDIS_URL`: bus en mémoire (utile pour démo/tests, volatile)

### Cookbook: tester l’import fiscal et la propagation

1. Vérifier les variables d’env (Render): `OPENAI_API_KEY` renseignée, `POST_TO_LEDGER_DEFAULT=true` (ou cocher « Publier au grand livre » dans l’UI avant import).
2. Frontend: aller sur « Revenus personnels imposables ».
3. Importer un PDF (T4/T5/rapport). L’année détectée recadre automatiquement l’affichage.
4. Attendu:
    - Notification succès avec le nombre de revenus créés, ou alerte si analyse sans création auto.
    - Dans « Pourquoi ce total ? »: total des revenus + (si publication) écritures comptables postées.
    - Dans « Événements récents »: événements d’import/extraction/persistance/posting visibles.
    - Dans « Graphe de dépendances »: possibilité de déclencher un recalcul et voir l’ordre de propagation.
