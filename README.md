# Installer Docker

Ce document a pour objectif de retracer les pas que j'ai effectués durant mes débuts avec Docker.

Il y aura nécessairement des répétitions avec la [documentation officielle](https://docs.docker.com/install/linux/docker-ce/debian/#install-docker-ce) que j'ai suivie pour arriver à mes fins.

La version de Docker installée est `docker-ce` soit la **Community Edition** de Docker, de manière à tester la technologie sans avoir à payer pour le soft.

## Installation via le repository privé Docker

### Configurer le repository

Mettre à jour la liste des paquets:

```bash
sudo apt-get update
```

Installation des paquets permettant à `apt` d'installer depuis un repository via HTTPS:

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

### Installation de docker-ce avec apt

On remet à jour les listes de paquets avec le nouveau repository:

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

Et après on réessaye le hello-world et ça devrait fonctionner :)

### Gestion des permissions

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
