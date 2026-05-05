# PROJECT ROADMAP

Ce document décrit **comment construire AWS Scan Pro étape par étape**, avec une logique orientée MVP, extensibilité et qualité d'architecture.

---

## 1. Principes directeurs

Le projet doit respecter les principes suivants :

1. **Extensible**
   - ajout simple de nouveaux services AWS,
   - ajout simple de nouvelles règles,
   - ajout simple de nouvelles remédiations.

2. **Lisible**
   - séparation claire des responsabilités,
   - conventions de nommage cohérentes,
   - modules petits et spécialisés.

3. **Testable**
   - logique métier isolée,
   - règles indépendantes,
   - fonctions unitaires facilement testables.

4. **Sûr**
   - pas d'auto-remédiation destructive dans le MVP,
   - accès AWS en lecture seule au début,
   - validation explicite pour les actions sensibles.

5. **Produit**
   - architecture pensée pour devenir un outil vendable,
   - reporting clair,
   - logique métier orientée impact business.

---

## 2. Ordre de création recommandé

Nous allons construire le projet dans cet ordre :

### Étape 1 — Foundation
Créer la structure technique de base :
- README
- documentation roadmap
- configuration Python
- arborescence backend
- API minimale

### Étape 2 — Core platform
Créer les modules transverses :
- configuration
- logging
- exceptions
- types communs

### Étape 3 — AWS access
Créer l'accès AWS :
- chargement des credentials
- AssumeRole
- sessions boto3
- découverte des régions

### Étape 4 — Discovery
Créer les premiers collecteurs :
- EC2
- EBS
- S3
- IAM
- CloudTrail

### Étape 5 — Findings engine
Créer le moteur de règles :
- classe de base
- registre de règles
- premières règles sécurité/coût

### Étape 6 — Storage
Créer la persistance :
- modèles de base
- scans
- findings
- ressources

### Étape 7 — API métier
Créer les endpoints :
- healthcheck
- comptes
- scans
- findings

### Étape 8 — Reporting
Créer les exports :
- JSON
- CSV
- résumés exécutifs

### Étape 9 — Remediation
Créer les recommandations :
- texte,
- snippets boto3,
- templates Terraform.

---

## 3. MVP fonctionnel détaillé

Le MVP doit permettre :

- d'enregistrer un compte AWS cible,
- d'assumer un rôle,
- de lister les régions actives,
- de scanner quelques services,
- de générer quelques findings,
- de renvoyer les résultats via API.

### Services du MVP
- IAM
- S3
- EC2
- EBS
- RDS
- Security Groups
- CloudTrail

### Exemples de règles MVP
- bucket S3 public,
- security group ouvert à `0.0.0.0/0` sur SSH,
- volume EBS détaché,
- Elastic IP non associée,
- ressource sans tag `Owner`,
- CloudTrail absent.

---

## 4. Architecture de code recommandée

```text
scanner_aws/
├─ apps/
│  ├─ api/
│  │  └─ main.py
├─ docs/
│  └─ PROJECT_ROADMAP.md
├─ src/
│  ├─ core/
│  ├─ auth/
│  ├─ discovery/
│  ├─ normalize/
│  ├─ rules/
│  ├─ findings/
│  ├─ remediation/
│  ├─ costs/
│  ├─ reports/
│  └─ storage/
├─ tests/
└─ pyproject.toml
```

---

## 5. Règles d'extensibilité

### 5.1 Ajouter un nouveau service AWS
Pour ajouter un nouveau service :
1. créer un collecteur dans `src/discovery/services/`,
2. créer un normaliseur si nécessaire,
3. créer les règles associées,
4. connecter le tout au registre.

### 5.2 Ajouter une nouvelle règle
Pour ajouter une règle :
1. créer un fichier dans la bonne catégorie,
2. hériter d'une base commune,
3. déclarer metadata + logique,
4. enregistrer la règle dans le registry.

### 5.3 Ajouter une nouvelle remédiation
Pour ajouter une remédiation :
1. créer une classe ou un template,
2. l'associer au type de finding,
3. ajouter niveau de risque / prérequis / rollback si applicable.

---

## 6. Standards de code

### Python
- type hints obligatoires autant que possible,
- dataclasses ou Pydantic pour les modèles,
- logique métier séparée des routes API,
- pas de logique AWS directement dans les endpoints.

### Nommage
- noms explicites,
- modules courts,
- règles nommées avec convention stable :
  - `security.s3.public_bucket`
  - `cost.ebs.detached_volume`

### Tests
- tests unitaires pour les règles,
- fixtures pour les réponses AWS simulées,
- pas de dépendance AWS réelle dans les tests unitaires.

---

## 7. Découpage des prochains fichiers

Les prochains fichiers recommandés sont :

1. `pyproject.toml`
2. `apps/api/main.py`
3. `src/core/config.py`
4. `src/core/logging.py`
5. `src/core/exceptions.py`
6. `src/auth/session_factory.py`
7. `src/auth/assume_role.py`
8. `src/discovery/regions.py`
9. `src/findings/models.py`
10. `src/rules/base.py`

---

## 8. Sprint 1 recommandé

### Objectif
Avoir une application qui démarre localement et qui expose une API minimale.

### Livrables
- structure des dossiers,
- configuration Python,
- FastAPI fonctionnel,
- route `/health`,
- base de configuration centralisée.

---

## 9. Sprint 2 recommandé

### Objectif
Pouvoir valider l'accès AWS et lister les régions.

### Livrables
- session boto3,
- AssumeRole,
- endpoint test de connexion,
- listing des régions.

---

## 10. Sprint 3 recommandé

### Objectif
Scanner les premières ressources et générer les premiers findings.

### Livrables
- collecteur EC2/EBS,
- règle volume détaché,
- règle security group SSH ouvert,
- sortie findings JSON.

---

## 11. Décision importante sur la remédiation

Pour le MVP :
- **pas d'exécution destructive automatique**,
- seulement :
  - recommandations,
  - snippets boto3,
  - propositions Terraform.

Les actions automatiques viendront après validation, audit trail et dry-run.

---

## 12. Travail dans VS Code

Dans VS Code, on travaillera idéalement ainsi :
1. créer les fichiers de base,
2. faire tourner l'API localement,
3. vérifier chaque étape avant de continuer,
4. ajouter service par service,
5. ajouter règle par règle,
6. documenter tout changement structurant.

---

## 13. Objectif de qualité

Le projet doit rester :
- simple à comprendre,
- simple à maintenir,
- simple à étendre,
- et crédible techniquement face à un client final.

C'est plus important qu'un grand nombre de features au début.
