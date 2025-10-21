# Module 1 - Scoring de clients : exploration et modélisation supervisée

## Introduction et objectifs

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

<details>
  <summary><strong></strong></summary>

  ## Rappels théoriques - statistiques descriptives et classification supervisée

### Variables
- Variable cible : binaire $y \in \{0,1\}$ (1 = bon payeur, 0 = mauvais payeur)
- Variables explicatives : quantitatives  (revenu, montant, durée) ou qualitatives (emploi, situation familiale)

### Régression logistique

$$P(y_i=1 \mid X_i)=\frac{1}{1+e^{-(\beta_0+\beta_1x_{i1}+\ldots+\beta_p x_{ip})}}$$

Chaque coefficient $\beta_j$ reflète l'influence de la variable explicative $x_j$ sur la probabilité d'appartenir à la classe 1 
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

$$r_{XY}=\frac{\mathrm{Cov}(X,Y)}{s_X s_Y},\quad |r|>0{,}8 \Rightarrow \text{variables redondantes.}$$

### Glossaire intégré

| Terme | Définition |
|--------|-------------|
| AutoML | Entraînement automatique de plusieurs algorithmes |
| Feature importance | Poids relatif d'une variable dans la performance |
| Overfitting | Sur-ajustement aux données d'entraînement |
| Threshold | Seuil de probabilité séparant les classes 0/1 |
| AUC | Aire sous la courbe ROC, proche de 1 = meilleur modèle |

---

</details>

## TP - Scoring clients

> Prérequis : avoir activé son compte Dataiku Cloud et téléchargé le fichier `credit_scoring.csv` sur sa machine.

### A. Création du projet et import du dataset

1. Ouvrir <https://profile.dataiku.com/> → Start free trial → Dataiku Cloud (si vous n'êtes pas redirigé sur Dataiku cloud, cliquez sur ce lien <https://launchpad-dku.app.dataiku.io/> , choisissez AWS Paris et nommez votre workspace
2. Un node s'affiche s'il est en **Running** cliquez sur Open Instance, sinon cliquez sur **...**  plus haut et puis **Turn On** 
3. Créer un nouveau projet : **+ New Project → Blank project**
   - Project name : `Dataiku_Bank_[PrenomNom]`
   - Key : `DB_[initiales]`
4. Importer le fichier : **Flow (vous y êtes déjà normalement) → + Add Item (en haut à droite) → Upload → Select Files** → sélectionner `credit_scoring.csv` → Create
5. Renommer le dataset : **Actions (à droite) → Rename → risk**
6. Vérifier le schéma : **Onglet "Settings"(dans le dataset) → Schema →** vérifier les types (numérique vs catégoriel), s'ils sont tous en string cliquez sur **CHECK NOW** → **INFER TYPES FROM DATA** → Save

### Questions

a. Quelle est la variable cible ?  
b. Combien le dataset contient-il d'observations et de variables (Explore) ?

<details>
  <summary><strong>💡</strong></summary>

a. Variable cible : `credit_risk`  
b. Dimensions : 1 000 observations et 21 variables

</details>

---

### B. Exploration initiale

1. Ouvrir `risk` → **Explore**
2. **Charts** : histogrammes pour `amount`, `duration`, `age` (utilisez Count of records pour Y). Vous pouvez créer plusieurs graphiques ou en regrouper. 
3. **Statistics → New Card → Automatically suggest analyses → Cocher les 4 features dans la partie droite** : Vous avez maintenant plusieurs analyses générées
4. **Missing values** : Utilisez les analyses statistiques générées pour repérer s'il y a d'éventuelles valeurs manquantes 
5. **Correlation matrix** : Utilisez la matrice de corrélation générée pour repérer les variables corrélées
6. **Charts → +Chart(en bas) → Scatter** (`amount` vs `duration`) ; **Facet** par `credit_risk`

### Questions 

a. A partir des analyses précédentes, quelle variable présente la plus forte asymétrie ?  
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

> **Astuce - Bonne pratique** : après avoir construit plusieurs recipes de préparation (Prepare, Code, Join…), penser à nommer clairement les sorties et à activer la documentation automatique dans le Flow (**More actions → Edit project documentation**). En fin de projet, cette documentation sert de traçabilité interne et peut être exportée comme rapport d’audit.

---

### C. Préparation des données et Code recipe

1. Depuis le **Flow**, sélectionner `risk` → **+ Recipe → Prepare → Output : risk_prepared → Create**
2. Renommer les noms de colonnes : **Columns → Rename columns** (`personal_status_sex`, `credit_history`, etc.)
3. Créer la variable `payment_intensity` : **+ Add a new step → Formula → Formula for : payment_intensity → Expression : amount /duration**
4. Recoder `personal_status_sex` : **+ Add a new step → Create If... Then... Else Statement** (aussi possible en utilisant Formula)
   - Si `personal_status_sex` ∈ {female div/dep/mar, female single} alors 1, sinon 0
   - Forcer le type en **Integer**
5. Encodage de `job` : **+ Add step → Unfold → Column : job** (la variable job peut ensuite être supprimé)
6. Exécuter la recette : **Run puis Allez dans Flow** et vérifiez la création de `risk_prepared`

#### Code recipe

