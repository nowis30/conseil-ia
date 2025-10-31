# Exports fiscaux et imports CSV

## Revenus - Import CSV

- **Endpoint** : `POST /api/revenues/import`
- **Content-Type** : `text/csv`
- **Colonnes attendues** (ordre libre) :
  - `propertyId` (numerique) ou `propertyName` (texte)
  - `label`
  - `amount`
  - `frequency` (`PONCTUEL`, `HEBDOMADAIRE`, `MENSUEL`, `TRIMESTRIEL`, `ANNUEL`)
  - `startDate` (ISO `YYYY-MM-DD`)
  - `endDate` (optionnel)
- Les lignes invalides sont ignorees et listees dans `errors` (ligne + message). Les enregistrements valides sont retournes dans `items`.

### Exemple de payload

```
propertyId,propertyName,label,amount,frequency,startDate,endDate
42,,Loyer 201,1200,MENSUEL,2024-01-01,
,Bloc A,Stationnement,75,MENSUEL,2024-02-01,2024-12-31
```

## Depenses - Export fiscal

- **Endpoint** : `GET /api/expenses/export/fiscal`
- **Query params** :
  - `year` (optionnel, defaut = annee courante)
  - `format=csv` pour declencher le telechargement CSV (sinon JSON)
- Le rapport regroupe les depenses par immeuble et categorie. Chaque ligne inclut la frequence, le nombre d'occurrences comptabilisees dans l'annee et les montants totaux.

### Colonnes CSV

- `Immeuble`
- `Categorie`
- `Libelle`
- `Frequence`
- `Occurrences`
- `Montant unitaire`
- `Total annuel`
- `Debut`
- `Fin`

Des lignes `TOTAL CATEGORIE`, `TOTAL IMMEUBLE` et une ligne `TOTAL` cloturent le fichier pour faciliter l'import dans un tableur.

## Frontend

- **Revenus** : bouton *Importer CSV* ouvre un dialogue d'upload avec recapitulatif des lignes importees/ignorees.
- **Depenses** : champ d'annee + bouton *Export fiscal* declenchent le telechargement CSV, avec messages de succes ou d'erreur auto-masques.
- **Bilan consolidé** : dashbord et écran *Companies* proposent l'export CSV/PDF du bilan via `/api/summary/export/*`.

### Export résumé CSV

Le fichier regroupe désormais plusieurs sections séparées par des lignes vides :

1. **Portefeuille immobilier** : en-tête `Immeuble` ... `Équité` identique au tableau principal (unités, loyer potentiel, dette, LTV, revenus, dépenses, dette, intérêts, capital, cashflow net, CCA, équité).
2. **Synthèse corporate** : tableau à deux colonnes `Indicateur`/`Valeur` (sociétés actives, actionnaires, classes d'actions, transactions, montants FMV et contrepartie, actifs/équité/bénéfice consolidés, dernier état, dernière résolution).
3. **États financiers (synthèse)** : liste des états par société avec colonnes `Type`, périodes ISO, drapeau d'audit et agrégats (actifs, passifs, capitaux propres, revenus, dépenses, bénéfice net, métadonnées).
4. **États financiers (détail des lignes)** : chaque ligne d'état (catégorie, libellé, montant, métadonnées).
5. **Résolutions corporatives** : date, type, titre, contenu condensé et métadonnées.

### Export résumé PDF

- Génére un rendu HTML via Puppeteer avec les mêmes sections et styles harmonisés.
- Les valeurs numériques sont formatées en CAD (`fr-CA`), les pourcentages sont arrondis à deux décimales, et les textes multiligne sont convertis en paragraphes.
