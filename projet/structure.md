# Structure du projet

barrins-project/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py  # Point d'entrée FastAPI
│   │   ├── models.py  # SQLAlchemy models
│   │   ├── schemas.py  # Pydantic schemas
│   │   ├── crud.py  # Fonctions d'accès à la BDD
│   │   ├── database.py  # Connexion à PostgreSQL
│   │   └── routers/
│   │       ├── __init__.py
│   │       ├── tournaments.py
│   │       ├── decks.py
│   │       └── players.py
│   └── requirements.txt  # fastapi, uvicorn, sqlalchemy, psycopg2, etc.
│
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   ├── pages/
│   │   │   ├── Home.tsx
│   │   │   ├── TournamentPage.tsx
│   │   │   ├── DeckPage.tsx
│   │   │   └── PlayerPage.tsx
│   │   ├── services/  # appels à l'API FastAPI
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── vite-env.d.ts
│   └── vite.config.ts
│
├── ml/
│   ├── notebooks/
│   │   ├── clustering.ipynb
│   │   ├── archetype_detection.ipynb
│   │   └── acp_exploration.ipynb
│   ├── scripts/
│   │   ├── extract_features.py
│   │   └── model_utils.py
│   └── models/
│       └── deck_classifier.pkl
│
├── data/
│   └── raw_scraped_data.json  # optionnel, pour stockage temporaire
│
├── .env  # infos de connexion BDD
├── docker-compose.yml  # backend, frontend, base de données
└── README.md
