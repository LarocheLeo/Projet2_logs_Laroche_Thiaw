# Utilise l'image officielle Nginx
FROM nginx:latest

# Copie la configuration Nginx
COPY default.conf /etc/nginx/conf.d/default.conf

# Copie la page HTML dans le répertoire par défaut de Nginx
COPY index.html /usr/share/nginx/html/index.html

# Expose le port 80
# mappage dans docker-compose.yml
EXPOSE 80 

# Lance Nginx
# -g permet de passer des directives de configuration à Nginx en ligne de commande
# deamon off permet de lancer Nginx au premier plan, utile vu qu'on utilise un conteneur avec un service
CMD ["nginx", "-g", "daemon off;"]
