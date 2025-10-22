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

Au préalable se rendre dans **Administration -> Settings -> Notifications & Integrations -> Messaging channels** et remplir les données comme ci-dessous :
![SMTP](/imgs/smtp.png)

Mot de passe : mmcf nkmo gkdy udtq

1. Dans le projet, ouvrir **Scenarios -> + New Scenario**.
   - Nom : `auto_fraud_pipeline`
   - Trigger : **Manually** (pour commencer)
2. Ajouter une étape : **+ Step -> Build / Dataset(s)**
   - Sélectionner les datasets : `fraud_hour`, `fraud_sampled`, `fraud_prediction`
3. Ajouter une étape : **+ Step -> Train model(s)**
   - Sélectionner le modèle `fraud_xgboost_model`
4. **+ Step -> Send message**
   - Destinataire : votre e-mail
   - Message  : "Pipeline terminé : modèle fraud réentraîné et scoré."
   - Ajouter vos pièces dans **Attachments** (par exemple : votre analyse du modèle)
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

1. Dans le **Flow**, sélectionner `fraud_prediction` -> **+ Recipe -> LLM -> Create Recipe**
   - Nom : `risk_explanation`
2. Choisir **Mistral API** comme fournisseur LLM
   - Si l'API n'est pas configurée :
     - **Administration -> Connections -> + New Connection -> Mistral AI**
     - Clé API : coller la clé fournie (`votre_clé_api`)
3. Dans la LLM Prompt :
   - Prompt :
     ```
     Vous êtes un analyste conformité.
     Résumez en quelques lignes pourquoi cette transaction est considérée comme potentiellement frauduleuse, en vous appuyant sur les variables les plus importantes du modèle.
     ```
   - Entrée : `fraud_prediction`
   - Exécuter : **Run**
4. Tester un **Agent** : **+ New -> Agent Tools**
   - Nom : `AnalystBot`
   - Action : Tester en lui donnant des instructions du type "Retourne les lignes où is_fraud = 1"
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

1. Créer un dashboard de supervision : **Dashboard -> llm_performance**
   - Ajouter :
     - Historique des versions de modèles (Model Evaluation Store)
     - Performance du modèle (Precision, Recall, AUC-PR)
     - Indicateur de dérive des données
     - Liste des dernières explications générées par le LLM
2. (Bonus Facile) Créer un scénario de contrôle : **Scénario -> + New Scenario -> health_check_llm**
   Objectif : Exporter le dashboard créé dans le point précédent dans un e-mail

### Questions (Pour poussez plus loin)
a. Pourquoi pourrait-il être intéressant de surveiller à la fois le modèle et l'API LLM ?  
b. Quels indicateurs peuvent signaler une dérive ?  
c. Quelle serait la réaction appropriée en cas de dérive détectée ?

<details>
  <summary><strong>💡</strong></summary>

a. Le modèle et le LLM sont deux points de défaillance : si l'un faiblit, la chaîne complète est compromise.  
b. Baisse du Recall, hausse du taux d'erreur, variation des distributions ou latence excessive de l'API.  
c. Examiner les logs, réentraîner le modèle si nécessaire, ou ajuster les seuils et prompts.

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

</details>

---

<small>[**Page d'accueil**](https://github.com/rsquaredata/atelier_dataiku/blob/main/README.md)</small>
