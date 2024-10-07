# Utilise l'image officielle Nginx
FROM nginx:latest

# Crée un répertoire pour les logs (ce répertoire sera monté comme un volume)
RUN mkdir -p /var/log/nginx

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

# Expose le port 80
EXPOSE 80

# Déclare un volume pour les logs
VOLUME ["/var/log/nginx"]

# Lance Nginx
CMD ["nginx", "-g", "daemon off;"]
