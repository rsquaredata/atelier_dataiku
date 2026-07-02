# Atelier Dataiku - Déploiement d'un modèle en API

## Objectif du module

Ce module clôture notre atelier en vous guidant dans la création, le test et le déploiement d'un service API sous **Dataiku Cloud**, à partir d'un modèle de prédiction déjà entraîné (`predict_fraudulent` dans le projet *MLOps Quick Start*).

L'objectif est de transformer un modèle de machine learning en une API REST capable de répondre à des requêtes en temps réel.

---

## Étape 1 - Accès à Dataiku Cloud

Pour commencer :

1. Rendez-vous sur le lien suivant : [https://launchpad-dku.app.dataiku.io/](https://launchpad-dku.app.dataiku.io/)
2. Si un **nœud (node)** apparaît :
   - S'il est en **Running**, vous pouvez passez à l'étape 2 
   - Sinon, cliquez sur **⋯ → Turn On** pour le démarrer.
3. Une fois le workspace lancé, vous êtes prêt à créer votre premier projet Dataiku.

> 💡 **Astuce** :  
> Si vous ne parvenez pas à accéder directement au Launchpad, connectez-vous via [https://profile.dataiku.com/](https://profile.dataiku.com/) puis relancez le lien ci-dessus.

---

## Étape 2 - Activation de l'API Node

1. Dans la **barre latérale gauche**, cliquez sur **Extensions**.  
2. Sélectionnez **Add an extension**.  
3. Choisissez **API Node**, puis cliquez sur **Add**.  
4. Revenez ensuite dans l'onglet **Overview**.

Vous verrez qu'un **nouveau node** intitulé **API Node** est apparu à côté du **Design Node**.

> 💡 **Remarque** :  
> Le **Design Node** sert à la conception (datasets, préparation, modélisation),  
> tandis que l'**API Node** permet de déployer les modèles sous forme d'API REST.

---

## Étape 3 - Création du projet à partir d'un template

1. Depuis la page **Overview**, ouvrez votre **Design Node** en cliquant sur **Open Instance**.  
2. Dans l'interface Dataiku :
   - Cliquez sur **+ New Project** (en haut à droite).  
   - Sélectionnez **Learning Project**.  
   - Choisissez le template **MLOps Quick Start**.  
   - Cliquez sur **Go to Flow** pour accéder au diagramme du projet.
3. Une fois sur le flow, cliquez sur **Flow Actions** (en bas à droite), puis sur Build All (vous pouvez suivre l'execution dans la partie job (le triangle dans la barre du haut)

> 💡 **Astuce** :  
> Ce projet d'exemple présente le cycle complet de mise en production d'un modèle :  
> préparation → entraînement → déploiement → suivi → API.

---

## Étape 4 - Création d'une API à partir du modèle

1. Dans le **Flow**, repérez le modèle **predict_fraudulent**.  
2. Cliquez dessus pour ouvrir sa fiche détaillée.  
3. Dans la **barre d'outils à droite**, cliquez sur **Create API**.  
4. Renseignez les champs suivants :
   - **API Service name** : `job_posting_api`
   - **Endpoint ID** : `predict_fake_job`
5. Cliquez sur **Append** pour ajouter le modèle au service d'API.

> 💡 **Astuce** :  
> Cette étape transforme votre modèle de prédiction en un service d'API réutilisable.  
> Il pourra ensuite être déployé sur l'**API Node** et testé en conditions réelles.

---

## Étape 5 - Test et publication de l'API

1. Dans l'interface de l'API , ouvrez l'onglet **Test Queries** (dans le menu gauche).  
2. Cliquez sur **Add Queries** :
   - Indiquez un nombre de requêtes, par exemple **5**.  
   - Sélectionnez **From Dataset : Test**. (Si vous n'avez pas build le flow à l'étape 3, cela peut planter
   - Cliquez sur **Add**.
3. Les requêtes de test sont alors générées automatiquement à partir du jeu de données **Test**.
4. Cliquez sur **Run Test Queries** pour les exécuter et visualiser les résultats.
5. Allez dans **Security** → mettez **Authorization** sur **Public** (pour rendre l'API accessible sans clé).

---

### Publication de l'API

1. Dans la barre supérieure,cliquez sur **⋯ → API Designer** pour revenir à la page principale de conception.
2. Cliquez sur **Publish On Deployer**.
3. Laissez le **nom de version par défaut** (ex : *v1*).  
4. Cliquez sur **Publish** pour déployer votre service API.

> ✅ **Résultat attendu** :  
> Votre modèle est désormais publié sur l'**API Node** et accessible sous forme d'API REST.  
> Il peut être interrogé pour prédire automatiquement si une offre d'emploi est frauduleuse.

---

## Étape 6 - Déploiement sur le Deployer local et test final

1. Dans la barre supérieure de Dataiku, cliquez sur l'icône **en forme de carré (9 points)** <img width="452" height="90" alt="Capture d'écran 2025-10-22 015100" src="https://github.com/user-attachments/assets/2c5d7dad-4b29-4c93-8423-d55fd9621614" /> , en haut à droite.  
   > Cette icône permet d'accéder à d'autres modules : Automation, Deployer, etc.
2. Sélectionnez **Local Deployer**.
3. Dans le menu, cliquez sur **Deploying API Services**.
4. Dans la barre gauche :
   - Cliquez sur la **version** de votre API (généralement **v1**).  
   - Cliquez ensuite sur **Deploy**.
5. Confirmez de nouveau en cliquant sur **Deploy**.
6. Vous êtes redirigé vers l'espace de **monitoring** de votre API déployée :
   - suivi des performances,  
   - nombre d'appels,  
   - état des requêtes.
7. Pour tester :
   - Ouvrez **Run and Test** dans la barre de gauche.  
   - Cliquez sur **Run All** pour exécuter toutes les requêtes de test.

> ✅ **Résultat attendu** :  
> Tous les tests s'exécutent correctement.  
> Votre modèle est désormais **déployé, testé et opérationnel** via un service d'API REST dans Dataiku.

---

## Étape 7 - Création d'un Recipe pour appeler l'API

1. Revenez sur le **Flow** du projet.  
2. Cliquez sur le dataset **Test**.  
3. Dans le panneau de droite, cliquez sur **+ Recipe → Code → Python**  
   (ou **R** selon vos préférences).  
4. Dans la configuration :
   - Sélectionnez **Test** comme dataset d'entrée.  
   - Créez un **nouveau dataset de sortie**, nommé `pred_api`.  
5. Cliquez sur **Create Recipe** pour finaliser la création.

> 💡 **Astuce** : ce recipe permettra d'interroger directement l'API et de générer des prédictions en lot ou à la demande.

---

## Étape 8 - Exécution du Recipe et récupération des prédictions

1. Dans le **recipe code**, copiez-collez le code Python suivant :  

```python
# -*- coding: utf-8 -*-
import dataiku
import pandas as pd
import dataikuapi

# --- Connexion à ton API Node (A modifier) ---
client = dataikuapi.APINodeClient(
    "https://api-4f915596-501230dc-dku.eu-west-3.app.dataiku.io",
    "job_postings"  # nom du service
)

# --- Lecture du dataset d'entrée ---
test = dataiku.Dataset("test")
test_df = test.get_dataframe()

# --- Préparation d'une liste pour stocker les prédictions ---
results = []

# --- Boucle sur chaque ligne du dataset ---
for i, row in test_df.iterrows():
    record_to_predict = row.to_dict()  # transforme la ligne en dictionnaire
    try:
        prediction = client.predict_record("predict_fake_job", record_to_predict)
        # Ajoute les résultats au dictionnaire
        record_to_predict["prediction_result"] = prediction.get("result")
        record_to_predict["prediction_proba"] = prediction.get("proba") or prediction.get("probabilities", None)
    except Exception as e:
        # En cas d'erreur, on garde trace de la ligne problématique
        record_to_predict["prediction_result"] = None
        record_to_predict["prediction_proba"] = None
        record_to_predict["error"] = str(e)
    results.append(record_to_predict)

# --- Conversion en DataFrame ---
pred_api_df = pd.DataFrame(results)

# --- Écriture du dataset de sortie ---
pred_api = dataiku.Dataset("pred_api")
pred_api.write_with_schema(pred_api_df)

print(f"✅ Prédictions terminées pour {len(results)} lignes.")

```

> ⚠️ **Remarque** : adaptez les noms des datasets si vos datasets d'entrée/sortie diffèrent (`Test` → dataset d'entrée, `pred_api` → dataset de sortie).

2. Avant de l'exécuter, il faut juste modifier les paramètres de connexion à l'API (voir la ligne 7 : **client = dataikuapi.APINodeClient(...)**)
   - Pour récupérer les paramètres de l'API, vous devez vous rendre sur le service de déploiement d'API, puis cliquez sur votre API, et cliquez dans l'onglet **Sample Code** (barre de gauche).
   - Ensuite, le langage est sur Shell par défaut, changez le sur **Python** et vous pouvez copier la ligne 3 -> **client = dataiku.APINodeClient("...","...")** et la coller dans votre script python à la place de ce qu'il y a ligne 7.
3. Cliquez sur **Run** pour exécuter le recipe.  
4. Retournez dans le **Flow** et visualisez le dataset `pred_api` :
   - Construisez le flow ou les sub-flows si nécessaire.  
   - Observez l'avant-dernière ligne : la réponse de l'API a bien été récupérée pour chaque ligne du dataset **Test**.

> ⚠️ **Limitation** :  
> Cependant vous pouvez aussi constater que la colonne stocke toute la réponse, malheureusement nous n'avons pas réussi à faire mieux que ça, cependant si vous vous en sentez le courage, vous pouvez relever le défi et résoudre ce problème
> La colonne de sortie contient actuellement **toute la réponse brute** de l'API.  
> 💡 **Défi optionnel** : vous pouvez essayer de parser la réponse pour extraire uniquement la prédiction finale.

## Flow Final 
<img width="967" height="448" alt="image" src="https://github.com/user-attachments/assets/2c1478b2-84ee-4aad-a147-bdf9b9938f2b" />

## Étape 9 - Monitoring des requêtes API

1. Dans la barre supérieure de Dataiku, cliquez sur l'icône **en forme de carré (9 points)** en haut à droite.  
2. Sélectionnez **Deployer → Deploying API Services**.  
3. Dans la section **Deployment**, cliquez sur la carte correspondant à l'API que vous avez mise en production.  
4. Vous accédez alors à l'espace de **monitoring** de l'API, où vous pouvez observer :  
   - les requêtes envoyées depuis votre script Python,  
   - l'état de chaque requête (succès ou erreur),  
   - le temps de réponse et autres métriques utiles pour suivre la performance de l'API en production.

> 💡 **Astuce** : ce monitoring vous permet de vérifier que votre pipeline de prédiction fonctionne correctement en production et d'identifier rapidement toute anomalie ou lenteur.

---

## Bilan du module

Vous avez appris à :

- Configurer un **workspace Dataiku Cloud** avec un **API Node**.  
- Créer un projet MLOps à partir d'un **template**.  
- Générer et tester une **API de prédiction** à partir d'un modèle existant.  
- Déployer cette API sur le **Local Deployer** et la tester en production.  
- Créer un **recipe code** pour interroger l'API et récupérer les prédictions.

> 🧠 **Prochaine étape** : explorer la section **Monitoring** pour suivre la performance du modèle en continu (drift, latence, appels récents…).

---

<sub>[Retour à la page d'accueil](README.md)</sub>

