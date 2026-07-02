# Module 3 - Automatisation, Agents et LLM (Diet MLOps)

Ce module aborde la mise en production simplifiée des projets Dataiku — une approche que nous appellerons ici, par commodité, « *Diet MLOps* » (ce qui est plus engageant que *Light* ou *Zero*).  
L'idée est d'en présenter les principes essentiels sans entrer dans la complexité d'une infrastructure complète de production.

L'objectif du module est de montrer comment :
- automatiser le réentraînement et le scoring d'un modèle ;  
- intégrer un LLM pour enrichir l'interprétation métier ;  
- superviser le pipeline à l'aide de scénarios et de dashboards.

Cette approche illustre comment Dataiku permet de relier, dans un même environnement, la rigueur technique du MLOps et la lisibilité métier attendue dans un contexte professionnel.

---

<details>
  <summary><strong></strong></summary>

## Rappels théoriques - concepts clés

### Qu'est-ce que le MLOps ?
Le MLOps (Machine Learning Operations) désigne l'ensemble des pratiques qui permettent de déployer, surveiller et maintenir des modèles de machine learning en production. Son objectif est d'industrialiser la chaîne de valeur de la data science: passer du prototype à un système exploitable, fiable et maintenable.

Un pipeline MLOps typique comprend:
1. Ingestion des données (collecte et validation)
2. Préparation et feature engineering
3. Entraînement du modèle
4. Déploiement en production
5. Supervision et alertes
6. Itération et réentraînement

Bénéfices principaux : automatisation, traçabilité, reproductibilité et gouvernance du modèle. Dataiku intègre ces fonctions via les **Scenarios**, le **Model Versioning** et les **Metrics Stores**.

---

### Automatisation : du pipeline au scénario
Un scénario Dataiku est une suite d'actions exécutées automatiquement selon un déclencheur (horaire, changement de dataset, exécution manuelle, etc.). Il permet par exemple de :
- actualiser les données brutes;
- réentraîner un modèle existant;
- lancer des prédictions;
- notifier une équipe (e-mail, webhook, Slack, etc.).

L'automatisation réduit le risque d'erreur humaine et garantit la cohérence du pipeline au fil du temps.

---

### Agents intelligents et orchestration
Les Agents sont des entités logicielles capables d'exécuter des tâches ou de répondre à des requêtes en utilisant des données et des modèles internes.
Dans Dataiku, ils peuvent :
- appeler des modèles prédictifs existants (par ex. `fraud_xgboost_model`);
- utiliser une **LLM Recipe** pour interagir en langage naturel;
- produire un rapport ou exporter des cas à vérifier.

Ils combinent plusieurs briques : automatisation, accès contextuel aux données, et interface conversationnelle.

---

### LLM: modèles de langage et intégration API
Les LLM (Large Language Models) sont des modèles pré-entraînés sur de grandes quantités de texte, capables de générer, résumer ou reformuler du langage naturel. Dataiku permet d'intégrer ces modèles via une **LLM Recipe**, en connectant une API externe comme Mistral, OpenAI ou Claude.

Principe :
1. Créer un connecteur d'API sécurisé (clé privée).
2. Rédiger une prompt adaptée au contexte métier.
3. Exécuter la recette pour générer un texte ou une décision.

Exemple : faire résumer par le LLM les transactions suspectes pour produire un rapport automatique.

---

### Supervision et surveillance des modèles
Une fois en production, les modèles doivent être surveillés pour détecter les dérives:
- Data drift : changement dans la distribution des variables.
- Concept drift : changement de la relation entre les variables et la cible.
- Performance drift : baisse du Recall, de la Precision ou du F1-score.

Dataiku propose un Model Evaluation Store pour centraliser et comparer les métriques au fil du temps.

---

### Glossaire
| Terme | Définition concise |
|-------|--------------------|
| MLOps | Pratiques visant à automatiser et fiabiliser le cycle de vie des modèles ML |
| Pipeline | Enchaînement de tâches : préparation -> modélisation -> prédiction -> surveillance |
| Scénario | Suite d'actions exécutées automatiquement dans Dataiku |
| Agent | Entité logicielle autonome capable d'agir ou de répondre à une demande |
| LLM | Modèle de langage massif, utilisé pour générer ou interpréter du texte |
| Drift | Evolution des données ou du comportement du modèle dans le temps |
| Monitoring | Suivi continu des performances et alertes automatiques |

---

</details>

## TP - Automatisation, Agents, LLM

### A. Création d'un scénario d'automatisation
Objectif : créer un pipeline automatique de mise à jour et d'évaluation du modèle de détection de fraude.

Au préalable se rendre sur <img width="70" height="63" alt="image" src="https://github.com/user-attachments/assets/3e39ed95-5bd3-4fcf-b3ca-cce8ba409301" /> (en haut à gauche)
Puis dans **Administration -> Settings -> Notifications & Integrations -> Messaging channels -> +Add Another Channel** et remplir les données comme ci-dessous :
![SMTP](/imgs/smtp.png)

