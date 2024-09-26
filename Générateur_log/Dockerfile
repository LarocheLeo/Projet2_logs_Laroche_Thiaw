# Utilise l'image officielle Nginx
FROM nginx:latest

# Copie la configuration Nginx
COPY default.conf /etc/nginx/conf.d/default.conf

# Copie la page HTML dans le répertoire par défaut de Nginx
COPY index.html /usr/share/nginx/html/index.html

# Expose le port 80
EXPOSE 80

# Lance Nginx
CMD ["nginx", "-g", "daemon off;"]
