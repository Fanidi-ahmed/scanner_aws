# AWS Scan Pro

AWS Scan Pro est un projet de plateforme d'audit AWS orientée **visibilité**, **sécurité**, **optimisation des coûts** et **remédiation guidée**.

L'objectif est de fournir un outil capable de scanner un ou plusieurs comptes AWS, sur plusieurs régions, afin d'identifier :

- les risques sécurité,
- les ressources obsolètes,
- les coûts inutiles,
- les problèmes de gouvernance,
- et les actions correctives possibles en **Terraform** ou **Python boto3**.

---

## 1. Vision du projet

Le projet ne doit pas être seulement un "scanner AWS" technique.

Il doit devenir un outil capable de :

1. **Détecter** les problèmes techniques et financiers.
2. **Expliquer** les impacts de manière compréhensible.
3. **Prioriser** les actions selon le risque, l'effort et l'économie potentielle.
4. **Proposer des remédiations concrètes**.
5. **Servir de base à un produit vendable** ou à une offre de service cloud/FinOps/SecOps.

---

## 2. Objectifs principaux

### Objectifs fonctionnels

- Scanner plusieurs comptes AWS.
- Scanner toutes les régions pertinentes activées sur les comptes ciblés.
- Collecter des informations sur un ensemble priorisé de services AWS.
- Détecter des findings de type :
  - sécurité,
  - coûts,
  - gouvernance,
  - obsolescence,
  - fiabilité.
- Afficher les résultats dans une interface détaillée.
- Générer des recommandations humaines et techniques.
- Fournir des remédiations assistées :
  - snippets boto3,
  - propositions Terraform,
  - plans d'action.

### Objectifs produit

- Créer un MVP rapidement exploitable.
- Structurer le projet pour évoluer vers un SaaS ou un outil de conseil interne.
- Faciliter la vente via des rapports lisibles et des gains financiers explicites.

---

## 3. Proposition de valeur

AWS Scan Pro doit répondre à trois besoins forts :

### 3.1 Visibilité
- Inventaire multi-comptes / multi-régions.
- Détection des ressources oubliées, non taguées ou non conformes.
- Vue consolidée par compte, service, région et criticité.

### 3.2 Réduction des risques
- Détection de configurations dangereuses.
- Explications claires du risque.
- Priorisation selon criticité et confiance.

### 3.3 Réduction des coûts
- Détection de ressources inutilisées ou surdimensionnées.
- Estimation d'économies mensuelles.
- Mise en avant des quick wins.

### 3.4 Remédiation concrète
- Recommandations textuelles.
- Snippets Python boto3.
- Exemples Terraform.
- Mode dry-run et validation humaine à terme.

---

## 4. MVP recommandé

Pour éviter un scope trop large, le MVP doit se concentrer sur des services à fort retour sur investissement.

### Services AWS MVP

- IAM
- S3
- EC2
- EBS
- ELB / ALB / NLB
- RDS
- VPC / Security Groups
- CloudTrail

### Catégories de règles MVP

- **Security**
  - bucket S3 public,
  - security group trop permissif,
  - utilisateur IAM avec clé ancienne,
  - CloudTrail absent ou incomplet,
  - ressource publique non attendue.

- **Cost**
  - volumes EBS détachés,
  - snapshots anciens,
  - Elastic IP non associées,
  - instances EC2 sous-utilisées,
  - load balancers inutilisés.

- **Governance**
  - ressources sans tags,
  - absence de tags owner/environment/cost-center,
  - ressources non conformes à des conventions.

- **Obsolete**
  - versions RDS ou moteurs en fin de vie,
  - AMI anciennes,
  - ressources dormantes.

---

## 5. Architecture cible du projet

Le projet sera structuré pour séparer clairement :

- l'API,
- les workers de scan,
- la logique AWS,
- le moteur de règles,
- la remédiation,
- le reporting,
- l'interface web.

### Arborescence cible

```text
scanner_aws/
├─ README.md
├─ docs/
│  └─ PROJECT_ROADMAP.md
├─ apps/
│  ├─ api/
│  ├─ worker/
│  └─ web/
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
├─ infra/
└─ scripts/
```

---

## 6. Stack technique proposée

### Backend
- **Python 3.12+**
- **FastAPI** pour l'API
- **boto3** pour AWS
- **Pydantic** pour les schémas
- **SQLAlchemy** pour l'accès base de données
- **PostgreSQL** pour stocker les scans, findings et historiques
- **Redis** pour cache et queue si nécessaire
- **Celery**, **RQ** ou **Dramatiq** pour les jobs asynchrones

### Frontend
- **Next.js / React**
- **TypeScript**
- **Tailwind CSS**
- composants de table et graphiques

### Infra / DevOps
- Docker / Docker Compose
- Terraform pour l'infra du projet
- CI/CD GitHub Actions plus tard

---

## 7. Sécurité du projet

Le projet manipule des données potentiellement sensibles. La sécurité doit être pensée dès le départ.

### Principes recommandés

