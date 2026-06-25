<p align="center">
  <img src="banner.svg" alt="AirSentinel — Real-time CO pollution-level classification & WHO alert system" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-0F6E6E?logo=python&logoColor=white">
  <img src="https://img.shields.io/badge/scikit--learn-ML-159191?logo=scikitlearn&logoColor=white">
  <img src="https://img.shields.io/badge/RapidMiner-Visualisation-5BA85B">
  <img src="https://img.shields.io/badge/Streamlit-App-E08A3C?logo=streamlit&logoColor=white">
  <img src="https://img.shields.io/badge/Docker-Conteneuris%C3%A9-2496ED?logo=docker&logoColor=white">
  <img src="https://img.shields.io/badge/M%C3%A9thode-CRISP--DM-1E2A30">
</p>

# AirSentinel — Qualité de l'air & alertes pollution en temps réel

Le monoxyde de carbone est invisible, inodore, et pourtant l'un des polluants les plus dangereux en ville. On ne le sent pas monter. **AirSentinel** part de cette idée simple : si un capteur peut le mesurer, alors une machine devrait pouvoir prévenir le public *avant* que l'air ne devienne nocif.

Concrètement, le projet lit des relevés horaires de qualité de l'air, classe chaque heure en pollution **faible, moyenne ou élevée** selon les seuils de l'Organisation mondiale de la santé, et déclenche une alerte dès que la limite est franchie. Le tout a été construit du début à la fin en suivant la méthodologie **CRISP-DM**, avec Python pour l'analyse et la modélisation, et RapidMiner pour la visualisation.

> **Projet 6** — Master of Science (MSc) M2 Intelligence Artificielle, ECE Paris
> Cours de Data Mining · Jeu de données *Air Quality* (UCI), 9 357 mesures horaires, Italie 2004–2005.

---

## L'histoire du projet

On s'est mis dans la peau d'une agence environnementale qui dispose déjà d'une station de mesure, mais qui ne veut pas seulement *décrire* la pollution passée : elle veut un outil capable de réagir en temps réel.

Le plus gros du travail n'a pas été la modélisation, mais le **nettoyage**. Dans ce jeu de données, un capteur en panne ne laisse pas une case vide : il écrit `-200`. Si on ne le repère pas, cette valeur impossible fausse silencieusement toutes les moyennes, toutes les corrélations et tous les modèles. Une fois ce piège désamorcé et les trous comblés intelligemment (moyenne ou médiane selon l'asymétrie de chaque variable), le reste a pu se construire sur des bases saines.

Pour la prédiction, on a volontairement choisi un **arbre de décision** plutôt qu'un modèle plus performant. La raison est moins technique qu'humaine : une alerte de santé publique doit pouvoir s'expliquer. Avec un arbre, on peut montrer à un citoyen la règle exacte qui a déclenché l'avertissement — ce qu'une forêt aléatoire, plus précise mais opaque, ne permet pas.

---

## Ce que ça donne

| Élément | Résultat |
|---|---|
| Lignes nettoyées | 9 326 (après retrait du capteur NMHC et des lignes mortes) |
| Valeurs manquantes traitées | 16 701 (`-200` → imputation par asymétrie) |
| Précision de l'arbre de décision *(déployé)* | **80 %** |
| Précision de la forêt aléatoire *(comparaison)* | **87 %** |
| Rappel sur la classe dangereuse `HIGH` | **77 %** |
| Heures dangereuses prises pour de l'air sain | **0** — aucune alerte critique manquée |
| Variable la plus prédictive | `PT08.S2(NMHC)` (≈ 0,87) |

Le chiffre dont on est le plus fier n'est pas la précision, mais le **zéro** : pas une seule heure réellement dangereuse n'a été étiquetée « air sain ». Pour un système d'alerte, c'est l'erreur qu'il fallait éviter à tout prix.

---

## Lancer le projet

> Python 3.10 ou plus recommandé.

```bash
# 1. cloner le dépôt
git clone https://github.com/Demba-SowAchta/AirSentinel-.git
cd AirSentinel-

# 2. (optionnel) créer un environnement isolé
python -m venv .venv
# Windows :        .venv\Scripts\activate
# macOS / Linux :  source .venv/bin/activate

# 3. installer les dépendances
pip install -r requirements.txt
```

**Pour explorer l'analyse complète**, ouvrez `code_projet6_python.ipynb` dans Jupyter et exécutez les cellules : valeurs manquantes, séries temporelles, corrélations, matrice de confusion, arbre de décision et clusters s'affichent au fil du notebook, qui exporte au passage la table nettoyée `data/air_quality_clean.csv`. La version déjà exécutée (avec les sorties) se trouve dans `code_executed.ipynb`.

**Pour lancer la démo en temps réel :**

```bash
streamlit run app_streamlit.py
```

L'application s'ouvre sur http://localhost:8501 avec trois onglets : *Données & nettoyage*, *Modèles*, et *Alertes temps réel*. Une version conteneurisée est disponible avec `docker compose up`.

---

## Comment c'est organisé

```
AirSentinel/
├── banner.svg                  # bannière du dépôt
├── README.md                   # ce fichier
├── requirements.txt            # dépendances Python
├── code_projet6_python.ipynb   # l'analyse complète, du nettoyage aux modèles
├── code_executed.ipynb         # le même notebook, déjà exécuté
├── app_streamlit.py            # l'application : interface + alertes temps réel
├── Dockerfile                  # image Docker de la démo
├── docker-compose.yml          # lancement en une commande
├── .gitignore
└── data/
    ├── AirQualityUCI.csv        # le jeu de données brut (UCI)
    └── air_quality_clean.csv    # la table nettoyée, produite par le notebook
```

---

## Les étapes, façon CRISP-DM

De la compréhension du problème jusqu'à la mise en service, le projet suit les six phases de CRISP-DM :

- **Compréhension métier** — partir des seuils OMS du CO et des enjeux de santé publique pour définir ce qu'est une alerte.
- **Compréhension des données** — explorer les séries temporelles et repérer le fameux marqueur `-200` des capteurs en panne.
- **Préparation** — convertir les `-200` en valeurs manquantes, retirer la colonne trop vide et les lignes mortes, imputer selon l'asymétrie, puis ajouter des variables d'heure et de saison.
- **Modélisation** — entraîner un arbre de décision et une forêt aléatoire (supervisé), et un K-Means pour regrouper les périodes (non supervisé).
- **Évaluation** — mesurer la précision, le rappel sur la classe dangereuse, la matrice de confusion et le score silhouette.
- **Déploiement** — livrer une interface Streamlit (et Docker) qui affiche les alertes en temps réel selon l'OMS.

---

## Les seuils OMS utilisés

Les niveaux d'alerte ne sont pas inventés : ils viennent directement des recommandations de l'OMS pour le CO (en mg/m³).

| Fenêtre | Seuil | Ce qu'il déclenche |
|---|---|---|
| 24 h | **4** | frontière `MEDIUM` / `HIGH` — l'alerte se déclenche |
| 8 h | 10 | niveau jugé dangereux |

---

## Auteurs

Projet réalisé par **Demba Sow Achta**, en Master of Science (MSc) M2 Intelligence Artificielle à l'ECE Paris (2025/2026), sous l'encadrement de **Mme Yosra HAJJAJI**.

## Licence

Projet académique. Le jeu de données provient du *UCI Machine Learning Repository* (libre d'usage pour la recherche et l'enseignement).
