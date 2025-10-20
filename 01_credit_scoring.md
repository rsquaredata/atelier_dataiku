# Module 1 - Scoring de clients : exploration et modélisation supervisée

## 1. Introduction et objectifs

Ce module guide la création d'un modèle de scoring de crédit sous Dataiku Cloud.
Le jeu de données utilisé est `credit_scoring.csv`, importé puis renommé `risk` dans Dataiku.
Les étapes suivantes permettent de préparer les données, d'entraîner un modèle supervisé et d'interpréter ses résultats.

Objectifs pédagogiques :
- Importer, renommer et explorer un jeu de données dans Dataiku Cloud
- Nettoyer et transformer les variables pour la modélisation
- Entraîner et évaluer un modèle de classification binaire
- Interpréter les métriques principales et les variables explicatives
- Construire un tableau de bord de restitution

---

## 2. Rappels théoriques - statistiques descriptives et classification supervisée

### Variables
- Variable cible : binaire $y \in \{0,1\}$ (1 = bon payeur, 0 = mauvais payeur)
- Variables explicatives : quantitatives  (revenu, montant, durée) ou qualitatives (emploi, situation familiale)

### Régression logistique

$$
P(y_i=1 \mid X_i)=\frac{1}{1+e^{-(\beta_0+\beta_1x_{i1}+\ldots+\beta_p x_{ip})}}
$$

Chaque coefficient $\beta_j$ reflète l'influence de la variable $x_j$ sur la probabilité d'appartenir à la classe 1.
Régularisation L1 (Lasso) : $\lambda\sum|\beta_j|$ ; L2 (Ridge) : $\lambda\sum\beta_j^2$

### Évaluation du modèle

| Indicateur | Formule | Interprétation |
|-------------|----------|----------------|
| Accuracy | $$\frac{TP+TN}{TP+TN+FP+FN}$$ | Pourcentage de prédictions correctes |
| Recall (Sensibilité) | $$\frac{TP}{TP+FN}$$ | Capacité à détecter les cas positifs |
| Precision | $$\frac{TP}{TP+FP}$$ | Fiabilité des cas prédits positifs |
| F1-score | $$2\,\frac{\text{Precision}\times \text{Recall}}{\text{Precision}+\text{Recall}}$$ | Compromis Precision-Recall |
| AUC (ROC) | Aire sous la courbe ROC | Pouvoir discriminant global |

### Corrélation et colinéarité

$$
r_{XY}=\frac{\mathrm{Cov}(X,Y)}{s_X s_Y},\quad |r|>0{,}8 \Rightarrow \text{variables redondantes.}
$$

### Glossaire intégré

| Terme | Définition |
|--------|-------------|
| AutoML | Entraînement automatique de plusieurs algorithmes |
| Feature importance | Poids relatif d'une variable dans la performance |
| Overfitting | Sur-ajustement aux données d'entraînement |
| Threshold | Seuil de probabilité séparant les classes 0/1 |
| AUC | Aire sous la courbe ROC, proche de 1 = meilleur modèle |

---

## 3. TP - Scoring clients

> Pré-requis : disposer localement de `credit_scoring.csv`.

### A. Création du projet et import du dataset

1. Ouvrir <https://profile.dataiku.com/> → Start free trial → Dataiku Cloud
2. Créer un nouveau projet : **+ New Project → Blank project**
   - Project name : `Dataiku_Bank_[PrenomNom]`
   - Key : `DB_[initiales]`
3. Importer le fichier : **Flow → + Dataset → Files → Upload your files** → sélectionner `credit_scoring.csv` → Create
4. Renommer le dataset : **More actions (...) → Rename → risk**
5. Vérifier le schéma : **Schema →** vérifier les types (numérique vs catégoriel) → Save

### Questions

a. Quelle est la variable cible ?  
b. Combien le dataset contient-il d'observations et de variables ?

<details>
  <summary><strong>💡</strong></summary>

a. Variable cible : `credit_risk`  
b. Dimensions : 1 000 observations et 21 variables

</details>

---

### B. Exploration initiale

1. Ouvrir `risk` → **Explore**
2. **Charts** : histogrammes pour `amount`, `duration`, `age`
3. **Statistics → Missing values** : repérer les éventuelles valeurs manquantes
4. **Statistics → Correlation matrix** : repérer les variables corrélées
5. **Charts → Scatter** (`amount` vs `duration`) ; **Facet** par `credit_risk`

### Questions

a. Quelle variable présente la plus forte asymétrie ?  
b. Quelle variable semble la plus corrélée au risque ?  
c. Y a-t-il des valeurs manquantes ?  
d. Quelles relations observe-t-on entre `amount`, `age` et `duration` ?

<details>
  <summary><strong>💡</strong></summary>

a. `amount` (très asymétrique)  
b. `duration` (corrélation négative modérée avec `credit_risk`)  
c. Aucune valeur manquante  
d. `age` faiblement positif avec bon crédit ; `amount` et `duration` associés à plus de défauts quand ils augmentent

</details>

---

### C. Préparation des données et Code recipe

