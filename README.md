# Conseil IA — documentation NOWIS

Ce dépôt conserve les décisions d’architecture, les spécifications métier et les plans de recette du projet Conseil IA. **Il ne contient pas une application à démarrer ou à déployer.** Les anciennes instructions de monorepo sont conservées dans [les archives 2025](docs/archive/README-monorepo-2025.md).

## Où se trouve le code ?

| Composant | Dépôt source | Contrôles à exécuter dans ce dépôt |
|---|---|---|
| API Express / Prisma et services conseillers | [backend-nowis](https://github.com/nowis30/backend-nowis) | Génération Prisma, lint, tests, build |
| Interface React / Vite | [frontend-nowis](https://github.com/nowis30/frontend-nowis) | Lint, Vitest, build avec API d’environnement explicite |
| Site public et portail de Création NOWIS | [site-nowis-web](https://github.com/nowis30/site-nowis-web) | Procédures et contrôles de ce projet distinct |

Cloner les dépôts de code concernés, puis suivre leurs README et exemples de configuration. Ne pas exécuter `cd backend` ou `cd frontend` ici : ces dossiers sont absents. Aucune clé, base locale ou donnée de client ne doit être ajoutée dans ce dépôt documentaire.

## Documentation disponible

- [Choix techniques](docs/adr/0001-tech-stack.md) et [graphe d’événements](docs/adr/0002-mdc-and-event-graph.md).
- [Plan de recette](docs/recette-plan.md).
- [Flux manuel sans IA](docs/remplir-sans-ia.md).
- [Module immobilier](docs/properties.md).
- [Exports financiers](docs/financial-exports.md).
- [Spécifications des conseillers IA](docs/phase4-ia.md).
- [Checklist historique de déploiement](docs/deployment-checklist.md).

Ces documents décrivent l’intention et l’historique du projet. Ils ne prouvent ni la conformité d’un calcul fiscal actuel, ni le fonctionnement de la version déployée. Vérifier le code, ses tests et les paramètres applicables avant usage.

## Maintenance et livraison

La CI applicative appartient aux dépôts qui contiennent le code. L’ancien workflow qui cherchait `backend/package-lock.json` et le Blueprint Render pointant vers `backend/` ont été déplacés sous `docs/archive/` pour éviter un déploiement à partir d’un dossier inexistant. Ces archives ne doivent pas être utilisées comme configuration active.

Pour une évolution : modifier d’abord la spécification pertinente ici, lier les PR des composants concernés, puis consigner le SHA et les résultats de tests effectivement vérifiés. Une affirmation historique « tests au vert » ne décrit pas nécessairement le main actuel.

Dernière clarification de structure : 26 octobre 2026.
