# Dockerize Django App with Nginx

Il s'agit de faire plusieurs containers :
- un container django (avec bdd sqllite)
- un container Nginx (un fichier de conf suffit, pas besoin de télécharger Nginx pour le dev, le fichier de conf sera poussé dans le container Nginx)
- un Volume contenant les fichiers static

Docker Compose pour ordonnancer les containers

- serveur Nginx écoute sur le port 80, (il sert de reverse proxy), puis il renvoie vers Django sur le port 8000 (Gunicorn retourne les pages à Nginx) et enfin Nginx sert les pages avec les fichiers static.

## explications

Le script entrypoint permet de lancer django dans le container, le script :
- migre la bdd
- Collecte les fichiers static
- execute gunicorn pour lancer le serveur

le script permet d'initialiser le projet Django en mode production. En effet le Dockerfile ne fait que télécharger les "requirements" et copier le projet dans le container.

- Dockercompose :
celui-ci va créer 2 services :
- un service que l'on a nommé "django_gunicorn", qui prend le Dockerfile présent dans répertoire, donc il build et lance le projet Django , du port 8000 vers le port 8000 et il récupére le fichier .env qu'il pourra injecter dans le container django_gunicorn et il servira du volume STATIC monté.
- un service nginx, qui dépend de django_gunicorn, qui va faire le build à partir du dockerfile dans le répertoire nginx.
Dockercompose :
- crée un service
- Build le dockerfile
- utilise un volume si nécessaire
- utilise un fichier .env si nécessaire
- listen sur un port

- Nginx
Configuration :
renomme le service django_unicorn en serveur "django" (dockecompose crée un réseau pour le container django_unicorn et donc on peut utiliser ce nom, !!! avec Docker il faut utiliser le nom de l'instance)

- fichiers static
Comment sont-ils servis ?
Comment docker sait qu'il doit utiliser le répetoire /static/ dans le container django pour le monter comme volume ?
1/Django collecte tous les fichiers statiques et les met dans le répertoire que l'on a défini dans SETTINGS au niveau de STATIC_ROOT
2/ Avec Docker-compose on crée un volume, qui sera mappé sur le répertoire du conteneur django
3/ Avec Docker-compose Ngnix aura le volume monté là où on lui indique


- Run APP :
docker-compose up --build
(pour voir le build)

docker-compose down

docker volume prune
(pour supprimer le volume, attention les images doivent être supprimées avant car sinon elles sont utilisées)

## Docker - Les bonnes pratiques :
https://www.youtube.com/watch?v=8vXoMqWgbQQ

▬▬▬▬▬▬ T I M E S T A M P S ⏰  ▬▬▬▬▬▬
- 0:00 - Intro
- 0:34 - BP 1: Use official and verified Docker Images as Base Image
- 1:13 - BP 2: Use Specific Docker Image Versions
- 2:12 - BP 3: Use Small-Sized Official Images
- 4:35 - BP 4: Optimize Caching Image Layers
- 10:09- BP 5: Use .dockerignore file
- 10:55 - BP 6: Make use of Multi-Stage Builds
- 14:15 - BP 7: Use the Least Privileged User 
- 16:06 - BP 8: Scan your Images for Security Vulnerabilities
- 17:50 - Wrap Up

## Questions

Attention aux versions utilisées dans le projet
A t-on besoin d'un dockerignore ? sinon db.sqlite3 va être copiée ainsi que les binaires python...

Les commandes `RUN` et `CMD` dans un fichier Dockerfile ont des rôles distincts et s'utilisent dans des contextes différents. Voici les différences principales :

### 1. **Commande `RUN` :**

- La commande `RUN` est utilisée pour exécuter des instructions pendant la **construction** de l'image Docker.
- Chaque commande `RUN` crée une nouvelle couche dans l'image Docker.
- Typiquement, elle est utilisée pour installer des packages, copier des fichiers, ou préparer l'environnement. Par exemple :

```dockerfile
RUN apt-get update && apt-get install -y nginx
```

- Une fois l'image construite, les commandes `RUN` ne sont plus exécutées lorsque le conteneur est lancé.

### 2. **Commande `CMD` :**

- La commande `CMD` spécifie l'instruction par défaut à exécuter lorsque le conteneur est **démarré**.
- Elle ne s’exécute **qu’au démarrage** du conteneur et n'est pas utilisée lors de la création de l'image.
- Contrairement à `RUN`, `CMD` n'est pas utilisée pour préparer l'image, mais pour définir ce que le conteneur doit faire par défaut, par exemple :

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

- Si une commande est passée lors du démarrage du conteneur (`docker run`), elle remplacera la commande définie par `CMD`.

### Résumé des différences :
- **`RUN`** est exécuté lors de la **construction** de l'image, tandis que **`CMD`** est exécuté lorsque le conteneur est **démarré**.
- **`RUN`** est utilisé pour installer des dépendances ou préparer l'environnement dans l'image.
- **`CMD`** définit la commande par défaut à exécuter dans le conteneur.

En résumé, `RUN` construit l'image, tandis que `CMD` définit ce que fait le conteneur lorsqu'il démarre.
