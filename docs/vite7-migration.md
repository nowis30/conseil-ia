# Plan de migration vers Vite 7

## Objectif
Mettre à jour Vite (et vitest) pour intégrer les correctifs de sécurité esbuild (GHSA-67mh-4wv8-2f99) sans regression.

## Étapes
1. **Création de branche**
   ```powershell
   git checkout -b chore/upgrade-vite7
   ```
2. **Mise à jour des dépendances**
   ```powershell
   cd frontend
   npm install vite@^7 vitest@^2 @vitejs/plugin-react@^5
   npm install -D @testing-library/react@^16 @testing-library/user-event@^14
   ```
   - Vérifier les peerDependencies et adapter si besoin.

3. **Mises à jour de configuration**
   - Vérifier `vite.config.ts` pour les nouvelles options (ex. `server.headers`).
   - Adapter `vitest.config.ts` (options renommées, couverture).
   - Mettre à jour les scripts npm si nécessaire (`preview` change de flags).

4. **Corrections de code éventuelles**
   - Ajuster les imports si Vite 7 renforce les règles d’extensions.
   - Tester les plugins MUI/Recharts pour les warnings éventuels.

5. **Vérification qualité**
   ```powershell
   npm run lint
   npm run test
   npm run build
   ```
   - Surveiller les warnings de build (tree-shaking, asset size).

6. **Recette ciblée**
   - Vérifier les écrans critiques (dashboard, imports/exports, modal dialogues).
   - Tester la navigation React Router (future flags déjà activés).

7. **Audit sécurité**
   ```powershell
   npm audit
   ```
   - Confirmer que l’advisory esbuild est résolu.

8. **Ouverture de PR**
   - Rédiger le changelog (BREAKING CHANGES si API modifiée).
   - Demander une revue croisée backend/frontend.

## Risques & mitigations
- **Risque** : rupture plugin ou configuration. *Mitigation* : tester progressivement, conserver la branche.
- **Risque** : taille du bundle augmentée. *Mitigation* : comparer `dist` avant/après (`npm run build -- --report`).
- **Risque** : adoption anticipée de features React Router 7. *Mitigation* : conserver les future flags et surveiller la release officielle.
