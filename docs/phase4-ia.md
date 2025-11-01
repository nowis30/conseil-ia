# Phase 4 – IA multi-expert (Version Gamma)

Cette phase introduit un comité d’experts virtuels pour accompagner les clients dans leurs décisions fiscales et corporatives. Tout est simulé localement (aucun appel à un modèle externe) pour garantir la reproductibilité.

## Modules d’expertise

| Expert          | Rôle principal                                                 | Points de sortie principaux |
| ----------------| -------------------------------------------------------------- | --------------------------- |
| Fiscaliste      | Optimise la déduction pour petites entreprises et la fiscalité | Taux effectif estimé, espace SBD restant |
| Comptable       | Analyse la trésorerie et la marge opérationnelle               | Couverture de fonds de roulement, rémunération suggérée |
| Planificateur   | Oriente l’utilisation de la liquidité excédentaire             | Ratio de réinvestissement, coussin de dividendes |
| Avocat          | Évalue la protection juridique et la conformité                | Indice de protection d’actifs, préparation successorale |
| Coordinateur    | Fusionne les conclusions et produit la synthèse finale         | Résumé global et liste des suivis |

Chaque module se trouve dans `backend/src/server/services/advisors/` et expose un analyseur déterministe. Le coordinateur combine les métriques et harmonise les recommandations.

## Stratégies IA

- Le backend expose deux modes : `heuristic` (défaut) et `gpt` (OpenAI). Configurez `ADVISOR_ENGINE` côté serveur.
- Pour `gpt`, fournissez `OPENAI_API_KEY` (obligatoire), et éventuellement `OPENAI_MODEL` (par défaut `gpt-4.1`) et `OPENAI_BASE_URL`.
- Le frontend peut relayer cette préférence via `VITE_ADVISOR_ENGINE`; à défaut il suit la configuration backend.
- La réponse API contient un bloc `engine` précisant le mode actif et un message d’information.

## API

| Endpoint                   | Méthode | Description |
| -------------------------- | ------- | ----------- |
| `/api/advisors/questions`  | `GET`   | Retourne la liste des questions à poser (ordre imposé) |
| `/api/advisors/evaluate`   | `POST`  | Reçoit le tableau des réponses et renvoie la prochaine question (ou, si terminé, l’ensemble des recommandations) |

Payload `POST /api/advisors/evaluate` :

```json
{
  "answers": [
    { "questionId": "taxableIncome", "value": "350000" },
    { "questionId": "profitMargin", "value": "28" }
  ]
}
```

Réponse typique une fois le questionnaire complété :

```json
{
  "completed": true,
  "nextQuestion": null,
  "coordinatorSummary": "Vous dépassez la limite SBD…",
  "recommendations": [
    { "expertId": "fiscaliste", "title": "Optimisation du revenu imposable", "summary": "…", "rationale": ["…"] }
  ],
  "metrics": [
    { "id": "effectiveTaxRate", "label": "Taux d’imposition", "value": "26,7 %", "explanation": "…", "expertIds": ["fiscaliste"] }
  ],
  "followUps": ["Valider la création d’une holding", "Préparer un plan de retraite corporatif"],
  "engine": {
    "mode": "gpt",
    "isSimulated": false,
    "note": "Propulsé par OpenAI gpt-4.1"
  }
}
```

## Frontend

- Nouvel écran `src/screens/Advisors.tsx` accessible via le menu « Conseillers IA ».
- Conversation guidée : chaque question apparaît dans le panneau gauche, l’utilisateur répond via le formulaire latéral.
- À la fin du questionnaire, l’écran affiche :
  - Synthèse globale
  - Cartes de recommandations par expert
  - Indicateurs cliquables donnant accès à l’explication détaillée (dialogue modal)
  - Pistes de suivi à intégrer au plan d’action
- Un chip rappelle le moteur IA actuellement actif et un bandeau informe si le mode GPT est simulé.
- Possibilité de relancer la simulation pour tester d’autres scénarios.

### Captation d’identité lors des imports (profil personnel)

- Lors d’un import de rapport d’impôt (onglet Revenus personnels), si le document contient des métadonnées d’identité lisibles (nom, adresse, téléphone, date de naissance), le profil actionnaire par défaut est mis à jour automatiquement.
- Le frontend invalide désormais le cache `personal-profile` après import afin que la section « Informations personnelles » se rafraîchisse sans rechargement.
- Ces informations de base servent de contexte aux modules IA (fiscaliste/planificateur/avocat) pour personnaliser les recommandations (constitution de compagnie, levier, gel successoral, etc.).

## Tests et validation

- Nouveau test d’intégration `src/__tests__/advisorsRoute.test.ts` (backend) qui vérifie :
  - L’accessibilité du questionnaire.
  - L’exécution séquentielle des questions.
  - La génération complète des recommandations.
- Les builds backend/frontend doivent réussir (`npm run build`).
- Les tests Jest nécessitent une base Postgres accessible via `DATABASE_URL` (voir README pour la configuration locale).

## Déploiement

1. Déployer le backend après le `npm run build`.
2. Mettre à jour les variables Render (`ADVISOR_ENGINE` pour le backend, `VITE_ADVISOR_ENGINE` côté frontend si nécessaire).
3. Déployer le frontend (nouvelle route `/advisors`).
4. Vérifier le flux complet en production : remplir le questionnaire et vérifier l’affichage des métriques et recommandations.

## Roadmap future

- Intégration d’un historique des conversations (persistance par utilisateur).
- Agrégation automatique dans le rapport annuel (PDF/CSV) avec annexes.
- Possibilité d’attacher des documents justificatifs à chaque recommandation.
