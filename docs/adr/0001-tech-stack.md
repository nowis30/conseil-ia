# ADR 0001 – Choix techniques Phase 1

- **Date** : 2025-10-23
- **Statut** : Accepté
- **Contexte** : La plateforme Nowis IA doit livrer en Phase 1 un tableau de bord immobilier complet avec exports CSV/PDF. Les prochaines phases introduiront de nouvelles fonctionnalités fiscales et IA, d'où la nécessité d'une stack extensible.

## Décisions

1. **Backend Node.js 20 + Express 5**
   - Cadre HTTP minimaliste et éprouvé.
   - Facile à enrichir avec middlewares (auth, validation, sécurité) et compatible avec l'écosystème TypeScript.

2. **ORM Prisma (SQLite en dev, Postgres en production)**
   - Schemas déclaratifs, migrations versionnées, génération de types.
   - Basculer vers Postgres ne nécessite qu'un changement de provider et une migration planifiée.

3. **Validation Zod**
   - Schémas partagés entre backend et frontend.
   - Rapports d'erreurs standardisés pour les API.

4. **Authentification JWT + bcrypt**
   - Compatible avec version mono-utilisateur initiale, extensible à multi-sessions.
   - Conservation d'un secret hors repo via `JWT_SECRET`.

5. **Frontend React 18 + Vite**
   - Temps de build rapides et HMR efficace pour l'équipe.
   - Ecosystème compatible TypeScript et tests Vitest.

6. **TanStack Query + Zustand**
   - TanStack pour cache réseau et synchronisation avec l'API.
   - Zustand pour état UI local sans lourdeur d'un store global structuré.

7. **UI MUI + Recharts**
   - MUI fournit composants accessibles (Table, Cards) et thème personnalisable.
   - Recharts suffit pour graphiques cashflow/équité phase 1.

8. **Génération PDF via Puppeteer**
   - Rendu HTML/CSS fidèle, permet réutiliser composants React pour les rapports.
   - Alternatives (PDFKit) gardées en secours si contraintes de déploiement.

9. **Exports CSV avec fast-csv**
   - Streaming efficace, évite charges mémoire pour gros jeux de données.

10. **Qualité & CI**
    - Tests : Jest + Supertest (API), Vitest + Testing Library (frontend), Playwright (E2E exports).
    - Outils : ESLint + Prettier + Husky (pre-commit), GitHub Actions (lint/test/build).
    - Journalisation backend avec Pino.

## Conséquences

- Nécessité de gérer les dépendances Puppeteer (Chromium) dans l'image Docker de production.
- Les développeurs doivent exécuter `prisma generate` après installation des dépendances backend.
- Les exports PDF requièrent une attention particulière pour les polices et l'internationalisation (français canadien).
- L'utilisation de Prisma impose des migrations structurées et des revues lors des changements de schéma.
