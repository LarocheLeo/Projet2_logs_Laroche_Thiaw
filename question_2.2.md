# LarocheLeo-SAE52_Collecte_traitement_des_logs


## Source utiliser pour la programmation : 

- Stack overflow, deamon off : https://stackoverflow.com/questions/25970711/what-is-the-difference-between-nginx-daemon-on-off-option

## Programme 

Pour la création de nos conteneurs, on à décider de générer un dockerfile qui servira à la création du conteneur générateur de logs et un docker-compose pour la collectre de logs. Nous avions décider ainsi pour deux raison: la premère, c'était tout simplement la demande du cahier des charges mais ensuite la seconde raison est qu'en cours de route, nous avions eu l'information qu'on pouvrait utilisé docker-compose, donc pour la simplicité, on va l'utiliser pour la seconde partie qui est bien plus complex que le docker-compose.    

### Génération des logs 

Pour générer nos logs, on va créer une simple page web avec Nginx. De plus dans les dockerfiles suivant, on va rassembler tout les informations en un seul fichier même si ce n'est pas conseiller. on préfera avoir tout au même endroit.

```
# Utilise l'image officielle Nginx
FROM nginx:latest

# Crée un répertoire pour les logs (ce répertoire sera monté comme un volume)
RUN mkdir -p /var/log/nginx
```
Dans un premier temps, nous définition qu'elle image on va utiliser et où on pourra retrouver les logs 
```
# Copie la configuration Nginx avec un logging dans le dossier créé
RUN echo 'server { \
    listen 80; \
    server_name localhost; \
    root /usr/share/nginx/html; \
    index index.html; \
    location / { \
        try_files $uri $uri/ =404; \
    } \
    access_log /var/log/nginx/access.log; \
    error_log /var/log/nginx/error.log; \
}' > /etc/nginx/conf.d/default.conf
```
après avoir définit certaines choses, on va maitenant faire la configuration du serveur Nginx qui va faire fonctionner notre page web, de plus on rajoute la création des fichiers où on va retrouver les fichiers logs mais aussi la rédirection de toute la configuration dans le chemin et le fichier qu'on souhaite.
```
# Crée une page HTML simple
RUN echo '<!DOCTYPE html>\
<html lang="fr">\
<head>\
    <meta charset="UTF-8">\
    <meta http-equiv="X-UA-Compatible" content="IE=edge">\
    <meta name="viewport" content="width=device-width, initial-scale=1.0">\
    <title>Nginx Docker</title>\
</head>\
<body>\
    <h1>Bienvenue sur Nginx avec Docker !</h1>\
    <p>Ceci est une page simple pour générer des logs.</p>\
</body>\
</html>' > /usr/share/nginx/html/index.html
```
Maintenant que nous avions notre server nginx, on va créer notre page web qui va permettre de créer nos logs, c'est une page web staatic donc assez simple à créer. puid on fait comme plus eu, on fait la rédériction de ce qu'on fait dans un simple fichier.
```
# Expose le port 80
EXPOSE 80

# Lance Nginx
CMD ["nginx", "-g", "daemon off;"]
```
Pour finir afin que tout fonctionne, nous envoyont tout sur le port 80. On va utiliser une commande nginx avec l'arguement -g permet de passer des directives de configuration à Nginx en ligne de commande et deamon off permet de lancer Nginx au premier plan, utile vu qu'on utilise un conteneur avec un seul service.
t 

### Collecte de logs

Maintenant que nous avions terminer avec notre génération de logs, on va regarder comment nous avions configurer notre docker-compose et nos différents fichier qui l'accompagnge. Pour commencer, notre solution est composé de 3 fichier : 
    - Le docker-compose 
    - Et 2 fichier config en yaml 

Mais avant aussi d'aller plus loin. Qu'elle est notre solution ? 
Notre solution est "simple", on va utilise 3 logiciels. Graphana qui va nous permettre d'afficher nos différentes informations. Graphana loki qui va pouvoir transmettre les informations qu'on récupère avec promtail pour l'envoyer à Graphana Et donc on utilisera promtail comme dit précédement pour que loki puisse lire les données envoyer car loki ne pouvait pas faire sa directement après nos recherches. 

Pour cette présentation, on va présenter par chapitre nos codes. En premier nos fichiers config avec promtail puis loki, enfin nous allions terminer notre présentation avec le docker-compose.


#### Promtail 





