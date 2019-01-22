# Guide de build d'une image Docker

[Tutoriel Microservices de Docker](https://training.play-with-docker.com/microservice-orchestration/)

## Principe de construction d'une image

Les images fonctionnent par couches : une image de base et plusieurs couches qui se superposent à chaque instruction dans le `Dockerfile`.

Cela permet de rendre les modifications immutables mais simples à modifier, légères à manipuler.

Il vaut mieux avoir plusieurs images indépendantes pour mieux gérer ses dépendances. Par exemple on peut imaginer une image `node:8.15` qui ne contient rien de plus que NodeJS et NPM, puis une nouvelle image qui utilise cette image comme base contenant juste les dépendances basiques (express, react, etc.), puis une autre image contenant les dépendances spécifiques au projet, etc.

Si on veut créer une image pour un projet existant, avec des dépendances déjà listées dans un fichier comme `package.json`, par exemple, il vaut mieux **d'abord** copier uniquement le fichier en question et installer les dépendances **puis** copier le reste du code du projet. Cela permet à Docker de faire une image pour les dépendances qui ne sera pas modifiée au prochain build si le fichier `package.json` n'a pas été modifié. Tandis que si la totalité du dossier est copiée **puis** les dépendances sont installées, alors à chaque build Docker installera à nouveau les dépendances même s'il n'y a qu'un caractère modifié dans le code source.

### Avant de créer une nouvelle image

Identifier les dépendances dont on a besoin pour le service qu'on construit.

En général on part d'une image que l'équipe de Docker fourni sur DockerHub. DockerHub propose des images pour les langages principaux, accompagnés de leurs outils de gestion de dépendances principaux (ex: Node + npm).

Partant de là, on évalue si on a besoin de librairies supplémentaires et on les ajoute soit avec le système d'installation de paquets de la distribution Linux sur laquelle l'image de base est fondée (`apt` pour les systèmes debian, `apk` pour les images alpine, etc.), soit avec le gestionnaire de dépendances (on peut par exemple faire un `npm i eslint -d` pour installer `eslintè comme dépendance de développement d'un projet).

On configure ensuite les volumes et/ou les *bind mounts* requis pour le fonctionnement du serveur ou le développement. En général si c'est du fonctionnement c'est un volume qu'il faut, un *bind mount* pour développer et voir les modifications.

## Créer des réseaux virtuels avec plusieurs services

Mieux vaut plusieurs images fonctionnant ensemble qu'une grosse image complexe qui réunit tout et qui est pénible à debugger.

`docker-compose` est l'outil permettant de décrire un réseau de services containairisés avec un fichier de conf YAML.

Comme c'est un sujet à part entière, je renvoie à la documentation et [au tuto listé ci-dessus](https://training.play-with-docker.com/microservice-orchestration/)