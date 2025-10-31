# Wireframes texte – Phase 1

## Vue Connexion
```
+--------------------------------------+
|      Connexion Nowis IA              |
|  [Courriel_______________________]   |
|  [Mot de passe___________________]   |
|  [ Se connecter ]                    |
|  Message erreur inline si besoin     |
+--------------------------------------+
```

## Tableau de bord
```
+--------------------------------------------------------------------------------+
| AppBar: Nowis IA – Gestion immobilière | Tableau de bord | Immeubles | Factures |
+--------------------------------------------------------------------------------+
| Cartes KPI (3 par rangée)                                                   |
| [Revenus  | $123 456 | helper] [Dépenses | $45 000] [Cashflow | $78 456]     |
| [Capital  | $12 345] [Intérêts | $9 876] [Équité | $300 000]                 |
+--------------------------------------------------------------------------------+
| Carte graphique "Performance par immeuble" (barres Revenus/Dépenses/Cashflow) |
|-------------------------------------------------------------------------------|
```

## Immeubles
```
+--------------------------------------------------------------------------------+
| Header: Titre + bouton "Ajouter un immeuble"                                  |
+--------------------------------------------------------------------------------+
| Tableau MUI                                                                  |
| | Nom                | Adresse           | Valeur actuelle | Actions [🗑] |   |
| |--------------------|-------------------|-----------------|-------------|   |
| | Immeuble A         | 123 Rue           | 1 400 000 $     | [Suppr]     |   |
| ...                                                                          |
+--------------------------------------------------------------------------------+
| Dialog modal "Nouvel immeuble"                                               |
| Champs: Nom, Adresse, Date acquisition, Valeur, Notes                        |
| Boutons: Annuler / Enregistrer                                               |
```

## Factures
```
+--------------------------------------------------------------------------------+
| Header: Titre + bouton "Ajouter une facture"                                  |
+--------------------------------------------------------------------------------+
| Tableau factures                                                              |
| | Date | Immeuble | Fournisseur | Catégorie | Montant $ |                     |
| |------|----------|-------------|-----------|-----------|                     |
| | 12/09| Immeuble A | Hydro     | Énergie   | 512.00 $  |                     |
| ...                                                                          |
+--------------------------------------------------------------------------------+
| Dialog modal "Nouvelle facture"                                              |
| Sélecteur immeuble, date, fournisseur, catégorie, montant, description       |
| Boutons: Annuler / Enregistrer                                               |
```

## Exports
```
Boutons dans Tableau de bord (section entête ou actions secondaires):
[Exporter CSV] [Exporter PDF]
```

> Ces wireframes servent de base rapide pour l'intégration MUI. Les composants finaux suivront ce layout en adaptant le thème et la hiérarchie typographique.
