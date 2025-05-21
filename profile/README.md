# Barrin's Project

[Français](README.md) | [English](README_en.md)

**Barrin's Project** est une initiative personnelle dédiée à l'analyse des données issues des tournois *Magic: The Gathering*, avec un accent particulier sur les formats compétitifs comme le *Duel Commander*. L'objectif est de collecter, structurer, analyser et visualiser ces données pour en extraire des tendances, construire des archétypes et suggérer des améliorations de decklists.

---

## 🗂️ Dépôts de l'organisation

* [`MTG_Viewer`](https://github.com/barrins-project/mtg_viewer) *(accès restreint)*
  Projet **fullstack** basé sur **FastAPI**, **React** et **PostgreSQL**, permettant :

  * la visualisation des decks, tournois et statistiques associées ;
  * la classification des decks par macrotype ;
  * l’annotation et l’évaluation collaborative via une interface web.

* [`MTG_Scraper`](https://github.com/barrins-project/mtg_scraper)
  Outils Python multithreadés pour le **scraping automatisé** des résultats de tournois depuis plusieurs sources (MTGGoldfish, MTGO, MTGTop8…), avec détection des Top 8, extraction des decklists, rounds, standings, etc.

* [`MTG_Decklist_Cache`](https://github.com/barrins-project/mtg_decklist_cache)
  Dépôt de stockage des données brutes **au format JSON** issues du scraper, classées par format et par date. Sert également de source pour l’outil d’import de l’application `MTG_Viewer`.

---

## 📊 Projets d’analyse

### 🔹 Analyse statistique

Exploration des métagames à partir des données collectées :

* **Clustering des decks** :

  * Réduction de dimension avec **ACP** (Analyse en Composantes Principales)
  * Clustering avec **KMeans**, sélection automatique du nombre optimal de clusters via le **coefficient de silhouette**
* **Agrégation de decklists** :

  * Fusion des listes d’un même cluster pour créer un *deck représentatif*
  * Tri des cartes par **popularité** (fréquence d’apparition) et par **synergie** (co-occurrence avec 1–2 autres cartes)
  * Réduction itérative jusqu’à obtenir un deck **légal en Commander** (100 cartes avec 1 ou 2 commandants)

### 🔹 Apprentissage automatique (en cours)

Des projets de **machine learning** sont en phase expérimentale :

* Prédiction du **macrotype** (aggro, midrange, contrôle, etc.)
* Suggestion de cartes ou de courbes de mana optimales
* Détection d’archétypes émergents

---

## 🧠 Technologies clés

* **Backend** : FastAPI, SQLAlchemy, PostgreSQL
* **Frontend** : React, Vite, Tailwind CSS
* **Data Science** : pandas, scikit-learn, XGBoost, matplotlib, seaborn
* **Scraping** : requests, BeautifulSoup, asyncio, multithreading

---

## 📌 Objectifs

* Construire une **base de données riche et ouverte** sur les decks MTG compétitifs.
* Aider les joueurs à **visualiser le métagame**, identifier les tendances et affiner leurs listes.
* Fournir un terrain d’expérimentation pour des modèles de **recommandation et de clustering** appliqués au deckbuilding.
