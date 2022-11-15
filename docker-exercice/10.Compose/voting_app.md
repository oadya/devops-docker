# Voting application

Dans ce lab, nous allons illustrer l’utilisation de *Docker Compose* et lancer l’application *Voting App*. Cette application est très utilisée pour des présentations et démos, c'est un bon exemple d'application micro-services simple.

## Vue d’ensemble

L’application *Voting App* est composée de plusieurs micro-services, ceux utilisés pour la version 2 sont les suivants:

![Voting App architecture](./images/architecture-v2.png)

* vote-ui: front-end permettant à un utilisateur de voter entre 2 options
* vote: back-end réceptionnant les votes
* result-ui: front-end permettant de visualiser les résultats
* result: back-end mettant à disposition les résultats
* redis: database redis dans laquelle sont stockés les votes
* worker: service qui récupère les votes depuis redis et consolide les résultats dans une database postgres
* db: database postgres dans laquelle sont stockés les résultats

##  Récupération des repos GitLab

Lancez les commandes suivantes afin de récupérer le répo de chaque microservice ainsi que celui de configuration:

```
mkdir VotingApp && cd VotingApp
for project in config vote vote-ui result result-ui worker; do
  git clone https://gitlab.com/voting-application/$project
done
```

## Le format de fichier docker-compose.yml

Plusieurs fichiers, au format Docker Compose, sont disponibles dans *config/compose*. Ils décrivent l’application  pour différents environnements.

Le fichier qui sera utilisé par défaut est le fichier *docker-compose.yml* dont le contenu est le suivant:

```
services:
  vote:
    build: ../../vote
    # use python rather than gunicorn for local dev
    command: python app.py
    depends_on:
      redis:
        condition: service_healthy
    ports:
      - "5002:80"
    volumes:
      - ../../vote:/app
    networks:
      - front-tier
      - back-tier

  vote-ui:
    build: ../../vote-ui
    depends_on:
      vote:
        condition: service_started
    volumes:
      - ../../vote-ui:/usr/share/nginx/html
    ports:
      - "5000:80"
    networks:
      - front-tier
    restart: unless-stopped

  result:
    build: ../../result
    # use nodemon rather than node for local dev
    command: nodemon server.js
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ../../result:/app
    ports:
      - "5858:5858"
    networks:
      - front-tier
      - back-tier

  result-ui:
    build: ../../result-ui
    depends_on:
      result:
        condition: service_started
    ports:
      - "5001:80"
    networks:
      - front-tier
    restart: unless-stopped

  worker:
    build:
      context: ../../worker
      dockerfile: Dockerfile.${LANGUAGE:-dotnet}
    depends_on:
      redis:
        condition: service_healthy
      db:
        condition: service_healthy
    networks:
      - back-tier

  redis:
    image: redis:6.2-alpine3.13
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: "5s"
    ports:
      - 6379:6379
    networks:
      - back-tier

  db:
    image: postgres:13.2-alpine
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    volumes:
      - "db-data:/var/lib/postgresql/data"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: "5s"
    ports:
      - 5432:5432
    networks:
      - back-tier

volumes:
  db-data:

networks:
  front-tier:
  back-tier:
```

Ce fichier est très intéressant car il définit également des volumes et networks en plus des services.

Ce n’est cependant pas un fichier destiné à être lancé en production notamment parce qu'il utilise le code local et ne fait pas référence à des images existantes pour les services *vote-ui*, *vote*, *result-ui*, *result* et *worker*.

## Lancement de l’application

Depuis le répertoire *config/compose*, lancez l’application à l'aide de la commande suivante (le fichier *docker-compose.yml* sera utilisé par défaut):

```
docker compose up -d
```

Les étapes réalisées lors du lancement de l’application sont les suivantes:
* création des networks front-tier et back-tier
* création du volume db-data
* construction des images pour les services *vote-ui*, *vote*, *result-ui*, *result*, *worker* et récupération des images *redis* et *postgres*
* lancement des containers pour chaque service

## Les containers lancés

Avec la commande suivante, listez les containers qui ont été lancés et assurez-vous qu'ils sont tous dans le status *running*

```
docker compose ps
```

## Les volumes créés

Listez les volumes avec la CLI, et vérifiez que le volume défini dans le fichier *docker-compose.yml* est présent.

```
docker volume ls
```

Le nom du volume est prefixé par le nom du répertoire dans lequel l’application a été lancée.

```
DRIVER    VOLUME NAME
local     compose_db-data
```

Par défaut ce volume correspond à un répertoire créé sur la machine hôte.

## Les networks créés

Listez les networks avec la CLI. Les deux networks définis dans le fichier *docker-compose.yml* sont présents.

```
docker network ls
```

De même que pour le volume, leur nom est préfixé par le nom du répertoire.

```
NETWORK ID     NAME                 DRIVER    SCOPE
71d0f64882d5   bridge               bridge    local
409bc6998857   compose_back-tier    bridge    local
b3858656638b   compose_front-tier   bridge    local
2f00536eb085   host                 host      local
54dee0283ab4   none                 null      local
```

Note: comme nous sommes dans le contexte d’un hôte unique le driver utilisé pour la création de ces networks est du type bridge. Il permet la communication entre les containers tournant sur une même machine.

## Utilisation de l’application

Nous pouvons maintenant accéder à l’application:

nous effectuons un choix entre les 2 options depuis l'interface de vote à l'adresse http://HOST_IP:5000 

![Vote interface](./images/vote_interface.png)

nous visualisons le résultat depuis l'interface de résultats à l'adresse http://HOST_IP:5001

![Result interface](./images/result_interface.png)

Note: remplacez HOST_IP par localhost ou bien par l'adresse IP de la machine sur laquelle a été lancée l'application

## Scaling du service worker

Par défaut, un container est lancé pour chaque service. Il est possible, avec l'option *--scale*, de changer ce comportement et de scaler un service une fois qu’il est lancé.

Avec la commande suivante, augmenter le nombre de worker à 2.

```
docker compose up -d --scale worker=2
```

Vérifiez qu'il y a à présent 2 containers pour le service worker:

```
docker compose ps
```

Notes: il n’est pas possible de scaler les services *vote-ui* et *result-ui* car ils spécifient tous les 2 un port, plusieurs containers ne peuvent pas utiliser le même port de la machine hôte

```
docker compose up -d --scale vote-ui=3
...
ERROR: for vote-ui  Cannot start service vote-ui: driver failed programming external connectivity on endpoint compose_vote-ui_2 (6274094570a329e3a4d9bdcdf4d31b7e3a8e3e7e78d3cc362ad56e14341913da): Bind for 0.0.0.0:5000 failed: port is already allocated
```

## Suppression de l’application

Avec la commande suivante, stoppez l’application. Cette commande supprime l’ensemble des éléments créés précédemment à l'exception des volumes (afin de ne pas perdre de données)

```
docker compose down
```

Afin de supprimer également les volumes utilisés, il faut ajouter le flag *-v*:

```
docker compose down -v
```