Mot de passe : mmcf nkmo gkdy udtq

1. Dans le projet, ouvrir **Scenarios -> + New Scenario**.
   - Choisir **Sequence of steps** (par défaut normalement)
   - Nom : `auto_fraud_pipeline`,
   - Scenario ID : `AUTO_FRAUD_PIPELINE` -> **Create**
   - Si Auto-triggers est sur OFF, continuez, sinon mettez le sur OFF 
2. Ajouter une étape : Dans l'onglet Steps -> **Add Step (en bas à gauche) -> Build / Train**, nommez la `Build`
   - Dans la partie Item, ajoutez les datasets : **Add Item ->  Dataset : `fraud_hour` -> ADD**, faite de même pour **`fraud_sampled`** et **`fraud_prediction`**
3. Ajouter une étape : **Add Step -> Build/Train**, nommez la `Train`
   -    - Dans la partie Item, ajoutez le Modèle : **Add Item ->  Model : `Predict is fraud` (c'est possible qu'il se nomme différement) -> ADD**
4. **+ Step -> Send message**, nommez la `Report`
   - Type : Mail
   - Channel : 1 (smtp)
   - Destinataire (To) : votre e-mail
   - Objet (Subject) : `Dataiku [${scenarioName}: ${outcome}]`
   - Message Source -> Changer en `Inline`, puis dans la boîte de saisie, écrire : "Pipeline terminé : modèle fraud réentraîné et scoré."
   - Ajouter vos pièces dans **Attachments -> Add Attachments** (par exemple : votre analyse du modèle)
5. Enregistrer le scénario puis exécuter : **Run**

### Questions
a. Quels avantages apporte l'automatisation à ce stade du projet ?  
b. Pourquoi est-il important d'intégrer la phase de réentraînement dans le scénario ?

<details>
  <summary><strong>💡</strong></summary>

a. L'automatisation réduit les erreurs manuelles, garantit la reproductibilité et libère du temps pour l'analyse.  
b. Le réentraînement périodique permet de s'adapter aux évolutions des données et de prévenir la dérive du modèle.

</details>

> **MLOps - Bonne pratique** : Les **Scenarios** dans Dataiku peuvent être configurés pour ne s’exécuter qu’en cas de succès des étapes précédentes. Cela permet d’éviter des réentraînements en cascade en cas d’erreur et renforce la fiabilité du pipeline.

---

### B. Création d'un Agent avec LLM Recipe
Objectif : créer un agent capable de produire automatiquement une synthèse textuelle des transactions suspectes à partir des prédictions.

Nous allons utiliser Mistral AI cet exercice.
Se rendre au préalable sur https://admin.mistral.ai/organization/api-keys.
Créer un compte et renseigner les différentes informations demandées.

Une fois connecté, se rendre dans les paramètres, puis API Keys : 

<img width="1830" height="723" alt="image" src="https://github.com/user-attachments/assets/2a1a046a-3c46-4c6c-aaad-2a4926f879a9" />

Appuyer sur **Create new key** :
Paramétrer comme vous le souhaitez la clé API.

<img width="616" height="615" alt="image" src="https://github.com/user-attachments/assets/1d91514f-2b7e-48f4-8d8d-38299ff45773" />

**Conseil : Ouvrir un fichier texte et coller la clé.**

*Note : Vous pouvez également entrer une date d'expiration, ce qui peut-être utile si vous décidez de partager une clé à quelqu'un d'autre pour un projet par exemple.
Vous pouvez entrer la date de fin de votre projet par exemple, si vous souhaitez automatiquement rendre la clé inopérante au moment où le projet touche à sa fin.*

1. Afin d'accélérer les requêtes nous allons filtrer le dataset en sélectionnant 1 enregistrement. (Ne pas oublier de build le nouveau dataset)
   - Pour cela, ajouter un **Sample/Filter**
   - Le dataset de sortie devra se nommer `fraud_prediction_filtered`
   - Filtrer sur is_fraud == 1
   - Laisser **No sampling (whole data)**.
2. Dans le **Flow**, sélectionner `fraud_prediction_filtered` -> **+ Recipe -> LLM recipes -> Prompt**
   - Nom : `risk_explanation`
3. **LLM** -> Choisir un model sous **Mistral AI**
   - Si l'API n'est pas configurée :
     - **Administration -> Connections -> + New Connection -> Mistral AI**
     - Clé API : coller la clé générée (`votre_clé_api`)
4. Dans la LLM Prompt :
   - Prompt :
     ```
     Vous êtes un analyste conformité.
     Résumez en quelques lignes pourquoi cette transaction est considérée comme potentiellement frauduleuse, en vous appuyant sur les variables les plus importantes du modèle.
     ```
   - Prompt inputs -> Rentrer dans Description | Column :
       - V24 | V24
       - Amount | Amount
       - V12 | V12
   - Aller dans l'onglet **Input/Output** (en haut à droite) et vérifier que l'input est bien défini : `fraud_prediction_filtered`
   - Revenir sur **Settings**, appuyer sur **Save**
   - Exécuter : **Run**
   - Une fois le job terminé, retourner dans le flow et observer les réponses dans le nouveau dataset (selon le modèle sélectionné vous devriez avoir des réponses plus ou moins intelligentes).
