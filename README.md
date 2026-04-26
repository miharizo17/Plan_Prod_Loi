[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/UR3vHHhL)
# Projet de génération de jeu de données
- *ANDRIANIONY Miharizo Kanto*
- *RAKOTONDRAMAKA Asandratra Mitia Ny Aina*

# Phase 1 - Modelisation
>[Diagramme de modelisation Phase 1 (DatasetGenerationProject.drawio.png)](DatasetGenerationProject.drawio.png)
## 1. Description d'architecture

```
ProjectManager
    └── Project
            └── Entity (composite récursif)
                    └── Attribute
                            ├── DataType (StringDataType | IntegerDataType | FloatDataType | BooleanDataType | DateDataType | EnumDataType)
                            ├── Constraint     (LengthConstraint | RangeConstraint | StatConstraint)
                            └── DataGenerator  (RandomTextGenerator | RandomNumberGenerator | ApiDataGenerator)
                                                        └── ApiCaller  (FetchNameApi | FetchNumberApi)
ProjectManager
    └── Exporter   (CSVExporter | JSONExporter | XMLExporter | SQLExporter)
```

- *ProjectManager* : Gestion de la liste des projets et exportation des datasets
- *Project* : Liste des Entity, exporte la fonction de generations de liste d'Entity.
- *Entity* : Individu portant une liste d'attributs
- *Attribute* : Specification des valeurs, avec leurs noms, type, et contrainte, avec la fonction de leur générations
- *DataType* : Interface d'exposition des types gerer dans le projets, implémentée par "StringDataType", "FloatDataType", "IntegerDataType", "BooleanDataType", "DateDataType", "EnumDataType" avec la méthode getDataType(): String
- *Constraint*: Interface pour la validation des contraintes des valeurs qui expose verify(Object, Class): boolean avec ces 3 implémentations dont il y a "LengthConstraint", "RangeConstraint" et "StatConstraint".
- *DataGenerator*: Interface pour la génération des valeurs avec la méthode generateData(List<Constraint>): Object dont "RandomTextGenerator", "RandomNumberGenerator" et ApiDataGenerator l'implémente à préciser que ApiDataGenerator délègue la génération à une API externe via un ApiCaller.
- *Exporter*: Interface pour exporter les valeurs en différent formes avec la méthode exportDataset(Project): String ,avec ces 4 formats "CSVExporter", "JSONExporter", "XMLExporter", "SQLExporter".

## 2. Justification des choix de conception

- *Pattern Stratégie:* qui est appliquée à DataGenerator, Exporter, ApiCaller, Constraint, DataType pour éviter les blocs if/else afin garder un code plus propre et évolutif.
- *Pattern Composite:* qui est appliquée à Entity via subentities: List<Entity> pour la modélisation naturelle de données hiérarchiques.
- *Pattern Façade:*  qui est appliquée ProjectManager vu que ProjectManager expose une interface simplifiée qui sont addProject, removeProject et  export, qui masque la complexité interne: gestion de la liste de projets, sélection de l'exporteur, orchestration de la génération.
- *Délégation de la génération à l'Attribute:* parce que l'Attribute connaît ses propres contraintes : il est le seul à pouvoir passer List<Constraint> au générateur.
- *Typage via interface DataType plutôt qu'un enum:* parce qu'une interface DataType est ouverte à l'extension : on crée simplement DateDataType implements DataType sans rien toucher d'autre.

## 3. Les principes SOLID respectés
- *Single Responsibility Principle*

| Classe | Responsabilité unique |
|---|---|
| `Attribute` | Modéliser un attribut et générer sa valeur |
| `LengthConstraint` | Vérifier une contrainte de longueur |
| `CSVExporter` | COnvertion d'un projet en CSV |
| `FetchNameApi` | Appeler une API externe |
| `ProjectManager` | Orchestrer les projets et les exports |

- *Open/Closed Principle*

Toutes les interfaces DataGenerator, Exporter, Constraint, DataType, ApiCaller permettent d'ajouter de nouvelles implémentations sans modifier le code existant par exemple si on veut créer une nouvelle contrainte il suffit juste de créer RegexConstraint implements Constraint

