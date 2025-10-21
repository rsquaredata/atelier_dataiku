# Module 2 - Détection de fraude : classification déséquilibrée et évaluation
## 1. Introduction et objectifs

Ce module prolonge le précédent en abordant la détection de fraude à partir de transactions bancaires.
Le jeu de données utilisé est `creditcard.csv`, importé puis renommé `fraud` dans Dataiku Cloud.
L'objectif est d'entraîner un modèle supervisé sur un jeu de données fortement déséquilibré, puis d'évaluer sa performance.

Objectifs pédagogiques :
- Explorer un jeu de données transactionnel
- Gérer le déséquilibre de classes en apprentissage supervisé
- Effectuer un feature engineering minimal validé (ex. `hour` à partir de `Time`)
- Entraîner un modèle de classification avec XGBoost
- Interpréter les métrriques adaptées au déséquilibre
- Créer un tableau de bord de suivi des prédictions

---

## 2. Rappels théoriques - classification déséquilibrée et métriques adaptées

### Problématique du déséquilibre de classes

En détection de fraude, la proportion de transactions frauduleuses est très faible (souvent inférieure à 1 %).
Ce déséquilibre biaise les modèles vers la classe majoritaire.
Il nécessite des métriques et approches adaptées.

| Méthode | Description | Objectif |
|----------|--------------|-----------|
| Undersampling | Réduction des observations de la classe majoritaire | Rééquilibrer le dataset |
| Oversampling | Réplication de la classe minoritaire | Augmenter les fraudes visibles |
| SMOTE | Génération artificielle de cas minoritaires | Mieux répartir les frontières de décision |

### Métriques spécifiques

Métrique locales (dépendent du seuil) :
- $\text{Precision} = \frac{TP}{TP+FP}$,
- $\mathrm{Recall} = \frac{TP}{TP+FN}$,
- $\mathrm{F1} = 2\cdot\frac{Precision\cdot Recall}{Precision+Recall}$.

Métriques globales :
$$
AUC-PR = \int_0^1 Precision(Recall)\,d(Recall)
$$

$$
AUC-ROC = \int_0^1 TPR(FPR)\,d(FPR)
$$

### Glossaire intégré

| Terme | Définition |
|--------|-------------|
| Imbalanced dataset | Jeu de données où une classe domine |
| SMOTE | Synthetic Minority Over-sampling Technique |
| AUC-PR | Aire sous la courbe Precision-Recall |
| AUC-ROC | Aire sous la courbe Receiver Operating Characteristic |
| Cost-sensitive learning | Pondération des erreurs selon leur coût |
| XGBoost | Gradient boosting optimisé pour la vitesse et la précision |

---

## 3. TP - Détection de fraude

> Pré-requis : disposer du fichier `creditcard.csv` et rester dans le même projet Dataiku Cloud que pour le scoring.

### A. Import du dataset et identification de la variable cible

1. Dans le **Flow**, ajouter un nouveau dataset : **+ Dataset -> Files -> Upload your files**.  
   - Importer le fichier `creditcard.csv`.  
   - Renommer-le immédiatement en **fraud** pour plus de clarté (**More actions (...) -> Rename -> fraud**).  
2. Ouvrir le dataset **fraud -> Explore**.  
   - Observer la structure : les colonnes sont nommées `V1`, `V2`, etc., plus `Time`, `Amount`, et `Class`.  
   - Identifier la variable cible (indice : `1` = fraude, `0` = transaction normale).  
3. Créer une recette de préparation : **Flow -> + Recipe -> Prepare -> Output : fraud_prepared -> Create**.  
4. Dans la préparation :  
   - **+ Add step -> Rename columns -> Class -> is_fraud**.  
   - Vérifier dans **Schema** que `is_fraud` est bien de type numérique (Integer).  
   - Exécuter la recette : **Run**.  

### Questions

a. Quelle est la variable cible et quelle transformation a été appliquée ?  
b. Combien le dataset contient-il d'observations et de variables ?  
c. Quelle est la proportion de transactions frauduleuses ?  

<details>
  <summary><strong>💡</strong></summary>

a. La variable cible est `Class`, renommée en `is_fraud`.  
b. Environ 285 000 observations et 31 variables.  
c. Environ 0.17 % de fraudes, soit un fort déséquilibre.  

</details>

---

### B. Exploration initiale et création de la variable `hour`

1. Ouvrir le dataset **fraud_prepared -> Explore**.  
   - Visualiser les premières lignes et repérer la colonne `Time`.  
   - Cette variable indique le nombre de secondes écoulées depuis le début de la collecte.  
2. Créer une nouvelle recette de préparation : **Flow -> + Recipe -> Prepare -> Output : fraud_hour -> Create**.  
3. Dans la préparation :  
   - **+ Add step -> Formula -> New column : hour -> Expression : floor(Time / 3600)**.  
   - Cette transformation convertit `Time` (secondes) en heures pour faire apparaître d'éventuels motifs temporels.  
   - Exécuter la recette : **Run**.  
4. Ouvrir le dataset **fraud_hour -> Explore -> Charts**.  
   - Créer un graphique en barres : X = `hour`, Y = `count`.  
   - Ajouter une **facet** sur `is_fraud` pour comparer la répartition des fraudes selon l'heure.  

### Questions

a. Quelle est la plage de valeurs de `hour` ?  
b. Certaines heures présentent-elles davantage de fraudes ?  
c. Pourquoi cette transformation peut-elle aider le modèle ?  

<details>
  <summary><strong>💡</strong></summary>

a. `hour` varie de 0 à environ 47 (48 heures).  
b. Les heures nocturnes peuvent présenter un léger excès de fraudes.  
c. Elle capte un comportement temporel, utile pour distinguer des transactions atypiques.  

