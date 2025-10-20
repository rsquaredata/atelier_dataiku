# Atelier Dataiku Cloud – Master 2 SISE

## Présentation générale

Ce dépôt constitue le support officiel de l'**atelier Dataiku Cloud**, élaboré dans le cadre du **Master 2 Statistique et Informatique pour la Science des Données (SISE)** de l'**Université Lumière Lyon 2**.

L'objectif de cet atelier est d'offrir une **mise en pratique complète des outils Dataiku** à travers des études de cas inspirées de problématiques financières réelles. Il articule des dimensions techniques (préparation, modélisation, automatisation) et méthodologiques (explicabilité, gouvernance, MLOps, IA générative).

---

## Objectifs pédagogiques

- Comprendre les principes fondamentaux de la plateforme **Dataiku** et son positionnement dans l'écosystème data.  
- Mettre en œuvre un **processus ETL** complet : ingestion, préparation, enrichissement et visualisation de données.  
- Expérimenter la **modélisation supervisée** (scoring de crédit, détection de fraude) via l'interface AutoML et les recettes visuelles.  
- Appliquer les concepts de **classification déséquilibrée**, d'**évaluation de modèles** et d'**interprétabilité**.  
- Initier les étudiants aux fondements du **MLOps**, de l'**automatisation** et des **LLM Recipes** dans Dataiku Cloud.  
- Développer une réflexion critique sur les apports et les limites de la data science industrialisée.

---

## Prérequis techniques

- Compte actif sur [**Dataiku Cloud (essai gratuit)**](https://www.dataiku.com/product/get-started/) (validité 14 jours).  
- Navigateur web à jour (Chrome, Firefox ou Safari).  
- Connaissances de base en Python, R ou SQL. 
- Accès internet stable et capacité de téléchargement de fichiers CSV.

---

## Organisation de l'atelier

L'atelier est structuré en quatre modules progressifs.

| Module | Contenu principal |
|---------|-----------------------------|
| [**Module 0 – Introduction à Dataiku Cloud**](https://github.com/rsquaredata/atelier_dataiku/blob/main/00_introduction_dataiku.md) | Démonstration guidée (taux de change BCE), création de projet et exploration de l'interface. |
| [**Module 1 – Scoring clients**](https://github.com/rsquaredata/atelier_dataiku/blob/main/01_credit_scoring.md) | Modélisation supervisée (classification binaire), exploration, préparation et interprétation. |
| [**Module 2 – Détection de fraude**](modules/02_fraud_detection.md) | Traitement d'un jeu de données déséquilibré, XGBoost, métriques avancées et tableau de bord. |
| [**Module 3 – Automatisation, Agents et LLM**](modules/03_automation_agents_llm.md) | Introduction au MLOps, automatisation de pipelines, agents de surveillance et explicabilité par LLM. |

---

## Jeux de données

| Fichier | Description | Hébergement |
|----------|--------------|--------------|
| `fx_rates_sample.csv` | Échantillon réel de taux de change publiés par la Banque centrale européenne (20/10/2025). | GitHub `/datasets/` |
| `credit_scoring.csv` | Données anonymisées de scoring client. | Google Drive |
| `creditcard.csv` | Données de transactions pour la détection de fraude. | Google Drive |

Les fichiers `credit_scoring.csv` et `creditcard.csv` proviennent de jeux publics sous licence **CC BY-NC-SA**, et sont réservés à un usage pédagogique.  
Le fichier `fx_rates_sample.csv` est issu d'un export authentique de la Banque centrale européenne, reproduit ici à des fins démonstratives.

---

## Liens utiles

- [**Site officiel Dataiku**](https://www.dataiku.com/)  
- [**Dataiku Academy**](https://academy.dataiku.com/)  
- [**Documentation Dataiku**](https://doc.dataiku.com/dss/latest/)  
- [**Slides de présentation de l'atelier**](https://www.canva.com/design/DAG1-dy-VF0/LKKCkJhkuKQ2k0Eo9ZxreQ/edit)  
- [**Jeu de données Scoring - Google Drive**](https://drive.google.com/file/d/1OdeW5F1lQZLGxk5RcTRPnXKO63Y6DqjD/view?usp=drivesdk)  
- [**Jeu de données Fraude - Google Drive**](https://drive.google.com/file/d/1zXWCW8hSSDz8wRZ1rvrD-3nqExQTtLp_/view?usp=drivesdk)

---

## Équipe

- **Constantin Rey-Coquais**
- **Cyrille Pecnik**
- **Rina Razafimahefa**
- **Yassine Cheniour**

---

## Remarques et prolongements

Certaines fonctionnalités avancées de Dataiku (plugins externes, versioning collaboratif, scénarios complexes, gouvernance multi-projets) ne sont pas couvertes dans cet atelier.  
Elles sont néanmoins introduites conceptuellement dans le **Module 3 – Automatisation, Agents et LLM**, pour préparer aux pratiques du MLOps moderne et à l'intégration de l'**IA générative** dans les chaînes analytiques.

--- 
Usage strictement académique et non commercial.