1. **Flow → + Recipe → Code → Python**
2. Input : `risk_prepared` ; Output : `risk_prepared` (ou `risk_prepared_code` si remplacer ne passe pas)
3. Coller le code suivant :

```python
# -*- coding: utf-8 -*-
import dataiku
import pandas as pd, numpy as np
from dataiku import pandasutils as pdu

# Read recipe inputs
risk_prepared = dataiku.Dataset("risk_prepared")
risk_prepared_df = risk_prepared.get_dataframe()

risk_prepared_df["log_amount"] = np.log1p(risk_prepared_df["amount"])
risk_prepared_code_df = risk_prepared_df # For this sample code, simply copy input to output


# Write recipe outputs
risk_prepared_code = dataiku.Dataset("risk_prepared_code")
risk_prepared_code.write_with_schema(risk_prepared_code_df)
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

1. Sélectionner `risk_prepared` (ou `risk_prepared_code`) → **Lab → AutoML Prediction → Create Predict model on credit_risk → Choose Algorithms → Create**
2. Target : `credit_risk` ; Prediction type : Two-class classification
3. Features : garder toutes les colonnes utiles, exclure identifiants
4. **Design → Algorithms** : cocher Logistic Regression, Random Forest, XGBoost
5. **Result → Train → Start**
6. **Performance** : observer Accuracy, Recall, Precision, F1, AUC
7. Une fois le lancement terminé, vous pouvez revenir dans **Design** changer les paramètres, les modèles,etc.. afin d'essayer d'obtenir de meilleurs mesures, vous pouvez ensuite naviguer entre sessions, models et tables pour comparer les différents modèles essayés
8. Cliquez sur le modèle que vous souhaitez retenir puis : **Deploy**

### Questions

a. À votre avis, quelle métrique faut-il privilégier en contexte bancaire ?  
b. Y a-t-il des variables problématiques ?  
c. Quel algorithme offre le meilleur compromis Precision/Recall ?  

<details>
  <summary><strong>💡</strong></summary>

a. Recall si le but est de minimiser les faux négatifs (ne pas accorder de prêt risqué)  
b. **Éthique** : attention aux variables sensibles (`sex`, `age`)  
b. XGBoost ou Random Forest selon les runs  

</details>

---

### E. Interprétation et restitution

1. Cliquez sur le nom d'un modèle: Parcourez **Interpretation → Feature importance, Partial dependence**, puis **Performance → Confusion Matrix**
2. Créer un dashboard : Dans la barre du haut, à gauche des ... > Dashboards**+ New Dashboard → risk_dashboard**
3. Ajouter : **New Tile** Feature importance, Confusion matrix(saved model report> Add Model> Saved model report options), et un bloc **Text** avec une synthèse des résultats (Pour Feature Importance, vous pouvez aussi exporter des rapports sous forme de datasets depuis les pages des modèles)
4. Publier le dashboard

### Questions

a. Quelle variable contribue le plus à la prédiction ?  
b. Comment le seuil influence-t-il la matrice de confusion ?

<details>
  <summary><strong>💡</strong></summary>

a. `duration`  
b. Augmenter le seuil = augmente les FP (perte potentielle de clients) / baisser le seuil = augmente les FN (risque de défaut) → seuil à déterminer selon la tolérance au risque.

**Exemple de synthèse pour le dashboard** : 
Ce modèle de scoring prédit le risque de défaut avec une AUC de 0,82 (avis ?). Les variables importantes sont la durée et le montant du prêt, toutes deux corrélées négativement à la probabilité de remboursement. Ce modèle constitue un outil d'aide à la décision pour les conseillers.

</details>

---

### Vérification du Flow

Le Flow doit présenter les noeuds suivants :

<img width="1103" height="234" alt="image" src="https://github.com/user-attachments/assets/ae8364aa-80f5-4869-8062-2bb4c1c76779" />


Si un dataset intermédiaire est créé (par exemple `risk_prepared_code`), il doit être l'entrée de la recette de scoring.

> **Astuce** : tout projet Dataiku peut être exporté sous forme de *bundle* (**Administration → Bundles → Export**) pour être réutilisé sur un autre espace. Cette fonctionnalité permet de partager un pipeline complet sans devoir tout recréer.

<details>
  <summary><strong></strong></summary>

---
   
## Perspective métier

Le scoring de clients estime la probabilité de défaut de paiement d'un client dans le cadre d'octroi d'un crédit.
En contexte bancaire :

- Utilité : pré-tri des dossiers pour étude plus poussée des cas risqués
- Métrique : Recall prioritaire car les faux négatifs sont coûteux
- Éthique : variables sensibles (sexe, âge) à encadrer/contrôler/justifier, voire exclure, en cas de mise en prod réelle
- Variables clés : `duration`, `amount`, `payment_intensity`

---

## Ressources

- Dataiku Academy - *Classification Models* : <https://academy.dataiku.com/latest/course-detail/dataiku-ml-practitioner.html>
- Documentation - *Machine Learning in Dataiku* : <https://doc.dataiku.com/dss/latest/machine-learning/index.html>
- Hastie, Tibshirani, Friedman - *The Elements of Statistical Learning* (PDF libre) : <https://hastie.su.domains/ElemStatLearn/>

</details>

---

<small>[**Page d'accueil**](https://github.com/rsquaredata/atelier_dataiku/blob/main/README.md)</small>
