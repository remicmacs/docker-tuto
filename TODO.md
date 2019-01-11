# À approfondir

- Registry privé ? Plusieurs registries ?
- Volumes
- Windows
- Swarm

## Swarm

Un *swarm* est un *cluster* de *runtimes* Docker gérés collectivement.

Docker Swarm propose des outils d'orchestration des containers (load balancing, DNS, networking, réplication, etc.). Cet outil permet de simplifier le déploiement des containers en production.

On n'est pas obligé d'avoir plusieurs *runtimes* pour faire un *swarm*, on peut commencer par un *swarm* d'un runtime unique.

### Création

[Documentation Docker - Create a swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/)

Pour un *swarm* avec un *node* unique:

```bash
docker swarm init
```

Pour un *swarm* avec plusieurs *nodes*, on créé le *node* `manager` en spécifiant son adresse réseau, de manière à ce qu'il puisse exposer un port pour la communication entre les *nodes*:

```bash
docker swarm init --advertise-addr $(hostname -i)
```

Cette commande va renvoyer un *token* permettant de rejoindre le *swarm* aux futurs nodes avec `docker swarm join`.

Ensuite les commandes relatives à l'administration du *swarm* ne pourront être prises en charge que par le `manager`.
