# Projet-Datamining

Réalisé par : Doublani Abdessalame , Bouamama Khalil , Taieb Maya , Belhadj Samia , Khenniche Idir , Boulkenater Nour El Houda

## Étape 1 – Nettoyage et vérification

Notebook : `notebooks/01_nettoyage.ipynb`

### Compréhension

Avant toute analyse, il faut une base fiable. Le fichier brut contient 940 réservations pour 32 hôtels. Les taux (annulation, par canal, par segment) et les modèles des questions suivantes dépendent de cette base : des doublons ou des valeurs aberrantes fausseraient tous les résultats.

### Récapitulatif des lignes

| Étape | Lignes |
|---|---|
| Fichier brut | 940 |
| Après suppression des doublons | 800 |
| Base de modélisation (sans anomalies) | 794 |

### Méthode et justification

**1. Doublons (940 → 800).**
Nous avons compté 140 doublons exacts. Ce nombre est égal au nombre d'identifiants `reservation_id` répétés, donc aucun identifiant n'apparaît avec des valeurs différentes : ce sont bien des copies et non des réservations distinctes. Nous les avons supprimés pour ne pas gonfler artificiellement les taux.

**2. Cohérence avec la table des hôtels.**
Toutes les réservations pointent vers un hôtel présent dans `Valmont_hotels.csv` (32 hôtels, tous utilisés). La jointure sera donc possible sans perte pour les questions Q3, Q4, Q6 et Q9.

**3. Valeurs manquantes.**
Après dédoublonnage, 20 valeurs manquent pour `nb_personnes` et 20 pour `petit_dejeuner`.
- `nb_personnes` : remplacé par la **médiane du segment déclaré** (affaires 1, loisir 2, groupe 11). La taille d'un séjour dépend fortement du segment ; une médiane globale (1) sous-estimerait les groupes. La médiane est aussi moins sensible aux valeurs extrêmes que la moyenne. Un indicateur `nb_personnes_impute` conserve la trace des lignes remplies.
- `petit_dejeuner` : remplacé par la modalité **`inconnu`**, car aucune règle fiable ne permet de deviner oui ou non sans inventer une information.

**4. Valeurs aberrantes (800 → 794).**
Nous avons relevé 3 séjours de 365 nuits et 3 réservations à 0 € la nuit, soit 6 lignes sans chevauchement. Elles ne sont **pas supprimées en silence** : elles sont comptées, enregistrées dans `data/clean/anomalies_a_part.csv` (réutilisé pour la Q9) et seulement exclues de la base de modélisation.

**5. Colonne `nuitees_cumulees_client`.**
Un cumul client devrait être supérieur ou égal au nombre de nuits de la réservation en cours. Ce n'est pas le cas pour 78 lignes. Ces lignes annulent un peu plus (33 % contre 25 % pour les autres), mais l'écart repose sur peu de lignes et n'est pas testé à ce stade. Nous avons conservé la colonne sans l'utiliser telle quelle.

### Vérification

- Le taux global d'annulation sur les 800 lignes est de 26,1 %, cohérent avec le premier constat du sujet (26 %).
- Les nombres 940, 800 et 794 sont recalculés par le notebook et affichés dans le tableau récapitulatif.
- L'égalité entre doublons exacts et identifiants répétés est contrôlée par une assertion dans le code.

### Conclusion et recommandations

- La base de travail est de 800 lignes propres, dont 794 pour la modélisation ; 6 anomalies sont conservées à part.
- Pour la Q5, le modèle de prédiction de l'annulation sera testé **avec et sans** `nuitees_cumulees_client`, car cette valeur pourrait être calculée après coup et contenir l'information sur l'annulation (fuite d'information).
- Pour vérifier l'effet de l'imputation, nous pourrons refaire un calcul sans les 20 lignes dont `nb_personnes` a été imputé.
- Pour la Q9, les 6 anomalies de ce fichier seront comptées avec celles détectées par l'IQR et l'isolation forest.