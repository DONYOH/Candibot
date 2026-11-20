# 🤖 CandiBot

**Automatisation du traitement des candidatures par e-mail (RPA)**

Projet RPA · MBA ESG Paris · MBA2 BDIA Janv26 – Classe 01 · Septembre 2026

![Couverture CandiBot](assets/slide-01-couverture.png)

## Équipe

| Nom | Rôle |
|---|---|
| Antoine Mekawy | Projet RPA |
| Sofiane EL MORABIT | Automatisation · RPA |
| Eric DONYOH | MBA ESG – Big Data & Intelligence Artificielle (MBDIA) |

## Sommaire

1. [Le problème actuel](#1-le-problème-actuel)
2. [Diagramme de flux](#2-diagramme-de-flux)
3. [La solution CandiBot](#3-la-solution-candibot)
4. [Démonstration du prototype](#4-démonstration-du-prototype)
5. [Bénéfices attendus](#5-bénéfices-attendus)
6. [Cas d'intervention humaine](#6-cas-dintervention-humaine)
7. [Prototype et ressources](#7-prototype-et-ressources)
8. [Planning et ROI](#8-planning-et-roi)
9. [Lancer le dashboard](#9-lancer-le-dashboard)
10. [Liens utiles](#10-liens-utiles)

---

## 1. Le problème actuel

Les candidatures arrivent directement par e-mail. Pour chaque nouvelle candidature, une personne doit :

- ouvrir le message et télécharger le CV ;
- relever les informations principales du candidat et les reporter dans un fichier de suivi ;
- ranger le CV dans le bon dossier ;
- répondre au candidat pour confirmer la réception.

Ce traitement prend environ **5 minutes par candidature**. Les difficultés au quotidien :

- du temps passé sur des tâches simples et répétitives ;
- des erreurs de saisie (mauvaise adresse, mauvais fichier) ;
- une candidature non enregistrée peut être oubliée ;
- une réponse tardive si le recruteur est indisponible.

![Le problème actuel](assets/slide-03-probleme.png)

## 2. Diagramme de flux

Déclencheur : réception d'un e-mail de candidature. Le flux suit ensuite quatre étapes standardisées :

```
Réception e-mail → Lecture → Classement CV → Mise à jour du suivi → Confirmation
```

| # | Étape | Action |
|---|---|---|
| 1 | Lecture de l'e-mail | Ouvrir le message, lire le nom du candidat, son adresse et le poste recherché |
| 2 | CV et classement | Télécharger le CV joint et l'enregistrer dans le bon dossier |
| 3 | Mise à jour du suivi | Ajouter une ligne au fichier de suivi et remplir les informations du candidat |
| 4 | Confirmation | Envoyer une réponse au candidat pour accuser réception |

En cas d'exception, une personne reprend la main.

![Diagramme de flux du processus](assets/slide-04-diagramme-de-flux.png)

## 3. La solution CandiBot

**CandiBot** est un robot *unattended* de traitement des candidatures, développé avec **UiPath Studio Web**.

Quand une candidature arrive via [e-telecom-soft.com](https://e-telecom-soft.com), CandiBot :

1. récupère les informations principales de l'e-mail (nom, adresse e-mail, poste recherché) ;
2. les ajoute automatiquement dans le fichier de suivi ;
3. enregistre le CV dans le bon dossier (Google Drive) ;
4. envoie automatiquement un message de confirmation.

La personne chargée du recrutement garde la partie qui compte : lire le CV et décider si le profil mérite d'être approfondi. Une page **Streamlit** permet de visualiser l'ensemble des candidatures.

- **Déclencheur** : réception d'un e-mail
- **Applications** : boîte e-mail de test · fichier de suivi · dossier de stockage
- **Outil** : UiPath Studio Web

**Pourquoi cette tâche est idéale à automatiser :**

- un processus répétitif, presque identique d'une candidature à l'autre ;
- des données déjà numériques (e-mails et CV), sans ressaisie nécessaire ;
- l'humain se concentre sur ce qu'il fait le mieux : analyser et décider.

![La solution CandiBot](assets/slide-05-solution.png)

## 4. Démonstration du prototype

### Étape 1 : postuler via le formulaire en ligne

Le candidat renseigne son nom complet, son adresse e-mail, le poste souhaité et son CV (PDF, 5 Mo maximum) sur [e-telecom-soft.com](https://e-telecom-soft.com/).

![Formulaire de candidature](assets/formulaire-candidature.png)

### Étape 2 : suivre les candidatures dans le dashboard Streamlit

Le dashboard lit les données enregistrées dans le fichier de suivi (Google Sheet) et propose :

- la recherche d'un CV par nom ;
- le flux du pipeline par étape ;
- des indicateurs : total des candidatures, en attente, traitées, postes distincts ;
- des filtres par statut et par poste ;
- une actualisation manuelle ou automatique.

![Dashboard Streamlit](assets/dashboard-streamlit.png)

## 5. Bénéfices attendus

Au-delà du temps gagné, CandiBot fiabilise chaque étape du traitement.

| Bénéfice | Détail |
|---|---|
| Gagner du temps | Sur les tâches répétitives de saisie et de classement |
| Réduire les erreurs | Moins de mauvaises adresses ou de fichiers mal nommés |
| Mieux classer les CV | Chaque CV rangé au bon endroit, sans oubli |
| Répondre plus vite | Une confirmation envoyée dès la réception |
| Un suivi à jour | Le fichier de suivi reflète toujours la réalité |
| Plus de temps d'analyse | Le recruteur se concentre sur les profils |

![Bénéfices attendus](assets/slide-09-benefices.png)

## 6. Cas d'intervention humaine

Le robot ne gère pas tout seul. Une personne reprend la main dans les situations suivantes, afin qu'aucune candidature ne soit mal traitée :

| Situation | Conséquence |
|---|---|
| Le CV est absent | Aucun fichier à traiter, rien à classer |
| Le nom n'est pas indiqué | Impossible d'identifier le candidat |
| Le poste n'est pas précisé | Le suivi ne peut pas être complété |
| Le fichier joint ne fonctionne pas | Format illisible ou corrompu |
| Le robot rencontre une erreur | Vérification manuelle nécessaire |
| Analyse réelle du profil | Reste toujours du ressort du recruteur |

![Cas d'intervention humaine](assets/slide-10-limites.png)

## 7. Prototype et ressources

Le prototype est une version simple du processus, réalisée pour valider l'approche avant d'aller plus loin.

**Ce que fait le prototype :**

- utiliser un e-mail de candidature de test ;
- récupérer le nom, l'e-mail et le poste recherché ;
- ajouter les informations dans le fichier de suivi ;
- enregistrer le CV dans un dossier ;
- afficher un message de fin de traitement.

**Ressources :**

- **Équipe** : une personne pour le besoin métier, une pour le prototype, une pour le diagramme et le PowerPoint ;
- **Outils** : UiPath Studio Web, une boîte e-mail de test, un fichier de suivi, un dossier de stockage ;
- **Données de test** : des CV fictifs et un dossier pour les enregistrer.

> Hors périmètre du prototype : le robot ne gère pas tous les cas possibles. Seules certaines parties du processus sont développées.

![Prototype et ressources](assets/slide-11-prototype.png)

## 8. Planning et ROI

**Hypothèse : 100 candidatures reçues par mois.**

**Planning :**

1. Définir le problème et le fonctionnement actuel
2. Préparer un e-mail, un CV et un fichier de suivi de test
3. Créer le prototype avec UiPath
4. Tester le robot et corriger les erreurs
5. Réaliser le diagramme et la présentation
6. Vérifier l'ensemble du projet avant l'envoi

**Temps de traitement par candidature :** 5 min en manuel contre 0,5 min avec CandiBot.

| Indicateur | Valeur |
|---|---|
| Temps gagné | 7 h 30 / mois |
| Économies | ~2 250 € / an |
| ROI estimé | 181,25 % |
| Amortissement du coût initial | ~3 mois |

Coût de création : 500 € · maintenance : ~25 €/mois · coût total année 1 : ~800 €. Gain net estimé la première année : ~1 450 €. Le coût initial de 500 € est amorti en ~3 mois.

> Ces chiffres sont des estimations utilisées uniquement dans le cadre du projet.

![Planning et ROI](assets/slide-12-planning-roi.png)

## 9. Lancer le dashboard

Le dashboard Streamlit lit les données du fichier de suivi (projet `candibot dashbord`). Depuis le dossier du projet :

```bash
streamlit run app.py
```

L'application est ensuite disponible sur <http://localhost:8501>.

## 10. Liens utiles

- Formulaire de candidature : lien retirer - image capture (formulaire-candidature.png)
- [Fichier de suivi des candidatures (Google Sheets)](https://docs.google.com/spreadsheets/d/1XCmiAU6BNcPe7IN2KXMVziPHHXxINVpQF0UAP83z42Y/edit?usp=drive_link)
- [Dossier de dépôt des CV (Google Drive)](https://drive.google.com/drive/u/0/folders/1mY0eP-OizQY4ltrKBTddfDuobw8AOSiA)

---

*Projet RPA · MBA ESG Paris · Juin 2026*
