# Partie 1 — Commandes Docker Compose et lecture du projet

**20 points**

## 1.1 Commandes de base avec Docker Compose

**10 points**

Écrivez les commandes permettant d’effectuer les actions suivantes.

Vous pouvez utiliser `docker compose` ou `docker-compose`.

### a) Construire et lancer l’environnement de développement

```bash
# Votre commande : docker compose -f docker-compose.dev.yml up --build
```

---

### b) Afficher les services en cours d’exécution pour l’environnement de développement

```bash
# Votre commande : docker compose -f docker-compose.dev.yml ps
```

---

### c) Afficher les logs du service `api` en temps réel

```bash
# Votre commande : docker compose -f docker-compose.dev.yml logs -f api
```

---

### d) Arrêter tous les services de l’environnement de développement

```bash
# Votre commande : docker compose -f docker-compose.dev.yml down
```

---

### e) Construire et lancer l’environnement de production en arrière-plan

```bash
# Votre commande : docker compose -f docker-compose.prod.yml up -d --build
```

---

## 1.2 Vérifier les services déclarés

**4 points**

Écrivez la commande permettant d’afficher la liste des services déclarés dans le fichier :

```txt
docker-compose.prod.yml
```

```bash
# Votre commande : docker compose -f docker-compose.prod.yml config --services
```

---

## 1.3 Communication entre les services

**6 points**

Dans le code de l’API, la connexion à MongoDB utilise le nom `db` :

```js
mongodb://db
```

Expliquez pourquoi l’API doit utiliser `db` au lieu de `localhost` lorsqu’elle fonctionne dans Docker Compose.

Votre réponse :

```
Dans Docker Compose, chaque service communique sur le réseau interne Docker en utilisant le nom du service comme nom d’hôte. Le service MongoDB s’appelle "db", donc l’API peut y accéder avec mongodb://db. Si on utilisait localhost, cela pointerait vers le conteneur de l’API lui-même et non vers le conteneur MongoDB.
```

---

# Partie 2 — Dockerfile et environnement de développement

**30 points**

## 2.1 Compléter le Dockerfile de production de l’API

**10 points**

On vous donne le fichier `api/Dockerfile.prod` incomplet suivant.

Complétez les éléments manquants.

```dockerfile
FROM node:lts-alpine

WORKDIR /app

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 80

CMD ["npm", "run", "prod"]
```

Contraintes :

- utiliser une image Node.js LTS légère ;
- placer l’application dans `/app` ;
- installer les dépendances ;
- exposer le port `80` ;
- lancer l’application avec le script de production défini dans `package.json`.

---

## 2.2 Compléter le script de production dans `package.json`

**5 points**

On vous donne l’extrait suivant du fichier `api/package.json`.

Complétez la section `scripts`.

```json
{
  "scripts": {
    "start": "node src/index.js --watch",
    "prod": "node src/index.js"
  }
}
```

Le script `prod` doit lancer l’application Node.js sans mode `watch`.

---

## 2.3 Compléter le fichier `docker-compose.dev.yml`

**15 points**

On vous donne le fichier `docker-compose.dev.yml` incomplet suivant.

Complétez les éléments manquants.

```yaml
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
```

Contraintes :

- le client doit être accessible en développement sur le port `5173` ;
- l’API doit être accessible directement en développement sur le port `3001` de la machine hôte ;
- le port interne de l’API est `80` ;
- le code du client doit être synchronisé avec le conteneur ;
- le code source de l’API doit être synchronisé avec le conteneur ;
- les données MongoDB doivent être stockées dans un volume nommé `dbtest` ;
- le dossier `node_modules` du client doit être protégé par un volume.

---

# Partie 3 — Environnement de production

**30 points**

## 3.1 Compléter le fichier `docker-compose.prod.yml`

**18 points**

On vous donne le fichier `docker-compose.prod.yml` incomplet suivant.

Complétez les éléments manquants.

```yaml
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
```

Contraintes :

- les services doivent utiliser les Dockerfile de production ;
- l’API doit fonctionner avec `NODE_ENV=production` ;
- l’API dépend de la base de données ;
- MongoDB doit stocker ses données dans un volume nommé `dbprod` ;
- le volume `dbprod` doit être externe ;
- les services doivent redémarrer automatiquement sauf arrêt manuel ;
- les mots de passe ne doivent pas être écrits directement dans ce fichier.

---

## 3.2 Créer le volume externe de production

**4 points**

Le fichier de production utilise un volume externe nommé `dbprod`.

Écrivez la commande permettant de créer ce volume avant de lancer l’application.

```bash
# Votre commande : docker volume create dbprod
```

---

## 3.3 Lancer l’application en production avec les variables d’environnement

**8 points**

Complétez la commande suivante afin de lancer l’application en production.

Les variables attendues sont :

- `MONGO_USERNAME`
- `MONGO_PWD`
- `MONGO_INITDB_ROOT_USERNAME`
- `MONGO_INITDB_ROOT_PASSWORD`

```bash
MONGO_USERNAME=app_user \
MONGO_PWD=api_password \
MONGO_INITDB_ROOT_USERNAME=root \
MONGO_INITDB_ROOT_PASSWORD=admin_password \
docker compose -f docker-compose.prod.yml up -d --build
```

Vous pouvez remplacer `docker compose` par `docker-compose`.

---

# Partie 4 — MongoDB, variables d’environnement et fichiers ignorés

**20 points**

## 4.1 Démarrer uniquement MongoDB pour l’initialisation

**4 points**

Écrivez la commande permettant de démarrer uniquement le service `db` avec Docker Compose, en fournissant les variables nécessaires à la création de l’utilisateur administrateur MongoDB.

Les valeurs à utiliser sont :

```txt
MONGO_INITDB_ROOT_USERNAME=root
MONGO_INITDB_ROOT_PASSWORD=admin_password
```

```bash
# Votre commande : MONGO_INITDB_ROOT_USERNAME=root MONGO_INITDB_ROOT_PASSWORD=admin_password docker compose -f docker-compose.prod.yml up -d db
```

---

## 4.2 Entrer dans la console MongoDB

**3 points**

Écrivez la commande permettant d’ouvrir la console MongoDB du conteneur `db`.

```bash
# Votre commande : docker compose -f docker-compose.prod.yml exec db mongosh
```

---

## 4.3 Authentification dans MongoDB

**4 points**

Dans la console MongoDB, complétez les commandes permettant de se placer dans la base `admin` et de s’authentifier avec l’utilisateur administrateur.

```js
use admin;

db.auth({
  user: "root",
  pwd: "admin_password",
});
```

---

## 4.4 Créer l’utilisateur utilisé par l’API

**5 points**

Dans MongoDB, écrivez la commande permettant de créer un utilisateur applicatif.

Contraintes :

- nom d’utilisateur : `app_user` ;
- mot de passe : `api_password` ;
- droit : `readWrite` ;
- base concernée : `test`.

```js
// Votre commande : db.getSiblingDB("test").createUser({
  user: "app_user",
  pwd: "api_password",
  roles: [
    { role: "readWrite", db: "test" }
  ]
});
```

---

## 4.5 Fichiers à ignorer

**4 points**

Proposez un contenu minimal pour les fichiers suivants.

### `.gitignore`

Le fichier doit éviter d’envoyer les fichiers d’environnement et les dossiers `node_modules` dans Git.

```gitignore
# Votre réponse : .env
node_modules/

.dockerignore
```

---

### `.dockerignore`

Le fichier doit éviter de copier les fichiers `.env` dans les images Docker.

```dockerignore
# Votre réponse : .env
```

---