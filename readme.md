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

## Questions

Attention aux versions utilisées dans le projet
A t-on besoin d'un dockerignore ? sinon db.sqlite3 va être copiée ainsi que les binaires python...