- Utiliser **STS AssumeRole** pour accéder aux comptes clients.
- Éviter les clés IAM longues durées quand c'est possible.
- Limiter les permissions de scan au **read-only** pour le MVP.
- Prévoir un rôle distinct si la remédiation automatique est activée plus tard.
- Utiliser un **External ID** pour les accès cross-account.
- Journaliser les opérations importantes.
- Chiffrer les secrets et les données sensibles.
- Prévoir une isolation propre entre clients / tenants.

---

## 8. Modèle fonctionnel de haut niveau

Le cycle d'un scan peut être pensé ainsi :

1. Un compte AWS est enregistré dans l'application.
2. L'application assume un rôle dédié dans le compte cible.
3. Les régions actives sont découvertes.
4. Les collecteurs interrogent les services AWS ciblés.
5. Les données sont normalisées.
6. Les règles sont exécutées.
7. Des findings sont générés.
8. Un score et des estimations d'économies sont calculés.
9. Les résultats sont affichés dans l'interface et exportables.
10. Des remédiations sont proposées.

---

## 9. Structure logique des modules

### `src/core/`
Contient les éléments transverses :
- configuration,
- logging,
- constantes,
- types communs.

### `src/auth/`
Gère :
- AssumeRole,
- sessions boto3,
- validation des credentials,
- gestion des erreurs d'accès.

### `src/discovery/`
Gère la découverte AWS :
- régions,
- services,
- pagination,
- agrégation des ressources.

### `src/normalize/`
Transforme les réponses AWS en modèles internes cohérents.

### `src/rules/`
Contient le moteur de règles et les règles par catégorie.

### `src/findings/`
Contient les structures de findings, scoring et sérialisation.

### `src/remediation/`
Contient les recommandations et la génération de remédiations.

### `src/costs/`
Contient la logique d'estimation de coûts et économies.

### `src/reports/`
Génère des exports lisibles :
- JSON,
- CSV,
- HTML,
- PDF plus tard.

### `src/storage/`
Gère la base de données et la persistance.

---

## 10. Convention de règles

Chaque règle doit être :

- identifiable de manière unique,
- testable,
- documentée,
- liée à une catégorie,
- liée à une sévérité,
- associée à une preuve,
- accompagnée d'une recommandation.

### Exemple de convention d'identifiants

- `security.s3.public_bucket`
- `security.ec2.open_ssh_to_world`
- `cost.ebs.detached_volume`
- `cost.ec2.idle_instance`
- `governance.tags.missing_owner`
- `obsolete.rds.engine_eol`

### Informations minimales d'un finding

- identifiant de règle,
- compte AWS,
- région,
- service,
- ressource concernée,
- sévérité,
- résumé,
- impact business,
- preuve technique,
- action recommandée,
- estimation d'économie éventuelle,
- niveau de confiance.

---

## 11. Roadmap fonctionnelle

### Phase 1 — Fondation
- créer la structure Python du projet,
- définir la configuration,
- mettre en place l'API,
- mettre en place les modèles de données,
- configurer le tooling local.

### Phase 2 — Auth AWS et discovery
- implémenter AssumeRole,
- lister les régions,
- créer les premiers collecteurs AWS,
- normaliser les données brutes.

### Phase 3 — Rules engine
- créer la base du moteur de règles,
- implémenter les premières règles sécurité/coût,
- générer des findings structurés.

### Phase 4 — Persistance et interface
- stocker comptes, scans et findings,
- exposer une API de lecture,
- créer une première interface web.

### Phase 5 — Reporting et remédiation
- rapports synthétiques,
- détails techniques,
- snippets boto3,
- templates Terraform.

### Phase 6 — Industrialisation
- scheduler,
- multi-comptes,
- historique,
- exports avancés,
- notifications,
- CI/CD.

---

## 12. Premières étapes recommandées dans VS Code

Quand on commencera à coder dans VS Code, l'ordre recommandé sera :

1. Initialiser la base backend Python.
2. Créer l'arborescence principale `src/`.
3. Ajouter les fichiers de configuration du projet.
4. Créer un premier `main.py` FastAPI.
5. Définir les modèles de base :
   - Account,
   - Scan,
   - Finding,
   - Resource.
6. Implémenter l'auth AWS.
7. Implémenter la découverte des régions.
8. Implémenter un premier collecteur simple.
9. Implémenter une première règle simple.
10. Afficher les résultats via API.

---

## 13. Méthode de travail recommandée

Pour garder le projet maîtrisable :

- avancer par lots petits et testables,
- définir un MVP strict,
- implémenter peu de services mais bien,
- écrire les règles comme des modules indépendants,
- documenter au fur et à mesure,
- éviter l'auto-remédiation destructive au début.

---

## 14. Ce que nous allons faire ensuite

Le fichier `docs/PROJECT_ROADMAP.md` servira de guide détaillé de construction.

Il décrira :
- les étapes de création du projet,
- l'ordre conseillé des fichiers,
- les premiers sprints,
- les décisions d'architecture,
- et la manière de progresser proprement dans VS Code.

---

## 15. Statut actuel

Ce dépôt est au début de sa phase de cadrage.

Prochain objectif recommandé :
- poser la structure de projet,
- définir les modules de base,
- puis commencer le backend Python du MVP.