1. Depuis le **Flow**, sélectionner `risk` → **+ Recipe → Prepare → Output : risk_prepared → Create**
2. Nettoyer les noms de colonnes : **Columns → Rename columns** (`personal_status_sex`, `credit_history`, etc.)
3. Créer la variable `payment_intensity` : **+ Add step → Formula → New column : payment_intensity → Expression : amount / nullif(duration,0)**
4. Recoder `personal_status_sex` : **+ Add step → If... Then... Else → New column : Gender_bin**
   - Si `personal_status_sex` ∈ {female div/dep/mar, female single} alors 1, sinon 0
   - Forcer le type en **Integer**
5. Encodage de `job` : **+ Add step → Encoding → One-Hot encoding → Columns : job → Drop original : Yes**
6. Exécuter la recette : **Run →** vérifier la création de `risk_prepared`

#### Code recipe

1. **Flow → + Recipe → Code → Python**
2. Input : `risk_prepared` ; Output : `risk_prepared` (ou `risk_prepared_code` si remplacer ne passe pas)
3. Coller le code suivant :

```python
import pandas as pd, numpy as np
df = input_dataset.copy()
df["log_amount"] = np.log1p(df["amount"])
output_dataset = df
```

4. Exécuter la recette : **Run →** vérifier l'apparition de la colonne `log_amount`

### Questions

a. Combien de colonnes reste-t-il après préparation ?  
b. Quelle transformation améliore la lisibilité des montants ?  
c. Pourquoi utiliser `log1p` plutôt que `log` ?  
d. Quelle est la moyenne et l'écart-type de `log_amount` ?

<details>
  <summary><strong>💡</strong></summary>

a. 26  
b. La transformation logarithmique  
c. `log1p` calcule $\log(1+x)$ : il gère $x=0$ et stabilise les très petites valeurs, utile en finance (micro-paiements, taux d'intérêt, arrondis, ...)  
d. Moyenne : ≈ 7.15 ; écart-type : ≈ 0.88

</details>

---

### D. Construction du modèle

1. Sélectionner `risk_prepared` (ou `risk_prepared_code`) → **Lab → + New → Visual analysis → Predict**
2. Target : `credit_risk` ; Prediction type : Binary classification
3. Features : garder toutes les colonnes utiles, exclure identifiants
4. **Design → Algorithms** : cocher Logistic Regression, Random Forest, XGBoost
5. **Train → Start**
6. **Performance** : observer Accuracy, Recall, Precision, F1, AUC
7. **Deploy → Create Scoring recipe → Output : risk_predictions → Create → Run**

### Questions

a. À votre avis, quelle métrique faut-il privilégier en contexte bancaire ?  
b. Y a-t-il des variables problématiques ?  
b. Quel algorithme offre le meilleur compromis Precision/Recall ?  

<details>
  <summary><strong>💡</strong></summary>

a. Recall si le but est de minimiser les faux négatifs (ne pas accorder de prêt risqué)  
b. - **Éthique** : attention aux variables sensibles (`sex`, `age`)  
b. XGBoost ou Random Forest selon les runs  

</details>

---

### 3.5. Interprétation et restitution

1. Depuis la page du modèle : **Interpretation → Feature importance, Partial dependence**
2. Créer un dashboard : **+ New → Dashboard → risk_dashboard**
3. Ajouter : Feature importance, Confusion matrix, et un bloc **Text** avec une synthèse des résultats
4. Publier le dashboard

### Questions

a. Quelle variable contribue le plus à la prédiction ?  
b. Comment le seuil influence-t-il la matrice de confusion ?

<details>
  <summary><strong>💡</strong></summary>

a. `duration` ressort souvent en tête  
b. Augmenter le seuil = augmente les FP (perte potentielle de clients) / baisser le seuil = augmente les FN (risque de défaut) → seuil à déterminer selon la tolérance au risque.

**Exemple de synthèse pour le dashboard** : 
Ce modèle de scoring prédit le risque de défaut avec une AUC de 0,82 (c'est bien ?). Les variables importantes sont la durée et le montant du prêt, toutes deux corrélées négativement à la probabilité de remboursement. Ce modèle constitue un outil d'aide à la décision pour les conseillers.

### Vérification du Flow

Le Flow doit présenter les noeuds suivants :

```
risk → risk_prepared → (code) → risk_predictions → risk_dashboard
```

Si un dataset intermédiaire est créé (par exemple `risk_prepared_code`), il doit être l'entrée de la recette de scoring.

</details>

---

## 4. Perspective métier

Le scoring de crédit estime la probabilité de défaut d'un client.
En contexte bancaire :

- Utilité : pré-tri des dossiers pour étude plus poussée des cas risqués
- Métrique : Recall prioritaire car les faux négatifs sont coûteux
- Éthique : variables sensibles (sexe, âge) à encadrer/contrôler/justifier voire exclure en cas de mise en prod réelle
- Variables clés : `duration`, `amount`, `payment_intensity`

---

## 5. Ressources

- Dataiku Academy - *Classification Models* : <https://academy.dataiku.com/latest/course-detail/dataiku-ml-practitioner.html>
- Documentation - *Machine Learning in Dataiku* : <https://doc.dataiku.com/dss/latest/machine-learning/index.html>
- Hastie, Tibshirani, Friedman - *The Elements of Statistical Learning* (PDF libre) : <https://hastie.su.domains/ElemStatLearn/>
