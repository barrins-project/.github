# Synthèse du projet

**But :** Outil d’analyse et de visualisation de résultats de tournois Magic (Duel Commander), avec extension prévue vers du ML.

**Utilisateurs :** Toi + petit groupe d’amis, ouverture publique + fonctionnalités premium plus tard.

**Données :** Issues du scrapping MTGO, decks / joueurs / tournois / résultats.

**Stack conseillée :**
- **Frontend** : React + TypeScript
- **Backend** : FastAPI
- **BDD** : PostgreSQL
- **ML** : Jupyter notebooks → intégration via FastAPI plus tard
- **Déploiement (plus tard)** : Docker + Railway / Render ou VPS léger

---

## 🚀 Étapes pour démarrer ton projet

### 1. **Modéliser la base de données**
Exemple de schéma de base :
- `tournaments` : id, date, format, nb joueurs
- `players` : id, pseudo, pays, etc.
- `decks` : id, commandant, couleurs, archétype (tag), joueur_id, tournoi_id, contenu_json
- `results` : deck_id, rang, winrate, points

### 2. **Backend FastAPI**
- Routes pour :
  - `/tournaments` : liste des tournois
  - `/tournaments/{id}` : détails + decks
  - `/decks` : tous les decks
  - `/players/{id}` : stats joueur
- Ajout CORS pour que React puisse faire ses requêtes
- Option : Swagger auto-docs pour tester l’API

### 3. **Frontend React**
- Pages :
  - Accueil → liste des tournois
  - Détail tournoi → liste des decks et résultats
  - Page deck → détail complet (commandant, contenu, stats)
- Bonus : Search / filtre par commandant, couleur, joueur

### 4. **Importer les données**
- Si les données sont dans un repo GitHub → petit script Python d’import et de remplissage BDD

### 5. **Structurer le ML à part**
- Créer un dossier `ml/` avec tes notebooks :
  - Clustering de decks
  - ACP
  - Identification archétypes (à base de features extraites des listes de cartes)
- Plus tard : exporter en `.pkl` et appeler depuis le backend
