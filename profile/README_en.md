# Barrin's Project
[Français](README.md) | [English](README_en.md)

**Barrin's Project** is a personal initiative dedicated to analyzing tournament data from _Magic: The Gathering_, with an exclusive focus on the competitive **Duel Commander** format. The goal is to collect, structure, analyze, and visualize this data in order to extract trends, map archetypes, and support deckbuilding work.

## 🗂️ Public repositories

* [`MTG_Scraper`](https://github.com/barrins-project/mtg_scraper)
  Multithreaded Python tools (`threading.Thread` + `queue.Queue` + `Lock`) for **automated scraping** of tournament data from three sources — MTGO (via Selenium / webdriver-manager), MTGTop8 and MTGPrime (via requests + BeautifulSoup) — extracting decklists, rounds, standings, and notes. CLI exposed through `python -m scraper --source {mtgo,mtgtop8,mtgprime} --date-from YYYY-MM --date-to YYYY-MM`, with normalized JSON output backed by Pydantic schemas.

* [`MTG_Decklist_Cache`](https://github.com/barrins-project/mtg_decklist_cache)
  Storage repository for the raw **JSON** data produced by the scraper, organized by source, date, and format. This repository also serves as the source for data ingestion into the main project's database: **Barrin's API** _(private repository)_.

## 🔒 Private repositories

* [`Barrins_API`](https://github.com/barrins-project/barrins_api)
  Main backend of the project, acting as a **BFF (Backend For Frontend)** for `Tolaria_News`: FastAPI service (Python 3.14, async SQLAlchemy via `asyncpg` + sync via `psycopg2-binary`, PostgreSQL, Alembic, Pydantic v2) fed by `MTG_Decklist_Cache`, Scryfall, MTGTop8, MTGO, and GitHub through httpx + tenacity, with Redis caching. JWT authentication (`python-jose`) + Argon2id (`argon2-cffi`), quality enforced by pytest (+ asyncio, ≥ 90 % coverage required), Ruff, strict mypy, and bandit.

* [`Tolaria_News`](https://github.com/barrins-project/tolaria_news)
  Frontend dedicated to Duel Commander metagame visualization, consuming `Barrins_API` through a Vite proxy and structured around five main views (*Metagame*, *Archetypes*, *Decklists*, *Tournaments*, *Trends*) plus a *Landing Page*. React 19 + Vite 8 application (React Router 7, TanStack Query 5, Recharts 3) with a custom design system, hand-crafted SVG visualizations, and a Husky pre-commit hook.

## 📊 Analysis projects

### 🔹 Clustering & archetype mapping

Pipeline `app/services/ml/clustering.py` operating on sliding windows (0–90 days):

* **Deck vectorization**: sparse TF-IDF matrix (`TfidfTransformer` with `sublinear_tf`) or 53-feature analytical vector (see below).
* **Dimensionality reduction**: high-dimensional PCA (≤ 20 components) for clustering, separate 2D PCA for visualization.
* **Three algorithms to choose from**:
  * `KMeans` — k selected by maximizing the **silhouette coefficient**
  * `DBSCAN` — `eps` auto-estimated at the 95ᵗʰ percentile of k-nearest-neighbor distances (noise points are placed in a dedicated cluster 0)
  * `GaussianMixture` — k selected by minimizing the **BIC**

### 🔹 Decklist aggregation

Five complementary methods (`aggregation.py` + `aggregation_advanced.py`) to produce a *representative deck* for a cluster or for the overall metagame:

| Method | Principle |
|---|---|
| **Frequency-based** | Importance score = Σ count(C) × 1/2^|C| over n-grams (order 1–3), iterative reduction down to 100 cards |
| **Centroid prototype** | Real deck closest to the cluster centroid (always legal) |
| **Consensus** | Cards present in ≥ threshold × N decks |
| **Standing-weighted** | Weight = 1 / rank (decks without a rank receive their tournament's max rank + 1) |
| **Temporal** | Exponential decay exp(−age / half_life), default half-life of 14 days |

### 🔹 Functional card classification

Module `features.py`: 14 categories detected via regex on oracle text (`text` from MTGJson/Scryfall) — *Removal, Counterspell, Card Draw, Ramp, Tutor, Recursion, Protection, Buff/Anthem, Evasion, Token Generation, Discard, Life Gain, Free Spell, Board Wipe*. A card can belong to several categories; cards with no match are labeled `Other`.

### 🔹 53-feature analytical vector

`build_deck_analytical_features` produces a vector ready for XGBoost / Gradient Boosting training, grouping:

* **Structure**: land ratio, deviation from Karsten's model (`19.59 + 1.90 × average CMC`), singletons, uniqueness.
* **Mana**: normalized WUBRGC pips, Shannon color entropy, modal CMC, variance, skewness, kurtosis.
* **Curve shape**: low/high densities, dominance, top-3 concentration.
* **Archetype signals**: interaction density, threat density, creatures vs non-creatures, combo potential, efficiency scores (draw, ramp, tutor, free spell).
* **Composition**: densities by type (creature, instant, sorcery, enchantment, artifact, planeswalker), rarities, multicolor.

`build_algorithm_datasets` then derives four optimized variants (XGBoost / linear / neural network / clustering).

### 🔹 PDF / Markdown metagame report

Pipeline `dc_report.py`: generation of complete Duel Commander reports (TF-IDF + KMeans + 2D PCA) with Plotly figures exported as PNG through Kaleido, laid out as PDF via `fpdf2`, or exported as Markdown. Includes segmentation by banlist period, temporal heatmaps, differential analysis of a personal deck vs the meta, and removal/addition suggestions.

### 🔹 Machine learning (in progress)

Installed dependencies (`scikit-learn`, `xgboost`, `optuna`, `umap-learn`, `mlxtend`, `networkx`) intended for:

* Predicting a deck's **macro-type** (aggro, midrange, control, combo, stax…)
* Estimating the **optimal number of lands** for a list (Karsten's model is already implemented as a baseline)
* **Card playability scoring** via functional (cosine) similarity with winning decks — already operational in `dc_report.card_playability()`

## 🧠 Key technologies

* **Backend (Barrins_API, Python 3.14)**: FastAPI, SQLAlchemy (async asyncpg / sync psycopg2-binary), PostgreSQL, Alembic, Pydantic v2 + pydantic-settings, Redis (hiredis), ijson, unidecode
* **Authentication**: JWT (HS256, `python-jose`), Argon2id (`argon2-cffi`), python-multipart
* **HTTP / external integrations**: httpx, tenacity, requests (Scryfall, MTGTop8, MTGO, GitHub)
* **ML / Data Science**: scikit-learn, XGBoost, Optuna, NumPy, pandas, SciPy, Plotly, Kaleido, fpdf2, mlxtend, networkx, umap-learn, pyarrow
* **Frontend (Tolaria_News)**: React 19, Vite 8, React Router 7, TanStack Query 5, Recharts 3
* **Scraping (MTG_Scraper, Python 3.12+)**: BeautifulSoup4, requests, Selenium, webdriver-manager, pandas, lxml, Pydantic
* **Quality**: Ruff + strict mypy + bandit (backend), ESLint 10 + Husky (frontend), pytest + pytest-asyncio + pytest-cov (≥ 90 % coverage enforced)

## 📌 Goals

* Build a **rich and structured database** of competitive Duel Commander decks.
* Help players **visualize the metagame**, identify trends, and refine their lists.
* Provide an experimentation ground for **clustering, aggregation, and scoring** models applied to deckbuilding.
* Eventually extend the analysis pipeline to the other formats already supported by the scraper (Standard, Pioneer, Modern, Legacy, Vintage, Pauper, Premodern) once Duel Commander automation has stabilized.
