# Installation de Docker

Ce document a pour objectif de retracer les étapes effectuées durant les premiers essais de Docker.

Il y aura nécessairement des répétitions avec la [documentation officielle](https://docs.docker.com/install/linux/docker-ce/debian/#install-docker-ce) que suivie pour réaliser ces tâches.

## Installer Docker sur Windows

C'est beaucoup plus simple si on veut l'installer sur Windows :

* Mettre Windows à jour
* Télécharger l'installeur
* Installer

C'est **après** l'installation que les problèmes commencent !

* Activation de la virtualisation Hyper-V dans le BIOS/UEFI
* Conflits de fonctionnement Docker/VirtualBox
* Problèmes de volumes

Si c'est possible, servez-vous de Docker sur Linux ;)

### Docker-compose

L'utilitaire `docker-compose` est inclus dans Docker pour Windows.

## Installer Docker sur Debian Linux

La version de Docker installée est `docker-ce` soit la **Community Edition** de Docker.

### Installation via le *repository* privé Docker

#### Configurer le *repository*

Mettre à jour la liste des paquets:

```bash
sudo apt-get update
```

Installation des paquets permettant à `apt` d'installer depuis un *repository* via HTTPS:

```bash
sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
```

On ajoute la clé GPG de Docker:

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

On ajoute le repository `stable` de Docker à la liste des sources:

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

#### Installation de `docker-ce` avec `apt`

On remet à jour les listes de paquets avec le nouveau *repository*:

```bash
sudo apt-get update
```

On installe la dernière version de `docker-ce`:

```bash
sudo apt-get install docker-ce
```

On teste l'installation avec l'image de test:

```bash
sudo docker run hello-world
```

Si ça ne fonctionne pas c'est peut-être que le **service** n'est pas activé:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Et après on réessaye le `hello-world` et ça devrait fonctionner 😀

#### Gestion des permissions

Ajout du groupe `docker` s'il n'existe pas:

```bash
sudo groupadd docker
```

Ajout de l'utilisateur `user` au groupe:

```bash
sudo usermod -aG docker user
```

Alternative en utilisant les variables d'environnement:

```bash
sudo usermod -aG docker $USER
```

Afin de tester si on peut utiliser la commande `docker` sans `sudo`, il faut **au moins** fermer et rouvrir la session, voire redémarrer.

### Installation de `docker-compose`

[Dockementation de `docker-compose`](https://docs.docker.com/compose/).

L'utilitaire `docker-compose` permet de décrire un réseau de services pris en charge par des containers qui peuvent communiquer entre eux. C'est utile pour le développement : afin de séparer la base de données du service Node qu'on développe, par exemple.

L'installation de `docker-compose` implique de vérifier sur [le repo Github](https://github.com/docker/compose/releases) quelle est la dernière version de `docker-compose` à avoir été publiée.

Une fois muni de cette information, on utilise le *one-liner* qui suit en remplaçant le numéro de version par celui qui convient. Préférer les versions taggées `lastest release` plutôt que `pre-release`.

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

On rajoute les permissions d'exécution sur le binaire `docker-compose`:

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

> **ATTENTION**: Si la commande ne fonctionne pas malgré tout, c'est que le chemin `/usr/local/bin` n'est pas dans le `PATH` utilisateur. Ou que l'utilisateur ne fait pas partie du groupe `docker` mais ça normalement ça a déjà été géré [durant l'installation de docker](#gestion-des-permissions).

### Ajout de la complétion de ligne de commande

#### ZSH

```bash
curl -fLo ~/.zsh/completion/_docker https://raw.github.com/felixr/docker-zsh-completion/master/_docker
```

et on ajoute

```zsh
fpath=(~/.zsh/completion $fpath)
autoload -Uz compinit && compinit -i
```

au fichier `~/.zshrc`, de manière à déclencher la complétion.

On redémarre le shell:

```zsh
exec $SHELL -l
```

#### BASH

@TODO