# TP4 – CI/CD, Tests, Quality Gates et Gouvernance MLOps

**ENSAF Fès – ISDIA | MLOps 2026–2027**  
Réalisé par : **JOHN MUHAMMED** | Encadré par : **Pr. M. AIRAJ**

---

## Objectif

Mettre en place un pipeline MLOps complet pour un classifieur de sentiments (avis produits en français) :

```
Code → Tests → Quality Gates → MLflow → Staging → Approbation → Production
```

---

## Structure du projet

```
mlops-tp4-cictcd/
├── src/
│   ├── data.py              # chargement et split stratifié
│   ├── model.py             # pipeline TF-IDF + LogisticRegression
│   └── train.py             # entraînement + métriques
├── tests/
│   ├── test_unit.py         # tests unitaires        (G1)
│   ├── test_data.py         # qualité des données    (G2)
│   └── test_model.py        # performance modèle     (G3, G4)
├── data/
│   └── reviews.csv          # 40 avis FR (20 pos / 20 nég)
├── train_and_register.py    # MLflow tracking + registre
├── promote.py               # promotion candidate → champion
├── params.yaml              # hyperparamètres centralisés
├── model_card.md            # documentation du modèle
├── requirements.txt
└── .github/workflows/
    └── mlops-pipeline.yml   # pipeline CI/CD GitHub Actions
```

---

## Démarrage rapide

```bash
# 1. Cloner et créer l'environnement
git clone https://github.com/Maha-Jr10/mlops-tp4-cictcd.git
cd mlops-tp4-cictcd
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / macOS
pip install -r requirements.txt

# 2. Lancer le serveur MLflow (terminal séparé)
mlflow server --host 127.0.0.1 --port 5000

# 3. Exécuter les tests
pytest tests/ -v --cov=src --cov-report=term-missing

# 4. Entraîner et enregistrer le modèle
set MLFLOW_TRACKING_URI=http://127.0.0.1:5000   # Windows
# export MLFLOW_TRACKING_URI=http://127.0.0.1:5000
python train_and_register.py

# 5. Promouvoir en production (approbation humaine simulée)
python promote.py             # vérification sans promotion
python promote.py --approve   # promotion → alias champion
```

---

## Quality Gates

| Gate | Condition | Seuil |
|------|-----------|-------|
| G1 | Tests unitaires | 100 % passés |
| G2 | Qualité des données | 100 % passés |
| G3 | Accuracy | ≥ 0.60 |
| G4 | F1-score | ≥ 0.50 |
| G5 | Couverture de code | ≥ 70 % |
| G6 | Scan sécurité (bandit + pip-audit) | 0 issue |

Résultats obtenus : **accuracy = 0.90 · F1 = 0.909 · couverture = 100 %**

---

## Pipeline CI/CD

Déclenché sur chaque push vers `main` ou `develop` :

```
quality (lint + tests + bandit + pip-audit)
    └─► staging  (environment: staging)
            └─► production  (environment: production — approbation requise)
```

Voir [`.github/workflows/mlops-pipeline.yml`](.github/workflows/mlops-pipeline.yml).

---

## MLflow

| Élément | Valeur |
|---------|--------|
| Expérience | `sentiment-classification` |
| Modèle enregistré | `sentiment-model` |
| Alias candidat | `candidate` |
| Alias production | `champion` |
| Tags | `validation_status=passed`, `use_case=sentiment` |

Interface : [http://127.0.0.1:5000](http://127.0.0.1:5000)

---

## Paramètres (`params.yaml`)

```yaml
data:
  test_size: 0.25
  random_state: 22
model:
  max_features: 5000
  ngram_range: [1, 2]
  C: 1.0
  max_iter: 1000
quality_gate:
  min_accuracy: 0.60
  min_f1: 0.50
  min_coverage: 70
```

---

## Dépendances principales

| Package | Rôle |
|---------|------|
| scikit-learn | TF-IDF + LogisticRegression |
| mlflow | Tracking + registre de modèles |
| pytest / pytest-cov | Tests + couverture |
| ruff | Linting |
| bandit | Analyse de sécurité statique |
| pip-audit | Audit des vulnérabilités CVE |
