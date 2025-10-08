# 🛵 Projet Simulation UberEats : Comparaison Redis vs MongoDB

Ce projet simule la logique de gestion des courses (**dispatch**) d'une plateforme de livraison type **UberEats**, à travers deux architectures de bases de données distribuées :  
- **Redis (Pub/Sub)**  
- **MongoDB (Change Streams)**  

---

## 🚀 1. Architecture du Projet

Le cœur du projet repose sur le concept **Producteur / Consommateur** et la **compétition en temps réel** entre plusieurs Livreurs.

### 1.1 Composants Communs

| Composant | Rôle | Description |
|------------|------|-------------|
| **Manager** | Producteur | Simule la plateforme : publie les annonces et sélectionne le livreur le plus rapide. |
| **Livreur** | Consommateur / Compétiteur | Simule une application livreur : reçoit les annonces et tente d'accepter une course. |
| **`denormalisation.json`** | Source de données | Contient les données de commandes (restaurants, clients, etc.) utilisées pour simuler les annonces. |

---

### 1.2 Architecture Redis — Messagerie Événementielle

Cette architecture utilise **Redis** comme un **courtier de messages** et un **stockage temporaire**.

- **Communication :** via le modèle **Pub/Sub** (diffusion instantanée des annonces).
- **Sélection :** le Manager attend 5 secondes pour collecter les intérêts via `HSET`, puis choisit le **timestamp le plus précoce**.
- **État :** stocké dans les clés :
  - `ubereats:course:active` → course en cours
  - `ubereats:interet:<order_id>` → intérêts reçus

---

### 1.3 Architecture MongoDB — Flux de Changement Persistant

Cette architecture repose sur un **replica set MongoDB** et exploite les **Change Streams** pour détecter les insertions en temps réel.

- **Communication :** le Manager insère un document dans `annonces`. Les Livreurs écoutent les changements.
- **Sélection :** mise à jour atomique conditionnelle (`$set` sur `status`).
  - Le premier Livreur à changer `ANNONCE_PUBLIEE → LIVREUR_SELECTIONNE` gagne.
- **État :** chaque annonce est un document complet et persistant dans MongoDB.

---

## ⚙️ 2. Prérequis Techniques

Avant de lancer le projet, assurez-vous d’avoir :

- **Python 3.x**
- **Redis Server** ≥ 5.0
- **MongoDB Server** ≥ 4.0 (

---

## 3. Structure des Dossiers
```bash
projet_uber
├── avec-mongo/
│   ├── denormalisation.json     # Données sources
│   ├── manager_mongo.py         # Manager utilisant MongoDB
│   └── livreur_mongo.py         # Livreur utilisant MongoDB
├── avec-redis/
│   ├── denormalisation.json     # Données sources
│   ├── manager_ubereats.py      # Manager utilisant Redis
│   └── livreur_ubereats.py      # Livreur utilisant Redis
└── denormalisation.py           # Script initial de préparation des données


MONGODB : 

###  PREMIER TERMINAL : 

```bash
mongod --port 27017 --dbpath /home/narz/BUT3/Base_De_Donnees/projet/projet-uber/avec-mongo/data/db --replSet rs0

### DEUXIEME TERMINAL : 
```bash
cd BUT3/Base_De_Donnees/projet/projet-uber/avec-mongo
source mymongo/bin/activate
python livreur_mongo.py

### TROISIEME TERMINAL : 
```bash 
cd BUT3/Base_De_Donnees/projet/projet-uber/avec-mongo
source mymongo/bin/activate
python manager_mongo.py

###  TERMINAL : 
```bash
mongosh
use uber_mongo
db.annonces.find().pretty()


### REDIS :

### SERVEUR TERMINAL : 

```bash
cd redis/redis-stable/src
./redis-server

### CLIENT TERMINAL : 

```bash
cd ~/redis/redis-stable/src
./redis-cli

--- Apres ---

```bash 
GET ubereats:course:active

### PREMIER TERMINAL :

```bash
cd BUT3/Base_De_Donnees/projet/projet-uber/avec-redis
source myredis/bin/activate 
python livreur_redis.py

### DEUXIEME TERMINAL : 

```bash
cd BUT3/Base_De_Donnees/projet/projet-uber/avec-redis
source myredis/bin/activate 
python manager_redis.py



