# Gestion des volumes

## Volumes gérés par Docker

Créer un volume avec un **nom** (il est conseillé d'avoir un nom):

```bash
docker volume create --name=le-nom-du-volume
```

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

## Monter un dossier local comme un volume

Pour utiliser un dossier local comme un volume et modifier le code à la volée, il faut **monter** le dossier comme un volume.

Apparemment il y aurait une différence entre les *bind mounts* et les *volumes* mais j'ai toujours pas bien compris ni vu de différence en les utilisant. [Voir la documentation sur ces questions](https://docs.docker.com/storage/). Il est probable que les différences apparaissent surtout au déploiement et en environnement de production.