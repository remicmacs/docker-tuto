# Utiliser un container pour MongoDB

Pour ne pas avoir à installer MongoDB sur sa machine, avoir un container peut être utile.

## Développement avec un container

Ce que ça change:

* Pas de serveur MongoDB qui prend de la place et ouvre des vulnérabilités quand on en a plus besoin
* Le container MongoDB ne tombe pas en panne
* On peut se servir du même container d'une application à l'autre, sans mélanger les données (volumes)

Il y a quand même des désavantages bien sûr:

* Devoir apprendre un nouvel outil et les commandes associées
* Faire attention pour la persistance des données pour éviter de les perdre

Le problème de la persistance des données n'en est normalement pas un car si vous êtes un développeur sérieux, vous devriez avoir des jeux de données de test stockées à part et prêt à être insérées dans la base de développement.

**Récapitulons**:

La méthode de développement quand on se sert d'un container pour MongoDB :

1. On lance MongoDB (et on n'a pas besoin de l'installer :D)
2. On insère ses données de test dans MongoDB
3. On se sert de MongoDB
4. On éteint MongoDB

Et quand on a fini on revient au numéro 1, et c'est comme si l'installation de MongoDB était toute propre !! :D

## Utilisation du container

### [Optionnel] Créer un volume

Il faut créer un volume Docker pour la persistance des données de la base Mongo.
Encore une fois c'est à faire *si on en a vraiment besoin*. Je ne saurais trop conseiller de **créer des données de test** et de scripter l'insertion de ces données dans la base à chaque fois qu'on la lance. À des fins de développement **il ne devrait pas y avoir besoin de conserver les données**. Sinon c'est que vous avez du travail non sauvegardable et non récupérable dans la base de données **qui ne peut pas être transférer dans un répertoire de gestion de version comme Git**.

Si vous voulez quand même avoir une persistance des données (je comprends dès fois y'a besoin):

On crée un volume avec un **nom**:

```bash
docker volume create --name=mongo-dev
```

Et... voilà c'est tout !

#### Deux trois autres trucs pour la gestion des volumes

Lister les volumes existants:

```bash
docker volume ls
```

Obtenir des informations sur un volume:

```bash
docker volume inspect <nom_volume>
```

Retirer tous les volumes qui ne sont pas en cours d'utilisation:

```bash
docker volume prune
```

Mais on peut aussi **labeliser** ses volumes pour éviter de les supprimer avec prune.

Création d'un volume avec label:

```bash
docker volume --name mongo-dev --label dev
```

Suppression de tous les volumes sauf ceux labellisés `dev`:

```bash
docker volume prune --filter label!=dev
```

On peut aussi supprimer les volumes nominalement:

```bash
docker volume rm mongo-dev
```

### Démarrer le container MongoDB

```bash
docker run --rm \
--publish 27017:27017 \
--name mongo-dev \
--detach \
mongo:3.4-jessie
```

Avec :

* `--rm` pour que le container soit détruit après utilisation (évite l'accumulation d'images intermédiaires);
* `--publish 27017:27017` pour que le port 27017 de l'hôte corresponde au port 27017 du container et que MongoDB soit accessible normalement;
* `--name` pour nommer le container, ce qui rend sa manipulation plus commode;
* `--detach` pour qu'il fonctionne en arrière-plan.

Pour démarrer le container MongoDB avec un volume sur lequel les données seront écrites:

```bash
docker run --rm \
-p 27017:27017 \
-v mongo-volume:/data/db \
--name mongo-dev \
-d mongo:3.4-jessie
```

On utilise un volume nommé `mongo-volume` pour qu'il prenne la place du dossier `/data` à l'intérieur du container.
Ici j'ai utilisé les versions courtes existantes des options.

**ATTENTION** : la persistance des données ne fonctionnera qu'en liant le volume au dossier interne `/data/db` et pas `/data`. Pour une raison inconnue le lier à `/data` ne fera que créer les dossiers `/data/db` et `/data/configdb`, mais **pas** leurs contenus. ¯(°_o)/¯

### Arrêter le container

Pour un container qui s'appelle `mongo-dev`.

```bash
docker stop mongo-dev
```
