# Sentiment Model

## Use case
Classification binaire de sentiment (positif/negatif) sur des avis produits en francais.

## Version
1 (alias: candidate -> champion apres approbation)

## Dataset
reviews.csv -- jeu pedagogique de 20 exemples (10 positifs, 10 negatifs).

## Method
TF-IDF + Logistic Regression (pipeline scikit-learn).

## Parameters
C = 1.0
max_features = 5000
ngram_range = (1, 2)

## Metrics
Accuracy = >= 0.60 (quality gate)
F1 = >= 0.50 (quality gate)

## Quality Gates
Unit tests = passed
Data tests = passed
Model tests = passed
Security checks = passed (bandit + pip-audit)

## Known limitations
- Petit jeu de donnees pedagogique.
- Langue principalement francaise.
- Evaluation non representative d'un contexte industriel.

## Governance
validation_status = passed
approval_status = approved (apres `python promote.py --approve`)
alias = candidate / champion

## Deployment decision
Le modele est promu de candidate a champion uniquement apres :
1. Passage de tous les Quality Gates (tests unitaires, donnees, performance, securite)
2. Validation du tag validation_status=passed dans MLflow Registry
3. Approbation humaine explicite via `python promote.py --approve`
4. Approbation dans l'environnement GitHub Actions production (required reviewer)
