# Installation de docker compose

- si vous utilisez *Docker for Mac* ou *Docker for Windows*, vous avez déjà accès à Docker Compose via la commande *docker compose*

- si vous êtes sur un environnement Linux, installez la dernière version de *Docker Compose* en suivant les instructions suivantes:

```
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```

Vérifiez que le binaire est correctement installé avec la commande suivante:

```
$ docker compose version
```