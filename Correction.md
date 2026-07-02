# 🧩 Corrigé de l'Atelier Dataiku - Master 2 SISE

Ce document présente les **éléments de correction et de vérification** associés à l'atelier Dataiku du **Master 2 Statistique et Informatique pour la Science des Données (SISE)** - Université Lumière Lyon 2.

---

## 📚 Structure générale des corrections

Chaque module (ou TD) de l'atelier est accompagné de ressources permettant de **vérifier, comparer ou importer** les résultats attendus.

| Module / TD | Support de correction | Format | Contenu principal |
|--------------|----------------------|---------|-------------------|
| **TD 0 - Introduction à Dataiku Cloud** | Fiche TD uniquement | `.md` | Découverte guidée (aucune correction séparée) |
| **TD 1 - Scoring clients** | Projet Dataiku exporté | `.zip` | Préparation, modélisation supervisée et interprétation |
| **TD 2 - Détection de fraude** | Projet Dataiku exporté | `.zip` | Gestion du déséquilibre, XGBoost, évaluation, dashboard |
| **TD 3 - Automatisation, Agents et LLM** | Projet Dataiku exporté | `.zip` | Scénarios, agents, MLOps et explicabilité automatisée |
| **TD 4 - API de prédiction** | Vidéo explicative | 🎥 | Déploiement d'une API de prédiction et test de requêtes |

---

## 💾 Accès aux fichiers de correction

L'ensemble des fichiers de correction (projets `.zip` et vidéo du TD4) est disponible sur le **Google Drive officiel de l'atelier** :

🎓 **Lien d'accès :** [➡️ Dossier Drive - Corrigés Atelier Dataiku](https://drive.google.com/drive/folders/1yWZ1AgzsKMRbm0Kn8juQdIkB2b7qRbB7?usp=sharing)

> ⚠️ **Important :** il n'est **pas nécessaire de décompresser** les fichiers `.zip`.  
> Ils peuvent être **importés directement dans Dataiku Cloud** à partir du fichier exporté.

---

## 🧭 Importer un projet Dataiku (.zip)

> 💡 **Astuce** : tout projet Dataiku peut être exporté ou importé pour être partagé entre espaces.  
> Cette fonctionnalité permet de transférer un pipeline complet sans tout recréer manuellement.

### 🔹 Étapes pour importer un projet

1. Dans votre espace Dataiku Cloud, cliquez sur **New Project → Import project**.  
2. Choisissez comme **Source file** le fichier `.zip` téléchargé depuis le Drive.  
3. Cliquez sur **Create** : le projet s'ouvrira avec toutes ses données, recettes et modèles.  
4. (Optionnel) Pour consulter son historique de développement :  
   - Barre supérieure → **Version Control** → historique des modifications.

### 🔹 Étapes pour exporter un projet (rappel)

1. Ouvrez votre projet.  
2. Cliquez sur le **nom du projet** (en haut à gauche, au-dessus du flow).  
3. En haut à droite, allez dans **Actions → Export this project**.  
4. **Cochez toutes les cases** pour inclure les données, recettes, modèles et scénarios.  
5. Cliquez sur **Download** → un fichier `.zip` complet est généré.

---

## 💡 Réponses intégrées dans les fiches TD

Chaque fiche de TD (`.md`) contient les **réponses aux questions** directement intégrées dans le document.  
Elles sont **cachées par défaut** et peuvent être affichées en **dépliant les éléments signalés par une petite ampoule 💡**.

Ces éléments couvrent :
- les explications de concepts (déséquilibre des classes, interprétabilité, etc.),  
- les paramètres de recettes ou modèles,  
- les interprétations des visualisations et résultats.

En suivant attentivement le TD, les réponses apparaissent au fur et à mesure de la lecture.

---

## 🧭 Vérification visuelle - Flows finaux

À la fin de chaque TD, une **capture d'écran du flow final attendu** est fournie.  
Elle permet de vérifier :
- la **structure complète du pipeline**,  
- les **liaisons entre recettes et datasets**,  
- et la **concordance** avec le projet `.zip` fourni en correction.

En suivant les étapes décrites dans le TD, **vous obtiendrez un flow identique** à celui illustré dans la correction.

---

## 🎥 Vidéo explicative - TD4 (API)

Le **TD 4 - API de prédiction** dispose d'une **vidéo de démonstration** (disponible dans le Drive).  
Elle montre en temps réel :
- la création d'un **endpoint d'API** à partir du modèle entraîné,  
- la configuration du **déploiement**,  
- et la **vérification de la prédiction** via des requêtes tests.

---

## 🧩 Objectif du corrigé

Ce corrigé vise à :
- offrir une **référence technique complète** pour chaque module,  
- permettre une **vérification autonome** par les étudiants,  
- et garantir la **reproductibilité** des projets entre espaces Dataiku.

---

## 📦 Contenu du dossier

- **`01_DB_Correction.zip`** → Projet complet du TD1 (*scoring clients*)  
- **`02_DF_Correction.zip`** → Projet complet du TD2 (*détection de fraude*)  
- **`03_.zip`** → Projet complet du TD3 (*automatisation et LLM*)  
- **`04_.mp4`** → Vidéo de démonstration du TD4 (*API de prédiction*)

> 💡 **Astuce :** Il n'est **pas nécessaire de décompresser** les fichiers `.zip`.  
> Ils peuvent être **importés directement** dans votre espace Dataiku Cloud via  
> `New Project → Import project → Source file : votre .zip`.



📦 Tous ces fichiers sont accessibles sur le Drive :  
➡️ [https://drive.google.com/drive/folders/1yWZ1AgzsKMRbm0Kn8juQdIkB2b7qRbB7?usp=sharing](https://drive.google.com/drive/folders/1yWZ1AgzsKMRbm0Kn8juQdIkB2b7qRbB7?usp=sharing)

---

**Usage strictement académique.**  
Toute diffusion ou réutilisation à des fins commerciales est interdite sans autorisation préalable.  

_Ce corrigé fait partie intégrante de l'atelier Dataiku du Master 2 SISE - Université Lumière Lyon 2 (promotion 2025-2026)._

---

<sub>[Retour à la page d'accueil](README.md)</sub>
