# Utilisations basiques de Docker

* [Le tutoriel d'orgine](https://training.play-with-docker.com/beginner-linux/)
* [Un article intéressant qui donne des bons exemples](https://medium.com/travis-on-docker/why-and-how-to-use-docker-for-development-a156c1de3b24)
* 

## Lancer une tâche

### Commande ponctuelle

Docker peut être utilisé pour demander à un container d'exécuter une tâche unique ponctuelle.

La commande suivante va démarrer un nouveau container et lui demander d'exécuter la commande `hostname`:

```bash
docker container run alpine hostname
```

On peut ensuite voir la liste des containers existants et leur statut avec:

```bash
docker container ls --all
```

On remarquera que le `CONTAINER ID` du container qu'on vient de lancer est **aussi** le résultat de la commande `hostname` exécutée par ce container.

### Lancer un tty interactif

On peut lancer un container pour exécuter des tâches interactives, comme le shell `bash`.

```bash
docker container run --interactive --tty --rm ubuntu bash
```

Une fois l'image importée et le container démarré, le *prompt* change, indiquant qu'on se situe à présent dans une session `bash`. Cette session se comporte exactement comme si c'était une session sur un autre OS.

* `--interactive` est l'option qui réclame une session interactive
* `--tty` permet d'obtenir un tty comme interface de session

Sans `--tty`, l'interface interactive est primitive (pas de *prompt*, autocomplétion, etc.), et sans `--interactive` on a un *prompt* mais après toutes les commandes n'ont en apparence aucun effet et on ne peut pas fermer le container manuellement.

Le raccourci le plus utilisé pour ces deux options à la fois est `-it`. Dès qu'on a besoin d'une ligne de commande il faut mettre cette combinaison d'options.

### Lancer un service en background

Pour lancer un container MySQL en tâche de fond:

```bash
docker container run \
--detach \
--name mydb \
-e MYSQL_ROOT_PASSWORD=my-secret-pw \
mysql:latest
```

avec `--detach` qui demande à Docker de le lancer en tâche de fond, `--name` qui lui donne un identifiant et `-e` pour spécifier une variable d'environnement disponible à l'intérieur du container. La commande renvoie l'identifiant unique du container de manière à avoir un moyen de le contrôler mais comme on l'a nommé, on n'en a pas besoin.

Pour lister les containers en service et voir le container `mydb`:

```bash
docker container ls
```

À ce stade le service `mysqld` est isolé dans le container car il n'y a pas de port explicitement "publié" vers l'hôte. Chaque port devant être exposé doit être explicitement déclaré.

Pour exécuter en interactif des commandes sur un container déjà en service comme celui-ci:

```bash
docker exec -it mydb \
mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
```

Ici on réutilise une variable d'environnement uniquement disponible dans le **container**. La commande est exécutée à l'intérieur du container et son résultat est affiché grace au mot-clé `exec`.

On peut démarrer une session shell interactive de la même manière:

```bash
docker exec -it mydb sh
```

et constater que la variable d'environnement `MYSQL_ROOT_PASSWORD` est bien présente:

```bash
echo $MYSQL_ROOT_PASSWORD
```

```txt
my-secret-pw
```

## Dockériser une application

On utilise l'application exemple fournie par Docker qu'on peut cloner avec `git`.

```bash
git clone https://github.com/dockersamples/linux_tweet_app
```

Dans cet exemple, on peut voir que l'essentiel du container repose sur le fichier de configuration, appelé `Dockerfile`.

```Dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html
COPY linux.png /usr/share/nginx/html

EXPOSE 80 443 	

CMD ["nginx", "-g", "daemon off;"]
```

Les mots-clés de base sont les suivants:

* `FROM` spécifie l'image de base à utiliser pour construire ce nouveau container;
* `COPY` demande à Docker de copier les fichiers spécifiés pour qu'ils soient disponibles à l'intérieur du container;
* `EXPOSE` spécifie les ports utilisés par le container (publics, côté hôte);
* `CMD` liste les commandes à exécuter lorsque le container est démarré depuis l'image

Pour créer une image sur la base de ce Dockerfile:

```bash
docker image build --tag <dockerid>/linux_tweet_app:1.0 .
```

Où `--tag` permet de *nommer* l'image ainsi créée afin de la retrouver facilement.

Le nom qui est donné est constitué de `<dockerid>/<nomimage>` le docker ID n'est pas *nécessaire* à la création de l'image en local mais permet d'identifier uniquement l'image de manière à pouvoir la stocker dans le repository en ligne [DockerHub](https://hub.docker.com/) à la manière d'un code repository comme GitHub.

Le numéro séparé du nom par `:` est tout simplement le numéro de version de l'image.

**ATTENTION**: noter le `.`, très important ! Il désigne le dossier à partir duquel construire l'image. Ici c'est le dossier courant.

Une fois l'image créée, on peut s'en servir comme les autres images préexistantes qu'on avait utilisées directement jusqu'ici !

```bash
docker container run \
--detach \
--publish 80:80 \
--name linux_tweet_app \
<dockerid>/linux_tweet_app:1.0
```

`--publish` est la nouveauté de cette commande : cette option permet de déclarer et publier un port utilisé par le container pour qu'il soit aussi utilisé par l'hôte et que le service soit disponible publiquement. On peut donc accéder à cette application web en se rendant sur `localhost` dans un navigateur.

## Modifier une application en cours de fonctionnement

### Bind mount

De manière à éviter de reconstruire l'image à chaque légère modification du code, Docker offre des outils pour que les modifications soient instantanément prises en compte à l'intérieur d'un container en fonctionnement. La méthode implique de monter le dossier contenant le code de l'application (situé dans la machine hôte) directement à l'intérieur du système de fichiers du container. C'est ce que Docker désigne par un [`bind mount`](https://docs.docker.com/storage/bind-mounts/).

On réutilise la même application en utilisant cette fois-ci un `bind mount`:

```bash
docker container run \
--detach \
--publish 80:80 \
--name linux_tweet_app \
--mount type=bind,source="$(pwd)",target=/usr/share/nginx/html \
<dockerid>/linux_tweet_app:1.0
```

L'option `--mount` spécifie de monter le dossier de travail directement dans le dossier de fichiers qui est fourni par le serveur web NGINX.

On peut maintenant désormais directement modifier les fichiers du projet dans le système hôte et voir les répercussion sur le service déployé via le navigateur.

**ATTENTION** : les modifications sont effectives à l'intérieur de **l'instance courante** du container, et ne seront pas sauvegardées sans faire des étapes supplémentaires. Les seules modifications qui seront conservées après la fin de l'instance courante du container seront celles sauvegardées dans l'hôte, pas à l'intérieur du container.

### Mettre à jour l'image

Pour que les changements faits à l'application soient persistants, **il faut construire une nouvelle version de l'image**.

Pour voir les images disponibles en local:

```bash
docker image ls
```

On a maintenant deux versions pour l'image `linux-tweeter-app` qui ne déploieront pas la même version du site avec `docker container run`.

À condition de choisir des ports différents et des noms de containers différents, les deux versions peuvent fonctionner en même temps.