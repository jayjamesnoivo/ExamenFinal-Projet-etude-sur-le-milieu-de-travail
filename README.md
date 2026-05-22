# ÉNONCÉ Examen final — Docker Compose et application multi-services

**Durée :** 2 heures (+ 45 minutes pour les étudiants du SAIDE)
**Travail :** Individuel
**Remise :** Teams

---

## Modalités de remise

Vous devez répondre directement dans ce fichier Markdown.

Avant la remise, renommez le fichier selon le format suivant :

```txt
nom_prenom_examen_final.md
```

Exemple :

```txt
Dupont_Marie_examen_final.md
```

Vous devez conserver la structure de l’examen, les numéros de questions et les blocs de code fournis.

Lorsque vous complétez une commande ou un fichier de configuration, écrivez votre réponse directement sous la question ou dans l’espace prévu.

La remise doit être faite sur Teams avant la fin de la période d’examen.

---

## Consignes générales

Répondez directement dans le document fourni ou dans un fichier de réponse clairement structuré.

Lorsque vous écrivez du code ou une configuration, portez une attention particulière :

- à l’indentation YAML ;
- aux noms des services ;
- aux chemins des dossiers ;
- aux noms des volumes ;
- aux variables d’environnement ;
- à la cohérence entre les fichiers.

Dans cet examen, les deux syntaxes Docker Compose sont acceptées :

```bash
docker compose
```

ou :

```bash
docker-compose
```

Par exemple, les deux commandes suivantes sont considérées comme équivalentes :

```bash
docker compose -f docker-compose.prod.yml up -d --build
```

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

---

## Notions non évaluées dans cet examen

Les notions suivantes ne sont pas évaluées dans cet examen :

- mise en ligne sur un VPS ;
- configuration SSH ;
- nom de domaine ;
- DNS ;
- certificat TLS / HTTPS ;
- Certbot ;
- PM2 ;
- configuration détaillée du reverse proxy ;
- routage Nginx.

Le service `reverse-proxy` peut apparaître dans les fichiers, mais il est considéré comme fourni et fonctionnel. Vous n’avez pas à écrire sa configuration Nginx.

---

# Contexte du projet

Vous travaillez sur une application web composée de plusieurs services.

La structure du projet est la suivante :

```txt
projet-final/
├── client/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   ├── package.json
│   └── src/
├── api/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   ├── package.json
│   └── src/
│       └── index.js
├── reverseproxy/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   └── conf/
├── docker-compose.dev.yml
└── docker-compose.prod.yml
```

L’application contient :

| Service         | Description                    |
| --------------- | ------------------------------ |
| `client`        | Application React/Vite         |
| `api`           | API Node.js/Express            |
| `db`            | Base de données MongoDB        |
| `reverse-proxy` | Service fourni, déjà configuré |

L’API se connecte à MongoDB avec une URI semblable à :

```js
mongodb://db
```

En production, l’API utilise aussi des variables d’environnement pour s’authentifier à la base de données.

---

# Partie 1 — Commandes Docker Compose et lecture du projet

**20 points**

## 1.1 Commandes de base avec Docker Compose

**10 points**

Écrivez les commandes permettant d’effectuer les actions suivantes.

Vous pouvez utiliser `docker compose` ou `docker-compose`.

### a) Construire et lancer l’environnement de développement

```bash
# Votre commande :
```

---

### b) Afficher les services en cours d’exécution pour l’environnement de développement

```bash
# Votre commande :
```

---

### c) Afficher les logs du service `api` en temps réel

```bash
# Votre commande :
```

---

### d) Arrêter tous les services de l’environnement de développement

```bash
# Votre commande :
```

---

### e) Construire et lancer l’environnement de production en arrière-plan

```bash
# Votre commande :
```

---

## 1.2 Vérifier les services déclarés

**4 points**

Écrivez la commande permettant d’afficher la liste des services déclarés dans le fichier :

```txt
docker-compose.prod.yml
```

```bash
# Votre commande :
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

```txt

```

---

# Partie 2 — Dockerfile et environnement de développement

**30 points**

## 2.1 Compléter le Dockerfile de production de l’API

**10 points**

On vous donne le fichier `api/Dockerfile.prod` incomplet suivant.

Complétez les éléments manquants.

```dockerfile
FROM ____________________

WORKDIR ____________________

COPY package.json ./

RUN ____________________

COPY . .

EXPOSE ____________________

CMD ____________________
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
    "prod": "____________________"
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
      - type: __________
        source: ./client
        target: /home/node
      - type: __________
        target: /home/node/node_modules
    ports:
      - "__________:__________"

  api:
    build:
      context: ./api
      dockerfile: Dockerfile.dev
    volumes:
      - type: __________
        source: ./api/src
        target: /app/src
    ports:
      - "__________:__________"
    depends_on:
      - __________

  db:
    image: mongo:7
    volumes:
      - type: __________
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
  __________:
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
      dockerfile: __________
    restart: __________

  api:
    build:
      context: ./api
      dockerfile: __________
    environment:
      - MONGO_USERNAME
      - MONGO_PWD
      - NODE_ENV=__________
    restart: __________
    depends_on:
      - __________

  db:
    image: mongo:7
    volumes:
      - type: volume
        source: __________
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
    restart: __________

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
  __________:
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
# Votre commande :
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
____________________=app_user \
____________________=api_password \
____________________=root \
____________________=admin_password \
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
# Votre commande :
```

---

## 4.2 Entrer dans la console MongoDB

**3 points**

Écrivez la commande permettant d’ouvrir la console MongoDB du conteneur `db`.

```bash
# Votre commande :
```

---

## 4.3 Authentification dans MongoDB

**4 points**

Dans la console MongoDB, complétez les commandes permettant de se placer dans la base `admin` et de s’authentifier avec l’utilisateur administrateur.

```js
__________;

db.auth({
  user: "__________",
  pwd: "__________",
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
// Votre commande :
```

---

## 4.5 Fichiers à ignorer

**4 points**

Proposez un contenu minimal pour les fichiers suivants.

### `.gitignore`

Le fichier doit éviter d’envoyer les fichiers d’environnement et les dossiers `node_modules` dans Git.

```gitignore
# Votre réponse :
```

---

### `.dockerignore`

Le fichier doit éviter de copier les fichiers `.env` dans les images Docker.

```dockerignore
# Votre réponse :
```

---

# Barème récapitulatif

| Partie    |                                                  Sujet |  Points |
| --------- | -----------------------------------------------------: | ------: |
| Partie 1  |          Commandes Docker Compose et lecture du projet |      20 |
| Partie 2  |           Dockerfile et environnement de développement |      30 |
| Partie 3  |                            Environnement de production |      30 |
| Partie 4  | MongoDB, variables d’environnement et fichiers ignorés |      20 |
| **Total** |                                                        | **100** |

---

# Rappel de fin d’examen

Avant de remettre, vérifiez que :

- vos commandes utilisent le bon fichier Compose ;
- vos services portent les bons noms ;
- vos fichiers YAML sont correctement indentés ;
- les volumes sont bien déclarés ;
- les ports sont cohérents ;
- les variables sensibles ne sont pas écrites directement dans les fichiers de production ;
- les fichiers `.env` et `node_modules` sont ignorés correctement ;
- votre syntaxe Docker Compose est cohérente.

Les deux syntaxes suivantes sont acceptées :

```bash
docker compose
```

et :

```bash
docker-compose
```
