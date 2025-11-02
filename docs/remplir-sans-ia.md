# Remplir sa déclaration sans IA

Ce guide rapide explique comment gérer votre déclaration de revenus personnels sans passer par l’IA. Tout se fait depuis l’écran "Revenus personnels".

## 1) Créer/Ouvrir le retour de l’année

- Choisissez l’actionnaire ("profil personnel") ou créez-en un via le bouton "Nouveau profil".
- Sélectionnez l’année fiscale.
- Cliquez "Créer/Ouvrir le retour" pour initialiser la déclaration de cette année.

## 2) Générer les lignes INCOME depuis vos revenus

- Saisissez vos revenus (T4, T5, pensions, entreprise…) via "Ajouter un revenu".
- Cliquez "Générer lignes depuis revenus":
  - Les lignes de la section INCOME sont créées automatiquement à partir de vos revenus saisis.
  - Un calcul d’impôt est lancé pour mettre à jour les agrégats (revenu imposable, impôt fédéral/provincial, solde).

## 3) Compléter les sections DEDUCTION et CREDIT

- Dans "Lignes du rapport d’impôt", les lignes sont regroupées par section (INCOME, DEDUCTION, CREDIT, SUMMARY, …).
- Dépliez une section, ajoutez/modifiez/supprimez les lignes.
- Les totaux de section sont affichés.

## 4) Calculer l’impôt

- Dans "Calcul du rapport d’impôt", ajustez les Déductions, Autres crédits et la Province (QC ou autre).
- Cliquez "Calculer l’impôt" pour voir les résultats. Le bouton est désactivé si l’actionnaire ou le retour ne sont pas prêts.

## 5) Dupliquer depuis N-1 (optionnel)

- Cliquez "Dupliquer lignes N-1" pour reprendre la structure de l’année précédente.
- Options disponibles:
  - Inclure la section SUMMARY
  - Écraser les lignes existantes dans les sections copiées (confirmation affichée si contenu déjà présent)
- Un calcul d’impôt est lancé après duplication pour mettre à jour les agrégats.

## Astuces

- Mode simplifié: masque les sections avancées (documents importés, feuillets IA, diagnostics).
- "Rafraîchir les données": force un recalcul et l’actualisation des caches.
- Les actions clés (génération, duplication) émettent des événements UI légers pour l’observabilité (si un collecteur est configuré côté serveur).
