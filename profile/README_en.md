# Barrin's Project

[English](README_en.md) | [Français](README.md)

**Barrin's Project** is a personal initiative focused on analyzing tournament data from *Magic: The Gathering*, with a strong emphasis on competitive formats like *Duel Commander*. The goal is to collect, structure, analyze, and visualize this data to identify trends, build archetypes, and suggest deck improvements.

---

## 🗂️ Repositories

- [`MTG_Viewer`](https://github.com/barrins-project/mtg_viewer) *(restricted access)* 
  A **fullstack application** (FastAPI + React + PostgreSQL) allowing:
  - deck and tournament visualization;
  - macro-archetype classification;
  - collaborative annotations and user feedback.

- [`MTG_Scraper`](https://github.com/barrins-project/mtg_scraper)  
  A Python tool using multithreaded scraping to extract tournament results and decklists from sources like MTGO, MTGTop8, and MTGGoldfish.

- [`MTG_Decklist_Cache`](https://github.com/barrins-project/mtg_decklist_cache)  
  A repository used to **store raw JSON data** scraped from various sources, sorted by format and date.

* [`MTG_Card_Function_Scoring`](https://github.com/barrins-project/mtg_card_function_scoring) *(restricted access)* 
  **Temporary project** to develop a scoring system for MTG cards based on their use (`removal`, `counterspell`, `damage`, etc.).
  Purpose is to integrate this system in the training features of models used by `MTG_Viewer`.
  *This project uses SQLite3 to store data unlike the other repos of this project.*

---

## 📊 Data Analysis Projects

### 🔹 Statistical Analysis

Used to explore and model the tournament metagame:
- **Clustering**:
  - PCA (Principal Component Analysis) to reduce deck dimensions
  - KMeans clustering with automatic number selection via the **silhouette score**
- **Deck Aggregation**:
  - Fusion of decks from the same cluster into a representative *super-deck*
  - Cards sorted by **popularity** (frequency) and **synergy** (co-occurrence with others)
  - Iterative filtering to produce a **legal 100-card Commander deck**

### 🔹 Machine Learning (In Progress)

Several experiments are underway:
- Predict deck **macrotype** (aggro, midrange, control, etc.)
- Recommend optimal **mana curves** or **key cards**
- Detect **emerging archetypes**

---

## 🧠 Tech Stack

- **Backend**: FastAPI, SQLAlchemy, PostgreSQL  
- **Frontend**: React, Vite, Tailwind CSS  
- **Data Science**: pandas, scikit-learn, XGBoost, matplotlib, seaborn  
- **Scraping**: requests, BeautifulSoup, asyncio, multithreading

---

## 🎯 Objectives

- Build a **rich, open database** of competitive MTG decklists
- Help players **visualize metagames** and fine-tune their decks
- Provide a testbed for **ML-driven deck suggestions and clustering**

