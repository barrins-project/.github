# Barrin's Project

[Français](README.md) | [English](README_en.md)

**Barrin's Project** est une initiative personnelle dédiée à l'analyse des données issues des tournois _Magic: The Gathering_, avec un accent particulier sur les formats compétitifs. Le projet couvre aujourd'hui le **Duel Commander** et le **Modern**, et a vocation à s'étendre progressivement à l'ensemble des formats supportés par le pipeline de collecte. L'objectif est de collecter, structurer, analyser et visualiser ces données pour en extraire des tendances, cartographier les archétypes et accompagner le travail de deckbuilding.

## 🗂️ Dépôts publics de l'organisation

* [`MTG_Scraper`](https://github.com/barrins-project/mtg_scraper)
  Outils Python multithreadés (`threading.Thread` + `queue.Queue` + `Lock`) pour le **scraping automatisé** des tournois depuis trois sources — MTGO (via Selenium / webdriver-manager), MTGTop8 et MTGPrime (via requests + BeautifulSoup) — avec extraction des decklists, rounds, standings et notes. CLI exposée via `python -m scraper --source {mtgo,mtgtop8,mtgprime} --date-from YYYY-MM --date-to YYYY-MM`, sortie JSON normalisée par schémas Pydantic.

* [`MTG_Decklist_Cache`](https://github.com/barrins-project/mtg_decklist_cache)
  Dépôt de stockage des données brutes **au format JSON** issues du scraper, classées par source, date et format. Ce dépôt sert également de source pour l'intégration des données dans la base de données du projet principal : **Barrin's API** _(dépôt privé)_.

## 🔒 Dépôts privés de l'organisation

* [`Barrins_API`](https://github.com/barrins-project/barrins_api)
  Backend principal du projet, jouant un rôle de **BFF (Backend For Frontend)** pour `Tolaria_News` et `Tamiyo_Scroll` : API FastAPI (Python 3.14, SQLAlchemy async via `asyncpg` + sync via `psycopg2-binary`, PostgreSQL, Alembic, Pydantic v2) alimentée par `MTG_Decklist_Cache`, Scryfall, MTGTop8, MTGO et GitHub via httpx + tenacity, avec cache Redis. Authentification JWT (`python-jose`) + Argon2id (`argon2-cffi`), qualité assurée par pytest (+ asyncio, cov ≥ 90 % requis), Ruff, mypy strict et bandit.

* [`Tolaria_News`](https://github.com/barrins-project/tolaria_news)
  Frontend dédié à la visualisation du métagame **Duel Commander**, consommant `Barrins_API` via un proxy Vite et structuré autour de cinq vues principales (*Metagame*, *Archetypes*, *Decklists*, *Tournaments*, *Trends*) plus une *Landing Page*. Application React 19 + Vite 8 (React Router 7, TanStack Query 5, Recharts 3) avec un design system maison, des visualisations SVG sur mesure et un hook pre-commit Husky.

* [`Tamiyo_Scroll`](https://github.com/barrins-project/tamiyo_scroll) _(en cours de développement)_
  Frontend dédié à la visualisation du métagame **Modern**, consommant `Barrins_API`. Conçu dans la continuité de `Tolaria_News`, il applique le même pipeline d'analyse (clustering, agrégation, scoring) au format Modern.

## 📊 Projets d'analyse

### 🔹 Clustering & cartographie des archétypes

Pipeline `app/services/ml/clustering.py` opérant sur des fenêtres glissantes (0-90 jours) :

* **Vectorisation des decks** : matrice sparse TF-IDF (`TfidfTransformer` avec `sublinear_tf`) ou vecteur analytique 53 features (cf. ci-dessous).
* **Réduction de dimension** : PCA en haute dimension (≤ 20 composantes) pour le clustering, PCA 2D séparée pour la visualisation.
* **Trois algorithmes au choix** :
  * `KMeans` — k sélectionné par maximisation du **coefficient de silhouette**
  * `DBSCAN` — `eps` auto-estimé au 95ᵉ percentile des distances au k-ième voisin (les points de bruit sont placés dans un cluster 0 dédié)
  * `GaussianMixture` — k sélectionné par minimisation du **BIC**

### 🔹 Agrégation de decklists

Cinq méthodes complémentaires (`aggregation.py` + `aggregation_advanced.py`) pour produire un *deck représentatif* d'un cluster ou du méta global :

| Méthode | Principe |
|---|---|
| **Fréquentielle** | Score d'importance = Σ count(C) × 1/2^|C| sur les n-grammes (ordre 1–3), réduction itérative jusqu'à 100 cartes |
| **Prototype centroïde** | Deck réel le plus proche du centroïde du cluster (toujours légal) |
| **Consensus** | Cartes présentes dans ≥ seuil × N decks |
| **Pondérée par standing** | Poids = 1 / rank (decks sans rang reçoivent le rang max + 1 de leur tournoi) |
| **Temporelle** | Décroissance exponentielle exp(−age / half_life), demi-vie par défaut 14 jours |

### 🔹 Classification fonctionnelle des cartes

Module `features.py` : 14 catégories détectées par regex sur l'oracle text (`text` MTGJson/Scryfall) — *Removal, Counterspell, Card Draw, Ramp, Tutor, Recursion, Protection, Buff/Anthem, Evasion, Token Generation, Discard, Life Gain, Free Spell, Board Wipe*. Une carte peut appartenir à plusieurs catégories ; les decks sans match sont étiquetés `Other`.

### 🔹 Vecteur analytique 53 features

`build_deck_analytical_features` produit un vecteur prêt pour entraînement XGBoost / Gradient Boosting, regroupant :

* **Structure** : ratio terrains, écart au modèle de Karsten (`19.59 + 1.90 × CMC moyen`), singletons, unicité.
* **Mana** : pips normalisés WUBRGC, entropie de Shannon couleur, modal CMC, variance, skewness, kurtosis.
* **Forme de courbe** : densités basse/haute, dominance, concentration top-3.
* **Signaux d'archétype** : densité d'interaction, de menaces, créatures vs non-créatures, potentiel combo, scores d'efficacité (draw, ramp, tutor, free spell).
* **Composition** : densités par type (créature, instant, sorcery, enchantment, artifact, planeswalker), raretés, multicolore.

`build_algorithm_datasets` dérive ensuite quatre variantes optimisées (XGBoost / linéaire / réseau de neurones / clustering).

### 🔹 Rapport méta PDF / Markdown

Pipeline `dc_report.py` : génération de rapports Duel Commander complets (TF-IDF + KMeans + PCA 2D) avec figures Plotly exportées en PNG via Kaleido, mises en page PDF via `fpdf2`, ou export Markdown. Inclut découpage par période de banlist, heatmaps temporelles, analyse différentielle d'un deck personnel vs méta, et suggestions de retraits/ajouts.

### 🔹 Apprentissage automatique (en cours)

Dépendances installées (`scikit-learn`, `xgboost`, `optuna`, `umap-learn`, `mlxtend`, `networkx`) en vue de :

* Prédiction du **macrotype** (aggro, midrange, contrôle, combo, stax…)
* Estimation du **nombre de terrains** optimal d'une liste (modèle Karsten déjà implémenté comme baseline)
* **Scoring de jouabilité** de cartes par similarité fonctionnelle (cosine) avec les decks gagnants — déjà opérationnel dans `dc_report.card_playability()`

## 🧠 Technologies clés

* **Backend (Barrins_API, Python 3.14)** : FastAPI, SQLAlchemy (async asyncpg / sync psycopg2-binary), PostgreSQL, Alembic, Pydantic v2 + pydantic-settings, Redis (hiredis), ijson, unidecode
* **Authentification** : JWT (HS256, `python-jose`), Argon2id (`argon2-cffi`), python-multipart
* **HTTP / intégrations externes** : httpx, tenacity, requests (Scryfall, MTGTop8, MTGO, GitHub)
* **ML / Data Science** : scikit-learn, XGBoost, Optuna, NumPy, pandas, SciPy, Plotly, Kaleido, fpdf2, mlxtend, networkx, umap-learn, pyarrow
* **Frontend (Tolaria_News)** : React 19, Vite 8, React Router 7, TanStack Query 5, Recharts 3
* **Frontend (Tamiyo_Scroll)** : en cours de définition
* **Scraping (MTG_Scraper, Python 3.12+)** : BeautifulSoup4, requests, Selenium, webdriver-manager, pandas, lxml, Pydantic
* **Qualité** : Ruff + mypy strict + bandit (backend), ESLint 10 + Husky (frontend), pytest + pytest-asyncio + pytest-cov (≥ 90 % de couverture imposée)

## 📌 Objectifs

* Construire une **base de données riche et structurée** sur les decks compétitifs MTG, tous formats confondus.
* Aider les joueurs à **visualiser le métagame**, identifier les tendances et affiner leurs listes.
* Fournir un terrain d'expérimentation pour des modèles de **clustering, d'agrégation et de scoring** appliqués au deckbuilding.
* Étendre progressivement le pipeline d'analyse à l'ensemble des formats supportés par le scraper (Standard, Pioneer, Modern, Legacy, Vintage, Pauper, Premodern), en partant du **Duel Commander** comme format de référence et du **Modern** comme premier format additionnel.
