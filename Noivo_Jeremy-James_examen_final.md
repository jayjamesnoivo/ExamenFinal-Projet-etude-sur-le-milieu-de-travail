# Partie 1 — Commandes Docker Compose et lecture du projet

## 1.1 Commandes de base avec Docker Compose

a) Construire et lancer l’environnement de développement
# Ma commande : docker compose -f docker-compose.dev.yml up --build

b) Afficher les services en cours d’exécution pour l’environnement de développement
# Ma commande : docker compose -f docker-compose.dev.yml ps

c) Afficher les logs du service api en temps réel
# Ma commande : docker compose -f docker-compose.dev.yml logs -f api

d) Arrêter tous les services de l’environnement de développement
# Ma commande : docker compose -f docker-compose.dev.yml down

e) Construire et lancer l’environnement de production en arrière-plan
# Ma commande : docker compose -f docker-compose.prod.yml up -d --build

## 1.2 Vérifier les services déclarés

Afficher la liste des services déclarés dans docker-compose.prod.yml
# Ma commande : docker compose -f docker-compose.prod.yml config --services

## 1.3 Communication entre les services

Pourquoi l’API utilise db au lieu de localhost ?
réponse : Dans Docker Compose, chaque service communique sur le réseau interne Docker en utilisant le nom du service comme nom d’hôte. Le service MongoDB s’appelle "db", donc l’API peut y accéder avec mongodb://db. Si on utilisait localhost, cela pointerait vers le conteneur de l’API lui-même et non vers le conteneur MongoDB.

# Partie 2 — Dockerfile et environnement de développement

## 2.1 Compléter le Dockerfile de production de l'API

FROM node:lts-alpine

WORKDIR /app

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 80

CMD ["npm", "run", "prod"]

## 2.2 Compléter le script de production dans package.json

{
  "scripts": {
    "start": "node src/index.js --watch",
    "prod": "node src/index.js"
  }
}

## 2.3 Compléter le fichier docker-compose.dev.yml

services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.dev
    volumes:
      - type: bind
        source: ./client
        target: /home/node
      - type: volume
        target: /home/node/node_modules
    ports:
      - "5173:5173"

  api:
    build:
      context: ./api
      dockerfile: Dockerfile.dev
    volumes:
      - type: bind
        source: ./api/src
        target: /app/src
    ports:
      - "3001:80"
    depends_on:
      - db

  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbtest
        target: /data/db

  reverse-proxy:
    build:
      context: ./reverseproxy
      dockerfile: Dockerfile.dev
    ports:
      - "80:80"
    depends_on:
      - client
      - api
      - db

volumes:
  dbtest:

# Partie 3 — Environnement de production

## 3.1 Compléter le fichier docker-compose.prod.yml

services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.prod
    restart: unless-stopped

  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
    environment:
      - MONGO_USERNAME
      - MONGO_PWD
      - NODE_ENV=production
    restart: unless-stopped
    depends_on:
      - db

  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbprod
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
    restart: unless-stopped

  reverse-proxy:
    build:
      context: ./reverseproxy
      dockerfile: Dockerfile.prod
    ports:
      - "80:80"
    restart: unless-stopped
    depends_on:
      - client
      - api
      - db

volumes:
  dbprod:
    external: true

## 3.2 Créer le volume externe de production

# Ma commande : docker volume create dbprod

## 3.3 Lancer l'application en production avec les variables d'environnement

MONGO_USERNAME=app_user \
MONGO_PWD=api_password \
MONGO_INITDB_ROOT_USERNAME=root \
MONGO_INITDB_ROOT_PASSWORD=admin_password \
docker compose -f docker-compose.prod.yml up -d --build

# Partie 4 — MongoDB, variables d'environnement et fichiers ignorés

## 4.1 Démarrer uniquement MongoDB pour l’initialisation

# Ma commande :
MONGO_INITDB_ROOT_USERNAME=root MONGO_INITDB_ROOT_PASSWORD=admin_password docker compose -f docker-compose.prod.yml up -d db

## 4.2 Entrer dans la console MongoDB

# Ma commande :
docker compose -f docker-compose.prod.yml exec db mongosh

## 4.3 Authentification dans MongoDB

use admin;

db.auth({
  user: "root",
  pwd: "admin_password",
});

## 4.4 Créer l’utilisateur utilisé par l’API

// Votre commande :
db.getSiblingDB("test").createUser({
  user: "app_user",
  pwd: "api_password",
  roles: [
    { role: "readWrite", db: "test" }
  ]
});

## 4.5 Fichiers à ignorer

.gitignore

# Votre réponse :
.env
node_modules/

.dockerignore

# Votre réponse :
.env