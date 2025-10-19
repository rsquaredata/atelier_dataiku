# Atelier Dataiku

Durée : 2h00  
Objectif : Découvrir les fonctions essentielles de **Dataiku Cloud** à travers deux cas métiers en finance.

---

## I. Mise en route (10 min)

### Objectif
Découvrir l’interface et l’organisation d’un projet, créer son premier projet.

1. Connectez-vous à [https://profile.dataiku.com/](https://profile.dataiku.com/).
2. Cliquez sur **Start Free Trial** → **Dataiku Cloud**.
3. Créez un projet :  
   - **+ New Project → Blank Project**  
   - Nom : `Dataiku_Bank_[PrénomNom]`  
   - Key : `DB_[initiales]`
4. Explorez les onglets : **Flow**, **Datasets**, **Recipes**, **Models**, **Dashboards**.
5. Importez le premier jeu de données :  
   - Fichier : `credit_scoring.csv`  
   - Type : *CSV uploaded*  
   - Vérifiez le schéma automatiquement détecté.

📦 **Dataset créé :** `credit_scoring`

**Questions :**
- Quelle est la variable cible dans ce dataset ?  
- Combien d’observations et de variables contient-il ?  
- Quels types de variables sont présentes ?

<details>
<summary>💡 Corrigé</summary>

- Cible : `credit_risk` (1 = bon crédit, 0 = mauvais crédit).  
- Dimensions : 1000 lignes, 21 colonnes.  
- Types : numériques (`duration`, `amount`, `age`, …), catégorielles (`job`, `housing`, …).
</details>

---

## II. Scoring de crédit (60 min)

### Objectif
Créer un modèle de scoring pour prédire la probabilité qu’un client fasse défaut sur son crédit.

---

### A. Explorer les données (10 min)

1. Cliquez sur le dataset `credit_scoring`.  
2. Onglet **Explore** (ou *Statistics* selon la version).  
3. Parcourez les histogrammes automatiques.  
4. Identifiez les variables numériques, catégorielles, et la cible `credit_risk`.

📦 **Dataset utilisé :** `credit_scoring`

**Questions :**
- Quelle variable présente la plus forte asymétrie ?  
- Quelle variable semble la plus corrélée avec le risque de défaut ?  
- Y a-t-il des valeurs manquantes ?  
- Quelle relation entre `amount`, `age` et `duration` ?

<details>
<summary>💡 Corrigé</summary>

- Asymétrie : `amount` et `people_liable`.  
- Corrélation : `duration` et `amount` (corrélation négative).  
- Pas de valeurs manquantes détectées.  
- Plus le montant et la durée augmentent, plus le risque croît.
</details>

---

### B. Préparer le dataset (20 min)

1. Créez une **Prepare Recipe**.  
2. Nettoyez/normalisez les noms de colonnes.  
3. Transformez les variables catégorielles :  
   - Recoder `personal_status_sex` pour créer `Gender_bin` (1 = femme, 0 = homme).  
   - Utilisez un *One-Hot encoding* pour la variable `job`.  
4. Créez une nouvelle variable :  
   `payment_intensity = amount / duration`
5. Supprimez uniquement les colonnes redondantes ou peu informatives (ex. `telephone` ou `people_liable` si jugées inutiles).  
6. Ajoutez une **Code Recipe** (Python) :

```python
import pandas as pd, numpy as np
df = input_dataset.copy()
df["log_income"] = np.log1p(df["income"])
output_dataset = df
```

📦 **Dataset créé :** `credit_scoring_prepared`

**Questions :**
- Combien de colonnes restent après préparation ?  
- Quelle transformation pourrait améliorer la lisibilité du jeu ?  
- Pourquoi utiliser `log1p` plutôt qu’un simple `log` ?  
- Quelle est la moyenne et l’écart-type de `log_amount` ?

<details>
<summary>💡 Corrigé</summary>

- Environ 25 colonnes après encodage.  
- On peut centrer-réduire (`Standardize`) les variables numériques.  
- `log1p` gère les zéros et stabilise les petites valeurs.  
- Moyenne ≈ 7.2, écart-type ≈ 0.8 (à vérifier selon échelle).  
</details>

---

### C. Construire un modèle AutoML (20 min)

1. Dans le **Flow**, cliquez sur `credit_scoring_prepared`.  
2. Ouvrez le **Lab → Visual analysis → Predict**.  
3. Variable cible : `credit_risk`.  
4. Laissez Dataiku sélectionner les *features*.  
5. Testez plusieurs algorithmes : *Logistic Regression*, *Random Forest*, *XGBoost*.  
6. Comparez leurs performances dans **Model results → Metrics**.

📦 **Dataset créé :** `credit_scoring_predictions` (via *Apply Model*)

**Questions :**
- Quel modèle obtient le meilleur AUC ?  
- Quelles variables dominent ? Certaines posent-elles un problème éthique ?  
- Quelle métrique privilégier en contexte bancaire ?

<details>
<summary>💡 Corrigé</summary>

- Meilleur AUC : souvent *Random Forest* ou *XGBoost*.  
- Variables dominantes : `duration`, `amount`, `age`.  
- Variables sensibles : `sex`, `age`. À surveiller.  
- Métrique clé : *Recall* ou *F1* selon le risque métier.
</details>

---

### D. Interpréter et restituer (10 min)

1. Ouvrez l’onglet **Model Results → Interpretation**.  
2. Consultez *Feature importance* et *Partial Dependence*.  
3. Créez un Dashboard “Credit Scoring” avec :  
   - un graphique d’importance des variables,  
   - une matrice de confusion,  
   - un texte de conclusion.  
4. Publiez le dashboard (*View → Publish*).

📦 **Dashboard créé :** `Credit_Scoring_Dashboard`

**Question :**
> En une phrase : comment une banque pourrait-elle utiliser ce modèle ?

<details>
<summary>💡 Corrigé</summary>

> Pour préqualifier automatiquement les dossiers clients et prioriser les cas à analyser manuellement.
</details>

---

## III. Détection de fraude (40 min)

### Objectif
Identifier les transactions suspectes dans un flux de paiements.

### A. Importer et explorer (10 min)

1. Importez le fichier `creditcard.csv`.  
2. Vérifiez les colonnes : `Time`, `Amount`, `Class`.  
3. Renommez `Class` → `is_fraud`.  
4. Comptez les fraudes (`is_fraud = 1`).

📦 **Dataset créé :** `creditcard`

**Questions :**
- Quelle proportion de fraudes ?  
- Les fraudes concernent-elles les gros montants ?

<details>
<summary>💡 Corrigé</summary>

- ≈ 0,17 % de fraudes (492 / 284807).  
- Les montants moyens sont légèrement plus élevés, mais la médiane est inférieure.
</details>

---

### B. Échantillonner pour l’analyse (10 min)

1. Créez une **Sample/Filter Recipe**.  
2. Réduisez à 10 000 lignes, conservez une proportion réaliste de fraudes (~0,5 %).  
3. Créez une colonne `hour = floor(Time / 3600) % 24`.

📦 **Dataset créé :** `creditcard_sampled`

**Questions :**
- Pourquoi faut-il équilibrer le jeu d’apprentissage ?  
- Quelles stratégies alternatives (SMOTE, undersampling, class weights) ?

<details>
<summary>💡 Corrigé</summary>

- Équilibrer améliore la sensibilité du modèle (*Recall*) et évite que l’Accuracy masque le déséquilibre.  
- Alternatives : *SMOTE* (ajoute des cas minoritaires synthétiques), *undersampling*, ou *class weights*.
</details>

---

### C. Lancer un modèle AutoML (15 min)

1. Depuis `creditcard_sampled`, créez un projet **Predict (Lab)**.  
2. Cible : `is_fraud`.  
3. Testez plusieurs modèles et comparez les métriques AUC, Recall, Precision, F1.  
4. Appliquez le meilleur modèle → `creditcard_predictions`.

📦 **Dataset créé :** `creditcard_predictions`

**Questions :**
- Pourquoi l’Accuracy n’est-elle pas une bonne métrique ici ?  
- Quel modèle maximise le Recall sans trop sacrifier la Precision ?  
- Comment pondérer les métriques selon le coût métier ?

<details>
<summary>💡 Corrigé</summary>

- L’Accuracy reste élevée même si le modèle ignore les fraudes (classe majoritaire).  
- *XGBoost* ou *Random Forest* avec *class weights* donnent souvent le meilleur Recall.  
- Pondération : privilégier le Recall si le coût d’une fraude non détectée est plus élevé.
</details>

---

### D. Visualisation finale (5 min)

1. Créez un Dashboard “Fraude” avec :  
   - score de fraude par montant,  
   - diagramme “fraude vs non fraude”,  
   - répartition horaire (`hour`).

📦 **Dashboard créé :** `Fraud_Dashboard`

**Questions :**
- Les transactions à haut montant sont-elles plus frauduleuses ?  
- À quelle heure les fraudes sont-elles les plus fréquentes ?

<details>
<summary>💡 Corrigé</summary>

- Tendance dispersée : quelques gros montants très suspects, mais beaucoup de petites fraudes.  
- Pic nocturne (2–5h), probablement en heures creuses.
</details>

---

### Transition vers le BONUS

> Nous allons maintenant enrichir notre pipeline avec des composants d’intelligence générative et d’automatisation, pour passer d’un modèle prédictif à un système intelligent complet.

---

## IV. BONUS – Intelligence générative, Agents et Automatisation

### A. Explication automatique des décisions

1. Sélectionnez `credit_scoring_predictions`.  
2. Créez une **LLM Prompt Recipe** :  

```text
Tu es analyste risques dans une banque.
Résume en 2-3 phrases pourquoi ce dossier a été classé "risqué".
Mentionne les 2 facteurs les plus importants.
```

📦 **Dataset créé :** `credit_explanations`

---

### B. Génération d’un rapport synthétique

1. Sélectionnez le dataset d’évaluation du modèle (`evaluation_store`).  
2. Créez une **LLM Prompt Recipe** avec :  

```text
Tu es data scientist dans une banque.
Rédige une synthèse sur le modèle :
- performances (AUC, F1, Recall, Precision)
- variables clés
- limites et pistes d’amélioration
```

📦 **Dataset créé :** `executive_summary.md`

---

### C. Agent intelligent : Risk Analyst Bot

1. Dans **Automation → Agents**, créez un agent `RiskAnalystBot`.  
2. Définissez un **Trigger** : `is_fraud_pred = 1`.  
3. Ajoutez des actions :  
   - Générer un résumé automatique (LLM).  
   - Établir une checklist métier.  
   - Exporter les cas à examiner vers `cases_to_review.csv`.  
   - Notifier l’équipe via e-mail.

📦 **Dataset créé :** `cases_to_review.csv`

---

### D. Automatisation du pipeline

1. Dans **Automation → Scenarios**, créez un scénario `Auto_Fraud_Pipeline`.  
2. Étapes :  
   - *Build dataset* (`creditcard_sampled`)  
   - *Run model training* (`FraudDetection_XGBoost`)  
   - *Run LLM Recipe* (`RiskExplanation`)  
   - *Export CSV* (`cases_to_review.csv`)  
   - *Send mail* (résumé du rapport).  
3. Ajoutez un *Trigger* horaire (tous les jours à 8h).

---

### E. Suivi des performances (Health Check)

1. Créez un dataset `llm_logs` contenant les prompts et résultats LLM.  
2. Calculez : taux de succès, temps moyen, cohérence.  
3. Créez un Dashboard “LLM Performance”.  
4. Ajoutez un scénario “HealthCheck_LLM” déclenché chaque soir pour alerter si >5 % d’erreurs.

📦 **Dataset créé :** `llm_logs`

---

### F. Vue d’ensemble du pipeline final

1. Préparation et scoring des crédits/fraudes  
2. Explication automatique des décisions  
3. Rapport exécutif généré par LLM  
4. Agent virtuel réactif  
5. Scénario automatisé  
6. Monitoring des performances IA

---

> En combinant **Dataiku DSS**, les **LLM Recipes**, les **Agents** et les **Scenarios**, vous obtenez un **pipeline complet et automatisé** : les modèles prédisent, expliquent, agissent et se surveillent.
