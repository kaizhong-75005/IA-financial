# Calculateur de contribution produit

Application web pour calculer la contribution de chaque produit vendu, à partir :

- du **détail de vente** de l'année (une ligne par vente) ;
- du **PNL** (une ligne par poste de charge/revenu, avec 12 colonnes mensuelles).

## Utilisation sur un PC d'entreprise (Windows)

L'application est un **fichier HTML unique** (`index.html`), sans installation :

1. Copie `index.html` sur le poste (clé USB, mail, OneDrive, peu importe).
2. Double-clique dessus pour l'ouvrir dans le navigateur (Chrome/Edge/Firefox).
3. Utilise-la normalement.

Tout le traitement (lecture des Excel, calculs, export) se fait **localement dans le
navigateur**, en JavaScript. Aucune donnée n'est envoyée sur un serveur, aucune
connexion internet n'est nécessaire une fois le fichier ouvert — ce qui convient à
des données financières sensibles et à un poste dont l'accès réseau ou les droits
d'installation sont restreints.

La lecture/écriture des fichiers Excel utilise la librairie [SheetJS](https://sheetjs.com/)
(licence Apache-2.0), embarquée directement dans `index.html`.

## Étapes de l'application

1. **Détail de vente** — importer le fichier Excel/CSV des ventes. Le mapping des
   colonnes est pré-rempli pour l'ordre : Catégorie, Référence produit, Description,
   Date de facturation, Quantité, Prix unitaire, Chiffre d'affaires, COGS — ajustable
   si l'ordre diffère.
2. **PNL** — importer le fichier Excel du PNL. Choisir la colonne du libellé de poste
   et les colonnes de montants mensuels à additionner (12 colonnes attendues).
3. **Classification des postes** — pour chaque poste du PNL, choisir :
   - **Revenu** : ignoré du calcul de coûts (sert juste à la réconciliation) ;
   - **Coût direct** : déjà présent dans le détail de vente (le COGS), non réparti à
     nouveau — utilisé seulement pour vérifier la cohérence avec le PNL ;
   - **Variable à répartir** : réparti entre les produits au prorata du CA ou de la
     quantité vendue (ex : frais commerciaux, marketing) ;
   - **Fixe (exclu)** : coût de structure non attribué aux produits (loyer, salaires,
     frais généraux...).

   Une classification par défaut est proposée automatiquement (par reconnaissance de
   mots-clés dans le libellé du poste) et reste modifiable à tout moment.
4. **Résultats** — tableau de contribution par produit (CA, COGS, marge directe,
   coûts variables alloués, contribution en valeur et en %), camembert de répartition
   du CA par produit (5 plus gros produits + "Autres"), graphique des principales
   contributions, réconciliation avec le PNL, export Excel/CSV.

## Méthode de calcul

Pour chaque produit :

```
Marge directe      = Chiffre d'affaires - COGS
Coûts variables     = somme, pour chaque poste "Variable à répartir",
alloués               du (montant annuel du poste × part du produit dans le CA ou la quantité totale)
Contribution        = Marge directe - Coûts variables alloués
% Contribution      = Contribution / Chiffre d'affaires
```

Les coûts classés "Fixe" ne sont pas répartis par produit ; ils sont affichés en
agrégat et utilisés uniquement pour estimer un résultat global de contrôle :

```
Résultat estimé = Total des contributions - Total des coûts fixes
```

Ce résultat estimé est comparé au résultat implicite du PNL (total des revenus -
total des charges) pour détecter d'éventuels écarts de mapping ou de classification.

## Confidentialité et réutilisation d'une année sur l'autre

- Le mapping des colonnes et la classification des postes sont mémorisés dans le
  navigateur (`localStorage`) : si le PNL a la même structure d'une année sur
  l'autre, tu n'as pas à tout reconfigurer. Le bouton "Réinitialiser aux valeurs par
  défaut" permet de repartir de la classification automatique.
- Cette mémorisation reste locale au navigateur/poste utilisé ; elle n'est pas
  partagée ni synchronisée.

## Licence

Ce projet est distribué sous licence **GNU Affero General Public License v3.0
(AGPL-3.0)** — voir le fichier [`LICENSE`](LICENSE). C'est une licence copyleft
« contaminante » : toute personne qui redistribue ce code, le modifie, ou le fait
tourner comme service accessible sur un réseau, doit proposer le code source
(y compris ses modifications) sous la même licence aux utilisateurs de ce service.

`index.html` embarque [SheetJS](https://sheetjs.com/) (Copyright SheetJS LLC),
distribué sous licence Apache 2.0 — voir
[`THIRD_PARTY_LICENSES_SheetJS_Apache-2.0.txt`](THIRD_PARTY_LICENSES_SheetJS_Apache-2.0.txt).
