# REPONSES


## B) Déployer la base de donnée

### 1) Déployer la base de données (mode standalone)

**docker pull mongo:latest** #téléchargement de la derrnière image de mongo trouvée sur le docker hub
**docker run -name mongodevops -d mongo:latest** #création du conteneur et on le run, *mongodevops* étant le nom donnée à notre conteneur
**docker ps** #permet d'obtenir des détails sur les conteneurs

**docker run -it --network mongodevopsclient --rm mongo mongosh --host mongodevops** #Nous voulions installer un client mongo en suivant le tutoriel sur docker huh cependant, nos essais n'ont pas marché. Après plusieurs essais, nous avons fini par changer de méthode en utilisant le docker file.

*slack.yml :* 

*version: '3.1'*

*services:*

  *mongo:*
    *image: mongo*
    *restart: always*
    *ports:*
      *- 27017:27017 #port utilisé/port du conteneur*
    *environment:*
      *MONGO_INITDB_ROOT_USERNAME: le_user*
      *MONGO_INITDB_ROOT_PASSWORD: le_password*

  *mongo-express:*
    *image: mongo-express*
    *restart: always*
    *ports:*
      *- 8081:8081*
    *environment:*
      *ME_CONFIG_MONGODB_URL: mongodb://user_non_caché:mdp_non_caché@mongo:27017/*

*slack.yml* est le nom de la première version du fichier. Nous l'avons crée avec le terminal et avons essayé de le modifier dans ce même terminal avec vim en vain. La cause étant que nous ne pouvions enregistrer puis quittre l'éditeur vim et revenir sur le terminal après l'édition. Finalement, après édition manuelle du fichier, nous avons lancé la commande **docker-compose -f stack.yml up**
Nous ne savions pas qu'il fallait ce connecter au mongo-express via l'adresse localhost:8081 et nous avions cru ne pas avoir trouvé la solution.
De plus, lorsque l'on faisait la commande **docker exec -it mongodevops bash** pour entrer dans le conteneur,nous n'arivions pas à nous connecter à la base de donnée via le mongosh.

Après avoir suivit la correction, nous avons approté quelques modifications.

*docker-compose.yml :*

*version: '3.1'*

*services:*

  *mongo:*
    *image: mongo*
    *restart: always*
    *ports:*
      *- 27017:27017* 
    *environment:*
      *MONGO_INITDB_ROOT_USERNAME: **${MONGODB_USERNAME}***
      *MONGO_INITDB_ROOT_PASSWORD: **${MONGODB_PASSWORD}***

  *mongo-express:*
    *image: mongo-express*
    *restart: always*
    *ports:*
      *- 8081:8081*
    *environment:*
      *ME_CONFIG_MONGODB_URL: mongodb://**${MONGODB_USERNAME}:${MONGODB_PASSWORD}**@mongo:27017/*

*docker-compose.yml* est la nouvelle version du fichier slack renomé. Engras sont les modification apporté au fichier. aussi nous avons édité un fichier *.env* qui sert à définir les variables d'environnement dans docker. c'est dans ce fichier qu'est stocké nos identifiant de connection afin d'avoir un fichier plus sécrisé et pouvant être partagé san dévoiler nos identifiants de connexion.

*.env :*

*MONGODB_USERNAME=le_user*
*MONGODB_PASSWORD=le_password*

après lancement de la commande **docker-compose up**, nous avons obtenu plusieurs erreurs qui ont pu être corrigé après suppression de nos conteneurs avec la commande **docker-compose down**.

### 2) Mongo Compass

Mongo Compass avait déjà été installé sur l'ordinateur sur lequel nous travaillions, ainsi que mongodb. Lorsque nous faisions la connexion avec mongo compass à l'adresse *mongodb://localhost:27017*, nous n'étions pas connecté à mongodb sur docker mais plutôt au mongodb qui est installé localement. Avec un peu d'aide, nous avons compris qu'il y avait un conflit car le mongodb installé localement et celui du conteneur utilisaient le même port d'où la modification apportée au fichier docker-compose présenté ci-dessous.

*docker-compose.yml :*

*version: '3.1'*

*services:*

  *mongo:*
    *image: mongo*
    *restart: always*
    *ports:*
      **- 27018:27017** #port utilisé/port du conteneur
    *environment:*
      *MONGO_INITDB_ROOT_USERNAME: ${MONGODB_USERNAME}*
      *MONGO_INITDB_ROOT_PASSWORD: ${MONGODB_PASSWORD}*

  *mongo-express:*
    *image: mongo-express*
    *restart: always*
    *ports:*
      *- 8081:8081*
    *environment:*
      *ME_CONFIG_MONGODB_URL: mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongo:27017/*

## C) Déploiement de l'API

L'un de nous a passé énormément de temps a installer docker pendant que l'autre est arrivé très retard à cause de la grève des taxis. Mis à part cela, le plus gros soucis a été l'intallation de node : node s'installait mais la commande *npm* ne marchait pas (il s'agissait d'un problème de version et nous n'arrivions pas à installer la version la plus récente de node). Par nous ne savons quel méthode, nous avons réussi à installer et à utiliser *yarn* cependant, la commande *yarn build* n'a pas marché à cause de l'absence de npm. Finalement vers la fin de la séance, nous avons réussi à installer grâce à de l'aide, une version récente de node et npm.

Nous avons voulu exporter la variable d'environnement **MONGODB_URI** mais elle se trouvait déjà dans notre fichier d'environnement. Cependant, il nous a fallu quand même la modifier car notre url de connexion était sur le port 27018 et non 27017 dû au changement de port dans docker-compose.
