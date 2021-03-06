# Tutoriel Docker

- [Liens utiles](#liens-utiles "Liens utiles")
- [Installation](#installation "Installation")
- [Commandes Docker](#commandes-docker "Commandes Docker")
- [Les attributs](#les-attributs "Les attributs")
- [Test de Docker](#test-de-docker "Test de Docker")
- [Docker-compose](#docker-compose "docker-compose")
- []()

## Liens utiles

- Site officiel Docker : [Docker](https://www.docker.com/ "Docker")
- Gestionnaire de containers : [Docker-Compose](https://docs.docker.com/compose/ "Docker-Compose")
- Hébergeur de containers : [DockerHub](https://hub.docker.com/ "DockerHub")
- S'amuser avec Docker sans avoir a l'installer : [Online Docker Lab](https://labs.play-with-docker.com/ "Online Docker Lab")

## Installation

Installer Docker en fonction de l'OS : [Docker Installer](https://www.docker.com/get-started "Docker Installer")
[Documentation Docker](https://docs.docker.com/ "Documentation Docker")

## Commandes Docker

- Version et info de Docker : `docker -v`, `docker version` et `docker info`
- Aide sur les commandes tapées : `docker MaCommandeDocker --help`
- Chercher l'image du container souhaité sur [DockerHub](https://hub.docker.com/ "DockerHub") ou via la commande `docker search MonContainer`
- Télécharger l'image d'un container : `docker pull mon_container`
- Lister les images des containers téléchargés : `docker images` [-a pour lister toutes les images]
- Lancer le container souhaité et les paramètres associés : `docker run mon_container -p 3300 -q mon_qarametre`
  - Liste des parametres possibles :
    - port : -p
    - azerty
- Lister les images téléchargées : `docker image ls` [--all ]
- Lister les containers qui sont actuellement lancés : `docker ps`
- Utiliser le container : `docker exec mon_container`
- Créer le container __mon_container__ qui exécutera le DOCKERFILE : `docker build -t friendlyhello .`
- Arrêter un container : `docker stop mon_container` (ou l'ID du container)
- Supprimer un container ou une image :
  - tous les containers arrêté : `docker container prune` (ou l'ID du container)
    - `--force, -f` : ne renvoi pas un message de confirmation
  - toutes les images non utilisées : `docker image prune -a` (ou l'ID de l'image)
    - `--all, -a` : toutes les images non utilsées
    - `--force, -f` : ne renvoi pas un message de confirmation
  - un container : `docker rm mon_container` (ou l'ID du container)
  - une image : `docker image rm mon_image` (ou l'ID de l'image)
- `docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mon_container` : renvoi l'adresse IP du container

### Construire son container

- Créer le container à partir du DOCKERFILE : `docker build -t nom_de_mon_image .`
  - `-f DOCKERFILE_Name`, ajouter cet attribut permet de spécifier un fichier qui a un nom différent du DOCKERFILE (`docker build -t nom_de_mon_image -f DOCKERFILE_Name .`)

## Les attributs

- `-it` : pour créer une nouvelle session docker dans le container
  - `-i`
  - `-t`
- `-d` : pour exécuter le container en arrière plan
- `-e VAR=1` : pour mettreen place dans le container la variable __VAR=1__
```shell
# Créer une nouvelle session dans le container `ubuntu_bash`
docker exec -it ubuntu_bash bash
// Créer une nouvelle session dans le container `ubuntu_bash` avec la variable `VAR=1`
docker exec -it -e VAR=1 ubuntu_bash bash
// Créer dans le container `ubuntu_bash` en arrière plan, le dossier `/tmp/execWorks`
docker exec -d ubuntu_bash touch /tmp/execWorks
```

## Test de Docker

- Lancer la commande `docker run hello-world` pour tester l'installation de docker.
```ini
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

## Le DOCKERFILE

Le DOCKERFILE  est un fichier Docker qui permet de configurer le container lors de sa création. Il commencera toujours par __FROM__.

```dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# to be able to use "nano" with shell on "docker exec -it [CONTAINER ID] bash"
ENV TERM xterm

# Modifie la ligne "bind-address=127.0.0.1" en "bind-address=0.0.0.0 dans le fichier "/etc/mysql/mariadb.conf.d/50-server.cnf"
# Grâce à la commande BASH "sed"
RUN sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/mariadb.conf.d/50-server.cnf


# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

### Configuration du DOCKERFILE

- `#` : permet de mettre un commentaire sur la ligne
- `FROM [...]` : indique la version de l'OS que l'on souhaite utiliser dans ce container. On peut trouver l'ensemble des systèmes sur [DockerHub](https://hub.docker.com/ "DockerHub")
- `WORKDIR [...]` : permet de définir l'endroit où l'on souhaite travailler
- `COPY . [...]` : copie l'emplacement courant du projet (ou le dossier/fichier spécifié par un chemin à remplacer par le ".") à l'emplacement indiqué ([...] souvent le même que le WORKDIR). _Exemple (comme ci-dessus) : choix de l'emplacement "WORKDIR /app" et copie du projet dans /app "COPY . /app"_``
- `CMD [...]` & `ENTRYPOINT [...]` permettent d'exécuter la commande lors du lancement du container
  - `CMD [...]` ne permet pas l'overwrite de cette commande contrairement `ENTRYPOINT [...]` __(A VERIFIER)__
- `\` : en fin de ligne permet d'aller à la ligne

### Exécution de commandes via le DOCKERFILE

Il est possible d'exécuter des commandes dans le container que l'on construit grâce à la commande `RUN [commande]`.  
__/!\ ATTENTION : Il est à noter que si on met à jour des packets sur une seule ligne, la commande ayant déjà était exécutée lors de la compilation, ne le sera pas ré-exécutée. Pour pouvoir mettre à jour correctement un paquet, il est préférable de mettre les différents paquets sur différentes lignes (comme ci-dessous) /!\\__

```docker
RUN sudo apt-get update && apt-get -y install \
  git \
  mysql
```

# Docker-compose

Le fichier **docker-compose.yml** pert de configurer un ensemble de conteneurs et de les manager (deploy, stop, restart, rm...) via un seul fichier.

Il y a possibilité d'ajouter un second fichier (ou plus) **docker-compose.override.yml** qui pourra prendre en paramètre les volumes, ports... et pourra override le `docker-compose.yml`. [Documentation](https://docs.docker.com/compose/extends "docker-compose.override.yml")

Liste des commandes possibles : 

```shell
docker-compose up -d
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d # Custom docker-compose.yml
```