- *Liskov Substitution Principle*

Toute implémentation d'une interface peut être substituée à une autre. Prenons un exemple, CSVExporter peut être remplacé par JSONExporter sans que ProjectManager ne le remarque.

- *Interface Segregation Principle*

Les interfaces sont petites et focalisées :


| Interface | Méthodes exposées |
|---|---|
| `DataType` | `getDataType()` |
| `Constraint` | `verify(Object, Class)` |
| `DataGenerator` | `generateData(List<Constraint>)` |
| `ApiCaller` | `callApi(String)` |
| `Exporter` | `exportDataset(Project)` |

- *Dependency Inversion Principle*

Aucune classe de haut niveau ne référence une implémentation concrète directement. Les dépendances sont injectées via les interfaces, ce qui rend le système entièrement testable et configurable.

| Module haut niveau | Dépend de (abstraction) | Implémentations concrètes |
|---|---|---|
| `ProjectManager` | `Exporter` | CSV, JSON, XML, SQL |
| `Attribute` | `DataGenerator` | Random, Api |
| `ApiDataGenerator` | `ApiCaller` | FetchName, FetchNumber |
| `Attribute` | `Constraint` | Length, Range, Stat |
| `Attribute` | `DataType` | String, Float |
---
# Phase 2 - Implémentation
>[Diagramme de l'architecture suivi en implementation (dataset.drawio.png)](dataset.drawio.png)
## 1. Paramètres du projet
- **Version Spring Boot** : 4.0.5
- **Version Java** : 25
- **Type** : Maven
- **Configuration** : Properties

## 2. Architecture suivie en développement
```
├── Controllers
│   ├── DataModelingController
│   ├── ProjectController
│   └── GeneratorController

├── Services
│   ├── ProjectService
│   ├── EntityService
│   ├── AttributeService
│   ├── DataGeneratorService
│   └── ExportService

├── Repositories
│   ├── ProjectRepository
│   ├── EntityRepository
│   └── AttributeRepository

├── Mappers
│   ├── ProjectMapper
│   ├── EntityMapper
│   └── AttributeMapper

├── Model / Domain
│   ├── ProjectEntity
│   ├── Entity -> EntityComponent
│   ├── Attribute -> EntityComponent
│   └── EntityComponent <<Abstract>>

├── DTOs
│   ├── Project
│   │   ├── ProjectCreateDTO
│   │   ├── ProjectUpdateDTO
│   │   └── ProjectResponseDTO
│   ├── Entity / Attribute
│   │   ├── CreationRequestDto
│   │   ├── UpdateRequestDto
│   │   ├── EntityResponseDto
│   │   ├── AttributeResponseDto
│   │   └── ComponentResponseDto
│   └── Others
│       ├── ExportSchema
│       └── GeneratorResult

├── Data Generation Layer
│   ├── DataType [BOOLEAN, DATE, FLOAT, INTEGER, STRING, ENUM]
│   ├── DataGenerator
│   │   ├── StringGenerator
│   │   ├── IntegerGenerator
│   │   ├── FloatGenerator
│   │   ├── BooleanGenerator
│   │   ├── DateGenerator
│   │   └── EnumGenerator
│   ├── CharacterSet
│   │   ├── LowercaseCharacterSet
│   │   ├── UppercaseCharacterSet
│   │   ├── NumberCharacterSet
│   │   └── SpecialCharacterSet
│   └── DataGeneratorFactory

├── Export
│   ├── Exporter
│   │   ├── CSVExporter
│   │   ├── JSONExporter
│   │   ├── XMLExporter
│   │   └── SQLExporter
│   └── ExportService

├── Utils
│   ├── DataGeneratorJsonConverter
│   └── JsonMapConverter

├── Exceptions
│   ├── GlobalExceptionHandler
│   ├── RequestValidationException
│   └── MessageException

└── Main
    └── DatasetApplication
```
## 3. Note de modifications (Évolution depuis la Phase 1)

- **Mise en place de l'architecture Spring Boot**
  - **Controllers** (DataModelingController, ProjectController, GeneratorController) pour exposer les endpoints REST.
  - **Services** (interfaces + implémentations) et Repositories pour préparer la persistance (JPA).
  - Introduction de plusieurs Mappers pour la conversion entre entités de domaine et DTOs.

- **DTOs**
  - **DTOs** : CreationRequestDto, UpdateRequestDto, ResponseDto et ComponentResponseDto.
  - Utilisation de ComponentResponseDto pour représenter uniformément Entity et Attribute dans les réponses API.

- **Système de génération de données**
  - **DataType** : Simplifié en enum.
  - **DataGenerator** : Refondu en générateurs spécialisés par type de données.
  - **CharacterSet** : les caractères autorisés dans la génération de textes.
  - **DataGeneratorFactory** pour instancier dynamiquement les générateurs.
  - Les contraintes de génération sont intégrées directement comme attributs dans chaque générateur (min, max, length, characterSets, etc.).

- **Gestion des sous-entités et attributs**
  - Interface EntityComponent implémentée par Entity et Attribute.
  - Relation parent / subentities clairement modélisée.

- **Ajouts par rapport à la Phase 1**
  - Gestion centralisée des exceptions (GlobalExceptionHandler, RequestValidationException, etc.).
  - GeneratorResult et ExportSchema pour structurer les résultats de génération et d’export.

- **Éléments modifiés de la Phase 1**
  - Suppression du système de Constraint indépendant (les contraintes sont maintenant intégrées dans les générateurs).
---

# Phase 3: Tests d’API & Validation Automatisée

Collection de tests pour l'API REST de génération de datasets.
Les tests sont écrits pour **Postman (GUI)** et exécutables en ligne de commande via **Newman (CLI)**.

---

## Structure des fichiers

```
collection_postman/
├── Dataset_Kanto_Asandratra.postman_collection.json   # Collection Postman
├── Dataset_Kanto_Asandratra.postman_environment.json  # Variables d'environnement 
├──newman/                               # Rapports HTML générés automatiquement
```

---

## Organisation de la collection

| Dossier | Type | Description |
|---------|------|-------------|
| **Tests Unitaires** | Unit tests | Chaque endpoint testé isolément avec des données valides |
| **Gestion des Erreurs** | Edge cases | Vérifie que l'API retourne 400/404 et ne crashe jamais en 500  |
| **Scénario Nominal** | Integration test | Happy path complet, bout en bout, idempotent grâce au DELETE final  |

---

## Prérequis

- **Node.js** ≥ 18 installé ([nodejs.org](https://nodejs.org))
- L'application Spring Boot démarrée sur `http://localhost:8080`

---

## Installation de Newman sur windows

```bash
npm install -g newman newman-reporter-htmlextra
```

- `newman` : exécuteur CLI de collections Postman
- `newman-reporter-htmlextra` : génère un rapport HTML détaillé

Vérifier l'installation :

```bash
newman --version
```

---

## Lancer les tests

### Commande
Lance la ligne de commande depuis le répertoire où se trouvent les collections Postman.

```bash
>newman run Dataset_Kanto_Asandratra.postman_collection.json -e Dataset_Kanto_Asandratra.postman_environment.json
```

### Commande complète avec rapport HTML

```bash
newman run Dataset_Kanto_Asandratra.postman_collection.json -e Dataset_Kanto_Asandratra.postman_environment.json -r html
```


Le rapport HTML est généré dans `newman/newman-run-report-2026-04-12-12-27-23-468-0.html`.

---

## Variables d'environnement

| Variable | Description | Gestion |
|----------|-------------|---------|
| `base_url` | URL de base de l'API | Valeur fixe : `http://localhost:8080` |
| `unit_project_id` | ID projet créé dans test unitaire |Utilisé pour les tests unitaires de la valeur de id_project. |
| `unit_entity_id` | ID entité créée dans test unitaire |Utilisé pour les tests unitaires de la valeur de id_entity |
| `current_project_id` | ID projet du scénario nominal | Utilisé pour les scenario de la valeur de id_project.” |
| `current_entity_id` | ID entité du scénario nominal | Utilisé pour le scenario nominal de la valeur de id_entity |
| `deleted_project_id` | ID projet supprimé en scenario nominal | Utilisé pour vérifier le 404 |

---

## Idempotence

Le scénario nominal est **rejouable à l'infini** : l'étape DELETE supprimer projet supprime le projet créé et nettoie toutes les variables. Relancer Newman deux fois de suite produit le même résultat.

---

## Installation de Postman sur windows
1. Va sur le site officiel : `https://www.postman.com/downloads/`
2. Clique sur Download for Windows
3. Lance le fichier .exe téléchargé
4. Suis l’installation (Next → Install → Finish)
5. Ouvre Postman et connectez-vous (ou utilise-le sans compte)

---

## Importer dans Postman GUI

1. Ouvrir Postman
2. **Import** → sélectionner `Dataset_Kanto_Asandratra.postman_collection.json`
3. **Import** → sélectionner `Dataset_Kanto_Asandratra.postman_environment.json`
4. Sélectionner l'environnement **"Dataset Generator - Local"** en haut à droite
5. Lancer la collection avec **Run collection**

---

# Phase 4 : Architecture Microservices

Migration de l'application monolithique vers une architecture microservices Spring Cloud, avec Dockerisation complète de l'écosystème.

---

## 1. Vue d'ensemble de l'architecture

```
                        ┌─────────────┐
                        │   Gateway   │ :8888
                        │ (API Entry) │
                        └──────┬──────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
     ┌────────▼───────┐        │       ┌────────▼────────┐
     │  Modelisation  │ :8081  │       │Generator Service│ :8082
     │  (CRUD + JPA)  │        │       │ (Data Gen + CSV)│
     └────────────────┘        │       └─────────────────┘
              │                │                │
              └────────────────┼────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Discovery (Eureka) │ :8761
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Config Server     │ :9999
                    │   (config-repo/)    │
                    └─────────────────────┘
```

| Service | Rôle | Port | Build |
|---|---|---|---|
| **config** | Centralise les configurations des services | 9999 | Gradle |
| **discovery** | Annuaire Eureka — les services s'y enregistrent | 8761 | Gradle |
| **modelisation** | CRUD Projets / Entités / Attributs + BDD H2 | 8081 | Maven |
| **generator-service** | Génération de données + export CSV/JSON/XML | 8082 | Gradle |
| **gatewayService** | Point d'entrée unique, routage vers les services | 8888 | Gradle |

---

## 2. Versions des dépendances

| Technologie | Version |
|---|---|
| Java | 25 (Amazon Corretto) |
| Spring Boot | 4.0.5 |
| Spring Cloud | 2025.1.1 |
| Spring Cloud Gateway (WebFlux) | 2025.1.1 |
| Spring Cloud Netflix Eureka | 2025.1.1 |
| Spring Cloud Config Server | 2025.1.1 |
| Spring Cloud OpenFeign | 2025.1.1 |
| Resilience4j (Circuit Breaker) | 2025.1.1 |
| Gradle | 9.4.1 |
| Maven | 3.x (via mvnw wrapper) |
| H2 Database | Runtime |
| Lombok | Latest |
| SpringDoc OpenAPI (Swagger) | 3.0.2 |
| Docker | 20+ |
| Docker Compose | 2.x |

---

## 3. Structure du projet microservices

```
microservices/
├── docker-compose.yml            # Orchestration de tous les services
├── config-repo/                  # Fichiers de configuration centralisés
│   ├── modelisation.properties
│   └── generator-service.properties
├── config/                       # Spring Cloud Config Server
├── discovery/                    # Eureka Server
├── modelisation/                 # Service de modélisation (Maven)
├── generator-service/            # Service de génération (Gradle)
├── gatewayService/               # API Gateway (Gradle)
└── tests/                        # Collection Newman/Postman
    ├── collection.json
    └── environment.json
```

---

## 4. Prérequis

- **Docker Desktop** ≥ 20 — [docker.com/get-docker](https://www.docker.com/get-docker/)
- **Docker Compose** v2 (inclus avec Docker Desktop)
- **Java 25** (pour lancer les services sans Docker)
- **Node.js** ≥ 18 (pour les tests Newman uniquement)

Vérifier les installations :
```bash
docker --version
docker compose version
java --version
```

---

## 5. Lancer l'écosystème complet avec Docker

### Première fois (build + démarrage)
```bash
cd microservices
docker-compose up --build
```

### Fois suivantes (sans rebuild)
```bash
docker-compose up
```

### Arrêter tous les services
```bash
docker-compose down
```

### Vérifier que tous les services sont démarrés
```bash
docker ps
```

Les services démarrent dans cet ordre automatiquement :
1. **config** (port 9999) — attend son healthcheck
2. **discovery** (port 8761) — attend son healthcheck
3. **modelisation**, **generator-service**, **gateway** — démarrent ensemble après discovery

> Le premier démarrage prend plusieurs minutes (téléchargement des images Amazon Corretto + compilation des services).

---

## 6. Lancer les services individuellement (sans Docker)

Si vous souhaitez lancer un service seul pour le développement :

### Config Server (Gradle)
```bash
cd microservices/config
./gradlew bootRun
# Accessible sur http://localhost:9999
```

### Discovery / Eureka (Gradle)
```bash
cd microservices/discovery
./gradlew bootRun
# Accessible sur http://localhost:8761
```

### Modelisation (Maven)
```bash
cd microservices/modelisation
./mvnw spring-boot:run
# Accessible sur http://localhost:8081
```

### Generator Service (Gradle)
```bash
cd microservices/generator-service
./gradlew bootRun
# Accessible sur http://localhost:8082
```

### Gateway (Gradle)
```bash
cd microservices/gatewayService
./gradlew bootRun
# Accessible sur http://localhost:8888
```

---

## 7. URLs utiles une fois démarrés

| URL | Description |
|---|---|
| `http://localhost:8761` | Dashboard Eureka (services enregistrés) |
| `http://localhost:9999/modelisation/default` | Config du service modelisation |
| `http://localhost:8081/api/projects` | API Modelisation |
| `http://localhost:8082/api/generator/{id}/preview` | Prévisualisation des données |
| `http://localhost:8081/h2-console` | Console H2 (BDD en mémoire) |
| `http://localhost:8888` | API Gateway |

---

## 8. Communication entre services

- **Config Server** → lit les fichiers `.properties` depuis `config-repo/` (mode `native`)
- **Modelisation** → s'enregistre auprès d'Eureka, expose un endpoint interne `/api/internal/projects/{id}/definition`
- **Generator Service** → appelle Modelisation via **Feign Client** avec **Circuit Breaker Resilience4j** (fallback : liste vide si Modelisation est indisponible)
- **Gateway** → route automatiquement vers les services via la découverte Eureka

---

## 9. Tests automatisés (Phase 3)

La collection de tests couvre les microservices `modelisation` (port 8081) et `generator-service` (port 8082).

### Installation de Newman
```bash
npm install -g newman newman-reporter-htmlextra
```

### Lancer les tests
```bash
cd microservices/tests
newman run collection.json -e environment.json
```

### Lancer avec rapport HTML
```bash
newman run collection.json -e environment.json -r htmlextra
```

### Résultats attendus
```
assertions : 115 exécutées / 0 échouées
```

### Organisation de la collection

| Dossier | Type | Description |
|---|---|---|
| **A – Tests Unitaires** | Unit tests | Chaque endpoint testé isolément |
| **B – Gestion des Erreurs** | Edge cases | 400/404 explicites, jamais de 500 |
| **C – Scénario Nominal** | Integration test | Happy path complet bout en bout, idempotent |

---

## 10. Dockerisation — détail de l'intégration

Chaque service possède son propre `Dockerfile` multi-stage :
- **Stage 1 (builder)** : compile le projet (Gradle `bootJar` ou Maven `package`)
- **Stage 2** : image finale légère avec uniquement le JAR

Le `docker-compose.yml` à la racine de `microservices/` orchestre l'ensemble :
- **Health checks** sur config et discovery pour garantir l'ordre de démarrage
- **Réseau dédié** `microservices-net` pour la communication inter-services
- **Volume** monté sur `config-repo/` pour que le Config Server lise les fichiers de configuration locaux
- **Variables d'environnement** injectées pour pointer vers les bons services (Eureka URL, Modelisation URL)
