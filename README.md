# TP4 : Docker et Docker Compose - Application MERN

Application MERN (MongoDB, Express, React, Node.js) conteneurisée avec Docker et orchestrée avec Docker Compose.

## Prérequis

- Docker Desktop
- Git

## Structure du projet

```
MyAppReact/
├── client/            # Application React (Vite)
│   ├── Dockerfile
│   └── .dockerignore
├── server/            # API Express
│   ├── Dockerfile
│   ├── .dockerignore
│   └── src/index.mjs
├── docker-compose.yml
└── README.md
```

## Étapes réalisées

### 1. Cloner le projet

```bash
git clone https://github.com/wiem-kb/MyAppReact.git
cd MyAppReact
```

### 2. Créer les Dockerfiles

- **server/Dockerfile** : image `node:lts-alpine`, `WORKDIR /app`, copie de `package*.json`, `npm install`, copie du code, `EXPOSE 9000`, démarrage avec `npm start`.
- **client/Dockerfile** : image `node:lts-alpine`, installation des dépendances, `npm run build`, installation du serveur léger `serve`, `EXPOSE 3000`, puis `serve -s dist -l 3000`.
  Le client utilise Vite : le dossier de build est `dist` (et non `build`).

Construction des images :

```bash
docker build -t mern-server ./server
docker build -t mern-client ./client
```

> Remarque : l'énoncé donne `docker build -t mern-server ./client` pour le client, ce qui écraserait l'image du serveur. Le nom correct est `mern-client`.

### 3. Créer le réseau Docker

```bash
docker network create mern-network
```

### 4. Lancer MongoDB

```bash
docker run -d --name mongodb --network mern-network mongo
```

### 5. Lancer le serveur et le client

```bash
docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mydb mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```

Le serveur se connecte à MongoDB grâce à la variable d'environnement `MONGO_URI` (le nom `mongodb` est résolu par le réseau Docker).

### 6. Docker Compose

Le fichier `docker-compose.yml` définit trois services (`mongodb`, `server`, `client`) et le réseau `mern-network` (driver bridge). Le serveur dépend de `mongodb` et le client dépend du serveur.

Avant de lancer Compose, supprimer les conteneurs manuels :

```bash
docker rm -f client server mongodb
docker network rm mern-network
```

Lancement :

```bash
docker compose up --build
```

Arrêt :

```bash
docker compose down
```

## Accès

- Client React : http://localhost:3000
- API Express : http://localhost:9000/api/health