5. Tester un **Agent** : Dérouler l'onglet à côté de celui où se trouve le flow et sélectionner **Agent Tools -> New Agent Tool**
   - Dataset Lookup
   - Nom : `AnalystBot`
   - Appuyer sur **Create**
6. Compléter selon la capture d'écran ci-dessous :
   <img width="1834" height="708" alt="image" src="https://github.com/user-attachments/assets/6b91df76-59c0-4353-b43b-6d66b774b98e" />

   - Exécuter : **Run Test**

### Questions
a. Quel rôle joue le LLM dans ce pipeline ?  
b. En quoi l'utilisation d'un agent complète l'approche MLOps ?  
c. Quels risques ou limites éthiques cela introduit-il ?

<details>
  <summary><strong>💡</strong></summary>

a. Le LLM produit une interprétation en langage naturel des résultats du modèle, facilitant la lecture pour un analyste non technique.  
b. L'agent automatise la décision post-prédiction, ce qui étend la chaîne MLOps jusqu'à la communication.  
c. Risques : hallucinations, biais, fuites de données sensibles. D'où l'importance du contrôle humain et des garde-fous pour la science et la gouvernance des données !

</details>

> **Dépendance aux APIs LLM** : Les **LLM Recipes** reposent sur des services externes (OpenAI, Mistral, etc.). Cela implique des contraintes de confidentialité, de coût et de souveraineté : une clé API exposée ou un quota dépassé peut interrompre la chaîne. Toujours prévoir un *fallback* (scénario de repli) en cas d'indisponibilité de l'API.

---

### C. Supervision, alertes et tableau de bord

1. Créer un dashboard de supervision : **Dashboard -> + New Dashboard -> le nommer `llm_performance`**
   - Ajouter :
     - Performance du modèle (Precision, Recall, AUC-PR)
     - Courbe de ROC
     - "What If ?"
     - Détails de l'algorithme
     - Liste des dernières explications générées par le LLM (indice : il faudra faire une préparation de données pour ne sélectionner que les explications)
2. (Bonus Facile) Créer un scénario de contrôle : **Scénario -> + New Scenario -> le nommer `health_check_llm`**
   Objectif : Exporter le dashboard créé dans le point précédent dans un e-mail

### Questions (Pour pousser plus loin)
a. Pourquoi pourrait-il être intéressant de surveiller à la fois le modèle et l'API LLM ?  
b. Quels indicateurs peuvent signaler une dérive ?  
c. Quelle serait la réaction appropriée en cas de dérive détectée ?

<details>
  <summary><strong>💡</strong></summary>

a. Le modèle et le LLM sont deux points de défaillance : si l'un faiblit, la chaîne complète est compromise.  
b. Baisse du Recall, hausse du taux d'erreur, variation des distributions ou latence excessive de l'API.  
c. Examiner les logs, réentraîner le modèle si nécessaire, ou ajuster les seuils et prompts.

</details>

> **Exemple - Détection d’une dérive** : une variation anormale du Recall accompagnée d’un déplacement de la distribution des variables (`amount`, `hour`) peut révéler une dérive des données. Dans Dataiku, ce suivi se met en place via le **Model Evaluation Store** couplé à un **Scenario** de contrôle automatique.

---

## Perspective métier - mise en production responsable dans Dataiku
La mise en production d'un pipeline Dataiku intégrant des modèles et des LLM engage la responsabilité opérationnelle et la qualité du pilotage.

- Traçabilité et documentation :
  chaque composant du Flow (dataset, recipe, modèle, scénario) est historisé. Le Model Evaluation Store conserve les versions et leurs métriques, permettant d'auditer les choix.
- Validation humaine et gouvernance :
  Dataiku ne remplace pas le contrôle métier. Les recettes automatisées et les agents doivent être supervisés par un analyste ou un data scientist avant toute décision opérationnelle. L'export `cases_to_review.csv` matérialise ce principe.
- Coût et scalabilité :
  l'intégration d'un LLM via API (ici Mistral) a un coût proportionnel aux requêtes. Calibrer la fréquence et cibler les usages à forte valeur ajoutée. Les scénarios peuvent conditionner l'exécution de la LLM Recipe aux seuls cas suspects.
- Surveillance continue et itération :
  le scénario `health_check_llm` illustre la capacité de Dataiku à déclencher des alertes ou des réentraînements automatiques selon des métriques définies.

---

## Ressources
- Documentation Dataiku - Scenarios : https://doc.dataiku.com/dss/latest/scenarios/index.html
- Dataiku - Agents and LLM Integrations : https://doc.dataiku.com/dss/latest/agents/index.html
- Mistral API Reference : https://docs.mistral.ai
- Dataiku Academy - MLOps Concepts : https://academy.dataiku.com



---

<sub>[Retour à la page d'accueil](README.md)</sub>
