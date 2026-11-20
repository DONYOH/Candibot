# 🤖 CandiBot — Automatisation RPA de Collecte et Centralisation de CV

![UiPath](https://img.shields.io/badge/UiPath-Automation%20Cloud-FA4616?style=for-the-badge&logo=uipath&logoColor=white)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success?style=for-the-badge)
![Architecture](https://img.shields.io/badge/Architecture-Serverless%20%2F%20Cloud-blue?style=for-the-badge)

## 📋 Table des matières
1. [Présentation du projet](#-présentation-du-projet)
2. [Problématique et valeur ajoutée](#-problématique-et-valeur-ajoutée)
3. [Architecture technique et flux de données](#-architecture-technique-et-flux-de-données)
4. [Prérequis et dépendances](#-prérequis-et-dépendances)
5. [Configuration sur UiPath Automation Cloud](#-configuration-sur-uipath-automation-cloud)
6. [Détail du Workflow (UiPath Studio)](#-détail-du-workflow-uipath-studio)
7. [Structure du projet](#-structure-du-projet)
8. [Déploiement et exécution](#-déploiement-et-exécution)
9. [Gestion des erreurs et bonnes pratiques](#-gestion-des-erreurs-et-bonnes-pratiques)
10. [Évolutions futures (Roadmap)](#-évolutions-futures-roadmap)
11. [Auteur](#-auteur)

---

## 🎯 Présentation du projet

**CandiBot** est un robot d'automatisation des processus métier (RPA) développé sous **UiPath Studio** et conçu pour s'exécuter de manière 100 % autonome dans un environnement **UiPath Automation Cloud (Serverless / Unattended)**.

Son rôle principal est d'automatiser la réception, le filtrage, le renommage standardisé et l'archivage centralisé des candidatures reçues par e-mail (Gmail) directement vers un espace de stockage sécurisé sur **UiPath Orchestrator** (*Storage Bucket*).

---

## 💡 Problématique et valeur ajoutée

Le traitement manuel des candidatures par e-mail présente plusieurs goulots d'étranglement :
* **Perte de temps** liée à l'ouverture unitaire des e-mails et au téléchargement des pièces jointes.
* **Risque d'écrasement ou de perte** lorsque plusieurs candidats nomment leur fichier `CV.pdf`.
* **Dépendance aux postes locaux** rendant le partage des CV difficile entre les recruteurs et les équipes Data/RH.

**Ce que CandiBot apporte :**
* **Zéro intervention humaine** : Surveillance continue ou planifiée de la boîte de réception.
* **Standardisation** : Renommage automatique incluant l'adresse de l'expéditeur (`<email>_<nom_fichier>.pdf`).
* **Indépendance matérielle** : Aucune dépendance à un disque dur local (`C:\`), exécution native sur conteneurs Cloud éphémères.

---

## 🏗️ Architecture technique et flux de données

L'architecture repose sur un transit en mémoire temporaire au sein du conteneur d'exécution Cloud avant le transfert définitif vers l'Orchestrator :

```text
┌─────────────────────────────┐
│      Boîte Gmail Cible      │
│   (Integration Service)     │
└──────────────┬──────────────┘
               │  1. Lecture des e-mails (CurrentEmail)
               ▼
┌─────────────────────────────┐
│  Download Email Attachments │
│     Filtre : "*.pdf"        │
└──────────────┬──────────────┘
               │  2. Extraction vers la zone de transit
               ▼
┌─────────────────────────────┐
│  Système de fichiers Cloud  │
│ System.IO.Path.GetTempPath()│
└──────────────┬──────────────┘
               │  3. Lecture via Attachments(0).LocalPath
               ▼
┌─────────────────────────────┐
│     Upload Storage File     │
│ Renommage : From + "_" + CV │
└──────────────┬──────────────┘
               │  4. Stockage persistant
               ▼
┌─────────────────────────────┐
│    UiPath Orchestrator      │
│ Storage Bucket: CV_Candidates│
└─────────────────────────────┘
```

---

## ⚙️ Prérequis et dépendances

### Environnement logiciel
* **UiPath Studio** (version 2023.10 ou supérieure, compatibilité *Windows* ou *Cross-Platform*).
* Accès à un tenant **UiPath Automation Cloud** avec licence d'exécution *Serverless* ou *Unattended*.

### Packages NuGet requis
| Package | Version recommandée | Rôle dans le projet |
| :--- | :--- | :--- |
| `UiPath.GSuite.Activities` | `>= 2.6.0` | Interaction avec l'API Gmail via Integration Service. |
| `UiPath.System.Activities` | `>= 23.10.0` | Gestion des variables, boucles et activités *Storage Bucket*. |

---

## ☁️ Configuration sur UiPath Automation Cloud

Avant d'exécuter le robot, deux ressources doivent être configurées sur votre espace **UiPath Cloud** :

### 1. Connexion Gmail (Integration Service)
1. Rendez-vous dans **Integration Service** > **Connectors** > **Gmail**.
2. Créez et authentifiez la connexion avec le compte cible (ex. : `scoeric3@gmail.com`).
3. Vérifiez que le statut de la connexion est actif (*Connected*).

### 2. Création du Storage Bucket (Orchestrator)
1. Rendez-vous dans **Orchestrator** et sélectionnez votre dossier de travail (*Folder*).
2. Allez dans l'onglet **Storage Buckets** > **Add Storage Bucket**.
3. Sélectionnez **Orchestrator** comme fournisseur de stockage.
4. Nommez le bucket exactement : **`CV_Candidates`**.

---

## 🔄 Détail du Workflow (UiPath Studio)

### Étape 1 : Lecture des e-mails (`For Each Email`)
* **Connexion** : Connecteur Gmail configuré dans Integration Service.
* **Filtre** : Recherche des e-mails non lus ou contenant des pièces jointes (ex. : sujet contenant `"Candidature"` ou `"CV"`).
* **Variable d'itération** : `CurrentEmail`.

### Étape 2 : Téléchargement temporaire (`Download Email Attachments`)
* **Email message** : `CurrentEmail`
* **Filter by filename** : `"*.pdf"` *(ignore les images de signature ou autres formats non désirés)*.
* **Options > Exclude inline attachments** : `False` *(ou `True` pour ignorer les logos intégrés dans le corps du mail)*.
* **Options > Destination path** :
  ```vbnet
  System.IO.Path.GetTempPath()
  ```
* **Options > If a file already exists** : `Replace`
* **Output > Attachments** : `Attachments` *(Liste de type `IEnumerable<GmailAttachmentLocalItem>`)*.

### Étape 3 : Vérification et Téléversement Cloud (`Upload Storage File`)
*(Condition recommandée : `Attachments IsNot Nothing AndAlso Attachments.Count > 0`)*
* **Storage bucket name** : `"CV_Candidates"`
* **File** :
  ```vbnet
  Attachments(0).LocalPath
  ```
* **Destination path** : *(Laisser vide pour enregistrer à la racine du bucket)*
* **Uploaded file name** :
  ```vbnet
  CurrentEmail.From.Address + "_" + Attachments(0).Name
  ```

---

## 📂 Structure du projet

```text
CandiBot/
├── Main.xaml                 # Point d'entrée principal du workflow
├── project.json              # Configuration des dépendances et métadonnées UiPath
├── .entities/                # Définitions des objets Integration Service
└── README.md                 # Documentation complète du projet
```

---

## 🚀 Déploiement et exécution

1. **Publication** : Depuis UiPath Studio, cliquez sur **Publish** pour envoyer le package NuGet vers le flux **Orchestrator Tenant Processes Feed**.
2. **Création du processus** : Dans Orchestrator (**Automations** > **Processes**), créez un processus lié au package `CandiBot`.
3. **Exécution** :
   * **Manuelle** : Lancez le job depuis Orchestrator en sélectionnant un robot *Cloud Serverless*.
   * **Planifiée (Trigger)** : Configurez un déclencheur horaire (ex. : toutes les 30 minutes) ou événementiel (à la réception d'un e-mail via Integration Service).

---

## 🛡️ Gestion des erreurs et bonnes pratiques

* **Absence de chemin local en dur** : L'utilisation de `System.IO.Path.GetTempPath()` garantit que le robot ne rencontrera jamais d'erreur `GSuiteException: The specified path does not exist` lors de son exécution sur les serveurs Cloud d'UiPath.
* **Nettoyage automatique** : Les conteneurs Cloud étant éphémères, les fichiers temporaires sont automatiquement purgés à la fin du job, garantissant la conformité RGPD et la sécurité des données.
* **Gestion multi-pièces jointes** : Si un candidat envoie plusieurs PDF (CV + Lettre de motivation), il est recommandé d'encapsuler l'activité `Upload Storage File` dans une boucle `For Each` sur la collection `Attachments`.

---

## 🔮 Évolutions futures (Roadmap)

* [ ] **Extraction de données par IA (Document Understanding / GenAI)** : Parser automatiquement le contenu des PDF (nom, compétences, années d'expérience) pour structurer un fichier JSON/CSV.
* [ ] **Alimentation d'un Data Warehouse / Base de données** : Injecter les métadonnées des candidats dans une table SQL ou BigQuery pour alimenter un tableau de bord Power BI RH.
* [ ] **Notification automatique** : Envoyer un accusé de réception personnalisé au candidat et notifier l'équipe recrutement sur Slack/Teams.

---

## 👤 Auteur

**Kossi Eric DONYOH**  
*Data Analyst & Intégrateur de Systèmes d'Information (RPA / Data / IA)*