</details>

---

### C. Échantillonnage et rééquilibrage

1. Sélectionner le dataset **fraud_hour** -> **+ Recipe -> Sampling -> Output : fraud_sampled -> Create**.  
2. Paramétrer le sampling :  
   - **Sampling type : Stratified sampling** sur `is_fraud`.  
   - **Sample size : 5000 lignes**.  
3. Exécuter la recette : **Run**.  
4. Ouvrir le dataset **fraud_sampled -> Explore** pour vérifier la nouvelle proportion de fraudes (nettement plus élevée).  

### Questions

a. Pourquoi est-il nécessaire d'équilibrer le dataset avant apprentissage ?  
b. Quelles autres techniques permettent de traiter le déséquilibre ?  

<details>
  <summary><strong>💡</strong></summary>

a. Un dataset trop déséquilibré pousse le modèle à prédire la classe majoritaire.  
b. Alternatives : oversampling, SMOTE, pondération des classes (`class_weight`).  

</details>

---

### D. Entraînement du modèle (XGBoost)

1. Sélectionner **fraud_sampled -> Lab -> + New -> Visual analysis -> Predict**.  
2. Définir les paramètres :  
   - **Target variable : is_fraud**.  
   - **Prediction type : Binary classification**.  
3. Inclure toutes les variables sauf `Time` (inutile car redondante avec `hour`).  
4. Dans **Design -> Algorithms**, ne garder que **XGBoost**.  
5. Dans **Settings -> Train/Test split**, choisir 80 % / 20 %.  
6. Lancer l'entraînement : **Train -> Start**.  
7. Une fois terminé, consulter **Performance -> Metrics** : Precision, Recall, F1-score, AUC-PR, AUC-ROC.  
8. Déployer le modèle : **Deploy -> Create Scoring recipe -> Output : fraud_prediction -> Create -> Run**.  

### Questions

a. Quelle métrique privilégier pour évaluer ce modèle ?  
b. Quelle valeur de Recall le modèle obtient-il ?  
c. Comment interpréter un Recall élevé avec une Precision faible ?  

<details>
  <summary><strong>💡</strong></summary>

a. Le Recall et l'AUC-PR sont les plus pertinents pour les jeux très déséquilibrés.  
b. Typiquement autour de 0.85 selon les runs.  
c. Un Recall élevé indique peu de fraudes manquées, mais beaucoup de faux positifs.  

</details>

---

### E. Analyse des résultats et interprétation

1. Ouvrir le modèle -> **Interpretation -> Feature importance**.  
   - Repérer les variables dominantes (`V14`, `V10`, `V12`, et éventuellement `hour`).  
2. Ouvrir **Performance -> ROC curve** et **Precision-Recall curve**.  
3. Expérimenter le déplacement du **seuil de classification** pour ajuster le compromis entre Precision et Recall.  

### Questions

a. Quelle variable influence le plus la détection ?  
b. Pourquoi l'AUC-PR complète-t-elle l'AUC-ROC ?  
c. Comment l'ajustement du seuil influe-t-il sur les faux positifs ?  

<details>
  <summary><strong>💡</strong></summary>

a. `V14` est souvent la plus discriminante.  
b. L'AUC-PR mesure mieux la qualité du modèle sur la classe minoritaire.  
c. Un seuil plus bas augmente le Recall mais aussi les faux positifs.  

> <small>**Contexte du dataset : `creditcard.csv`**  
> Le jeu de données provient d'une étude menée par Andrea Dal Pozzolo (Université de Liège, 2015).  
> Il contient environ 285 000 transactions bancaires européennes, dont 0,17 % sont frauduleuses.  
> Les variables `V1` à `V28` sont issues d'une analyse en composantes principales (ACP) des variables réelles, masquées pour confidentialité.  
> Ces composantes n'ont pas de signification métier directe, mais certaines (notamment `V14`, `V10`, `V12`) sont très corrélées à la fraude.  
> `Time` représente le nombre de secondes écoulées depuis le début de la collecte, `Amount` le montant, et `Class` la variable cible.  
> </small>

</details>

---

### F. Restitution et tableau de bord

1. Créer un tableau de bord : **+ New -> Dashboard -> fraud_dashboard**.  
2. Ajouter les éléments suivants :  
   - Feature importance.  
   - Precision-Recall curve et ROC curve.  
   - Confusion matrix.  
   - Table triée des transactions à haut risque (tri sur `probability_1` décroissant).  
   - Bloc **Text** avec synthèse des résultats.  
3. Publier le dashboard.  

### Questions

a. Quelle est la valeur du F1-score du modèle ?  
b. Quelle interprétation donner à un AUC-PR proche de 1 ?  

<details>
  <summary><strong>💡</strong></summary>

a. F1-score souvent faible, reflet du compromis entre rappel et précision.  
b. excellent pouvoir de détection malgré le déséquilibre.  

</details>

---

### Vérification du Flow

Flow final attendu :  

```
creditcard.csv -> fraud -> fraud_prepared -> fraud_hour -> fraud_sampled -> fraud_prediction -> fraud_dashboard
```

Un modèle **fraud_xgboost_model** doit apparaître dans le **Lab**.  

---

## 4. Perspective métier

La détection de fraude en banque repose sur un compromis entre Recall et Precision.
- Un Recall élevé garantit qu'on ne laisse passer presque aucune fraude.
- Une Precision plus faible implique davantage de faux positifs, donc une vérification humaine accrue.

Les institutions fixent le seuil selon leur tolérance au risque et leurs capacités opérationnelles.

---
