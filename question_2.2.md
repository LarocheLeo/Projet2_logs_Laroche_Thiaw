# Projet 2 : Gestion des logs 

## Source utiliser : 

- Loki : https://grafana.com/docs/loki/latest/configure/
- Promtail : https://grafana.com/docs/loki/latest/send-data/promtail/configuration/
- Graphana : https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/
- Nginx : https://nginx.org/en/docs/
- Plus vidéo youtube 
- Stack overflow, deamon off : https://stackoverflow.com/questions/25970711/what-is-the-difference-between-nginx-daemon-on-off-option



## Codage des différents conteneurs  

Pour la création de nos conteneurs, on a décidé de générer un dockerfile qui servira à la création du conteneur générateur de logs et un docker-compose pour la collecte de logs et deux autres informations la rame et le cpu utiliser. Nous avions décidé ainsi pour deux raisons : la première, c'était tout simplement la demande du cahier des charges, mais ensuite, la seconde raison est qu'en cours de route, nous avions eu l'information qu'on pouvrait utiliser docker-compose, donc, pour la simplicité, on va l'utiliser pour la seconde partie qui est bien plus complexe que le docker-compose.

### Génération des logs

Pour générer nos logs, on va créer une simple page Web avec Nginx. De plus, dans les dockerfiles suivants, on va rassembler toutes les informations en un seul fichier, même si ce n'est pas conseillé. On préfera avoir tout au même endroit.

```
# Utilise l'image officielle Nginx
FROM nginx:latest

# Crée un répertoire pour les logs (ce répertoire sera monté comme un volume)
RUN mkdir -p /var/log/nginx
```
Dans un premier temps, nous déterminerons quelle image on va utiliser et où on pourra retrouver les logs.
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
Après avoir défini certaines choses, on va maintenant faire la configuration du serveur Nginx qui va faire fonctionner notre page web. De plus, on rajoute la création des fichiers où on va retrouver les fichiers logs, mais aussi la rédirection de toute la configuration dans le chemin et le fichier qu'on souhaite.
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
Maintenant que nous avions notre serveur Nginx, on va créer notre page web qui va permettre de créer nos logs. C'est une page web staatic, donc assez simple à créer. Puis on fait comme plus haut, on fait la rédéfinition de ce qu'on fait dans un simple fichier.
```
# Expose le port 80
EXPOSE 80

# Lance Nginx
CMD ["nginx", "-g", "daemon off;"]
```
Pour finir, afin que tout fonctionne, nous envoyons tout sur le port 80. On va utiliser une commande Nginx avec l'argument -g permet de passer des directives de configuration à Nginx en ligne de commande et deamon off permet de lancer Nginx au premier plan, utile vu qu'on utilise un conteneur avec un seul service.

### Collecte de logs

Maintenant que nous avions terminé avec notre génération de logs, on va regarder comment nous avions configuré notre docker-compose et nos différents fichiers qui l'accompagnent. Pour commencer, notre solution est composé de plusieurs fichiers : 

- Le docker-compose.
- La configuration de loki en fichier yaml
- La configuration de promtail en fichier yaml
- un fichier yml pour prometheus
- un ficher yaml nommer datasource
- un autre yaml nommer dashboards
- et pour finir un ficher json nommer information_logs

Mais avant aussi d'aller plus loin. Qu'elle est notre solution ? Notre solution est « simple », on va utiliser différents logiciels. Graphana qui va nous permettre d'afficher nos différentes informations. Graphana Loki qui va pouvoir transmettre les informations qu'on récupère avec Promtail pour les envoyer à Graphana. Et donc, on utilisera Promtail comme dit précédemment pour que Loki puisse lire les données envoyées, car Loki ne pouvait pas faire cela directement après nos recherches. Pour finir, on va utiliser prometheus qui va nous permetrre de collecter des métriques en temps réel comme la consomation de ram et cpu. 

Pour cette présentation, on va présenter par chapitre nos codes. En premier, nos fichiers config avec Promtail puis Loki et nos autre fichier utiliser, nous allions terminer notre présentation avec le docker-compose.

### Promtail 

Rappel de ce qu'est Promtail : 
Promtail est un agent utilisé pour collecter et envoyer des logs à Loki, un système de gestion de logs développé par Grafana. Il fait partie de la suite d'outils d'observabilité Grafana et est souvent utilisé avec Loki pour une solution complète de journalisation.

```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

```

On va configurer le serveur avec les arguments suivants : 
- **http_listen_port: 9080 :**
Ce paramètre configure Promtail pour écouter les requêtes HTTP sur le port 9080. Cela permet de surveiller Promtail lui-même via des interfaces HTTP pour des besoins tels que la santé (health checks) ou les métriques. Ce choix de port est assez courant pour un service de monitoring qui utilise HTTP comme celui-ci (8080, 9090, ou 9080 sont souvent utilisés dans ce contexte).  
- grpc_listen_port: 0 :
Le port GRPC est désactivé (0 signifie que le service ne sera pas lancé sur un port). GRPC peut être utilisé pour des communications internes dans des systèmes distribués, mais ici, il n'est pas nécessaire puisque Promtail envoie les logs à Loki via HTTP. Le choix de ne pas utiliser GRPC est logique vu que notre architecture n'utilise que HTTP pour la communication avec Loki.

```

positions:
  filename: /tmp/positions.yaml
```

Postion : Position dans la configuration de Promtail sert à garder une trace de l'avancement de la lecture des fichiers de log.
 - filename: /tmp/positions.yaml :
Promtail suit les fichiers de logs et doit savoir où il s'est arrêté dans le fichier pour reprendre l'envoi des logs en cas de redémarrage ou d'incident. Cette section définit où Promtail va stocker cette information. Le fichier /tmp/positions.yaml est utilisé pour enregistrer les positions des logs déjà traités. Chaque fois que Promtail envoie une nouvelle entrée de log, il met à jour ce fichier avec la position dans le fichier de log, évitant ainsi l'envoi de doublons. On à choisi de mettre /tmp/, cela signifie que ces informations ne sont pas persistées à long terme (elles disparaissent lors d’un redémarrage), ce qui peut être suffisant pour un environnement comme le notre.


```
clients:
  - url: http://loki:3100/loki/api/v1/push

```
clients : La section clients liste les destinations où les logs doivent être envoyés. 
- url: http://loki:3100/loki/api/v1/push : 
Cela indique à Promtail où envoyer les logs une fois collectés. Dans notre cas, ils sont envoyés à notre serveur Loki situé à l'URL http://loki:3100 via l'API /loki/api/v1/push. Le choix de loki:3100 est typique dans un environnement Docker où loki est le nom du service Loki dans le réseau Docker et 3100 est le port par défaut utilisé par Loki pour recevoir des données.
De plus l'URL correspond à l'API Loki utilisée pour pousser les logs dans son backend.

```
scrape_configs:
  - job_name: nginx-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log  # Chemin vers vos logs Nginx
```
scrape_configs: La section scrape_configs dans la configuration de Promtail détermine les sources de logs que Promtail va collecter et envoyer à Loki. C'est une section essentielle qui définit les jobs de collecte de logs ainsi que les chemins spécifiques des fichiers de logs sur le système local.

- job_name: nginx-logs :
Cette section définit un "job" de scraping de logs, ici nommé nginx-logs. Chaque job_name peut correspondre à un ensemble de sources de logs que Promtail doit surveiller.

- static_configs :
Cela permet de définir des configurations statiques, c’est-à-dire des sources de logs dont les chemins ne changent pas. On utilise cette configuration pour les fichiers de logs situés sur le disque, comme dans le cas des logs Nginx ici.

  - targets : Permet de cibles ce qu'on veut surveiller.
    - localhost : Cela spécifie que Promtail va surveiller les logs sur la machine locale.

  - labels : Les labels sont des métadonnées associées aux logs pour mieux les catégoriser dans Loki.
    - job: nginx : Est un label job ajouté aux logs pour identifier qu'ils proviennent du job nginx. Ce label sera utilisé par Loki et Grafana pour filtrer et requêter les logs. C’est un label très utile pour regrouper les logs d’un même service.
  
    - path: /var/log/nginx/*.log : path est un chemin vers les fichiers de logs que Promtail doit surveiller. Ici, il est configuré pour suivre tous les fichiers logs se trouvant dans /var/log/nginx/ (via le masque *.log qui capture tous les fichiers se terminant par .log).


### Loki

Rappel de ce qu'est loki : 
   Loki est un système de gestion des logs open-source développé par Grafana Labs, conçu pour être une alternative légère et scalable à d'autres solutions comme ELK (Elasticsearch, Logstash, Kibana). Il est optimisé pour gérer les logs de manière efficace tout en minimisant les ressources nécessaires.

```
auth_enabled: false
```
auth_enabled: false
    Cette directive désactive l'authentification pour l'accès à l'API de Loki. Pourquoi ce choix, tout simplement cela signifie que toutes les requêtes HTTP vers Loki n'auront pas besoin d'authentification, ce qui est utile dans des environnements locaux come notre cas, mais peut représenter un risque en production. Si Loki est exposé à un réseau public, l'authentification devrait être activée pour sécuriser l'accès.

```
server:
  http_listen_port: 3100
```
Configuration serveur : 
- Configuration du serveur HTTP de Loki pour qu'il écoute sur le port 3100. Le port 3100 est le port par défaut pour Loki. Il est généralement ouvert pour permettre aux clients comme Promtail ou Grafana de se connecter et d'envoyer ou lire des logs via des API REST.
```
ingester:
  wal:
    enabled: true
    dir: /loki/wal
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 3m
  max_chunk_age: 1h
  chunk_retain_period: 30s
  max_transfer_retries: 0
```

ingester:Cette section configure l'ingestion des logs, c’est-à-dire la manière dont Loki reçoit, traite, et stocke les logs en tant que chunks.

- wal : Activation du Write Ahead Log (WAL), qui est une méthode de journalisation pour assurer la durabilité des données. Loki écrit d'abord les données dans un fichier WAL avant de les traiter.
  - dir : le répertoire où ces fichiers WAL sont stockés est /loki/wal.
- lifecycler : le lifecycler est un composant qui gère la coordination et la disponibilité des ingesters
  - ring: Le ring est une structure de données distribuée utilisée pour suivre et coordonner les ingesters dans un cluster Loki.
    - kvstore : Le Key-Value Store (magasin clé-valeur) est l'endroit où l'état du ring est stocké. Il contient des informations cruciales comme les adresses IP des ingesters, leurs responsabilités (portions de données qu'ils gèrent) et leurs statuts (actif, en panne, etc.).
      - store: inmemory : Stocke cette information en mémoire (utile pour des configurations simples ou tests).
    - replication_factor: 1 : Aucun mécanisme de réplication n'est utilisé (valeur à 1), donc les données ne sont pas redondantes.
- chunk_idle_period: 3m : Si aucun nouveau log n'arrive dans un chunk pendant 3 minutes, ce chunk sera fermé et prêt à être stocké.
- max_chunk_age: 1h : Un chunk ne peut pas dépasser 1 heure d'âge. Après cette durée, il sera forcé à être fermé, même s'il continue de recevoir des logs.
- chunk_retain_period: 30s : Après qu'un chunk est marqué comme prêt à être déplacé vers le stockage permanent, il est conservé pendant 30 secondes avant d'être supprimé de la mémoire.
- max_transfer_retries: 0 : Limite à 0 le nombre de tentatives de réessayer de transférer des données entre ingesters en cas d'échec. Cela peut indiquer une configuration simple où l'on ne prévoit pas de récupération automatique.

```
schema_config:
  configs:
    - from: 2023-01-01
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h
```
schema_config: Cette section définit la configuration du schéma utilisé par Loki pour stocker et indexer les logs.

- configs :  Dans Loki, la section configs au sein de schema_config permet de définir la structure et le comportement du schéma d'indexation des logs sur une période de temps spécifique.
  - from: 2023-01-01 : Ce schéma est en vigueur à partir du 1er janvier 2023.

  - store: boltdb-shipper : Utilise BoltDB Shipper pour gérer les index. Cela permet une meilleure gestion des index en les déplaçant vers un stockage partagé.

  - object_store: filesystem : Les objets (chunks de logs) sont stockés sur le système de fichiers local. Cela signifie que Loki ne s'appuie pas sur un service d'objets distants comme S3 ou GCS.

  - schema: v11 : Utilise la version 11 du schéma, qui est une version optimisée pour des performances accrues.

  - index : la section index fait référence à la manière dont les logs sont indexés, c'est-à-dire comment les données sont organisées pour faciliter la recherche rapide.

    - prefix: index_ : Les index des logs stockés auront pour préfixe "index_", ce qui permet de les identifier facilement.
    - period: 24h : Un nouvel index est créé tous les 24 heures.

```
storage_config:
  boltdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache
    shared_store: filesystem
  filesystem:
    directory: /loki/chunks
```
storage_config: Cette section configure le stockage des logs et des index.

- boltdb_shipper : boltdb_shipper est un composant crucial pour le stockage des index de logs lorsque Loki est configuré pour être distribué. Il permet de gérer l'indexation des logs de manière locale tout en assurant la synchronisation avec un stockage centralisé pour les déploiements distribués.

  - active_index_directory: /loki/index : Le répertoire où sont stockés les index actifs sur le disque local.
  - cache_location: /loki/cache : Emplacement du cache pour améliorer les performances de lecture des index.
  - shared_store: filesystem : Le stockage partagé est ici le système de fichiers local.

- filesystem : filesystem est une option de configuration pour le stockage des logs et des index dans un système de fichiers local. Il est souvent utilisé dans des environnements où Loki fonctionne sur une seule machine ou dans des systèmes distribués qui partagent un système de fichiers réseau.

  - directory: /loki/chunks : Répertoire sur le système de fichiers où les chunks de logs (données) sont stockés. Loki divise les logs en chunks pour faciliter le stockage et la recherche.

```
limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
```
limits_config : limits_config dans la configuration de Loki permet de définir des limites et des restrictions sur l'utilisation des ressources et le comportement de certaines fonctionnalités. Elle est cruciale pour gérer la performance de Loki dans des environnements à grande échelle, en évitant des surcharges de requêtes ou des utilisations excessives des ressources.

- enforce_metric_name: false : Désactive l'exigence d'avoir un nom de métrique spécifique pour chaque log (par défaut à false pour les logs).

- reject_old_samples: true : Active le rejet des échantillons de logs trop anciens.

- reject_old_samples_max_age: 168h : Définit l'âge maximum des logs à 168 heures (7 jours). Tout log ayant un timestamp plus vieux que 7 jours sera rejeté.
```
chunk_store_config:
  max_look_back_period: 0s
```
chunk_store_config: chunk_store_config dans la configuration de Loki définit les paramètres liés à la gestion des "chunks", c'est-à-dire des segments de données compressés utilisés pour stocker les logs de manière plus efficace. Les chunks sont des blocs de logs regroupés par périodes de temps ou par d'autres critères, et leur gestion a un impact direct sur les performances de Loki, en particulier lors de la recherche et de la récupération des logs.
- max_look_back_period: 0s : Indique que l'on peut remonter dans l'historique sans limite lors de la consultation des chunks. C'est configuré à 0s pour indiquer qu'il n'y a pas de restriction.
```
table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
```
table_manager: table_manager dans la configuration de Loki est responsable de la gestion des tables d'index utilisées pour organiser et accéder aux logs stockés. Les tables d'index aident Loki à retrouver efficacement les logs sans avoir à parcourir tous les fichiers de logs bruts. Le table_manager régule le cycle de vie des tables, notamment en ce qui concerne la rétention et la suppression des données selon les politiques définies.
- retention_deletes_enabled: false : Désactive la suppression des anciens chunks basée sur une politique de rétention.
- retention_period: 0s : Le délai de rétention des logs est défini à 0s, ce qui signifie qu'aucune rétention (suppression automatique des anciens logs) n'est activée.

### Prometheus

Avant de commencer, c'est quoi prometheuse exactement ? Prometheus est une plateforme open-source de surveillance et d'alerte, conçue pour collecter des métriques en temps réel à partir d'applications et d'infrastructures. Il stocke ces données sous forme de séries temporelles et permet de les interroger via un langage de requête, tout en offrant des fonctionnalités d'alertes basées sur les seuils définis.

maintenant le code : 

```
global: 
  scrape_interval: 15s
```
avec global on va configurer les paramètres globaux de prometheus
-  scrape_interval: 15s : Définit l'intervalle global pour la collecte des métriques. Ici, Prometheus collecte les métriques toutes les 15 secondes depuis les cibles configurées. C'est une configuration globale qui s'applique à toutes les tâches de scraping
```
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```
La section scrape_configs définit les différentes cibles ou groupes de cibles dont Prometheus doit collecter les métriques. Chaque tâche est configurée avec un job_name et une ou plusieurs cibles (targets)
- job_name : Cette tâche de scraping est nommée prometheus. Cela signifie que Prometheus va surveiller ses propres métriques (auto-surveillance).
- static_configs : Liste les cibles statiques pour cette tâche.
  - targets: ['localhost:9090'] : La cible est localhost (machine locale) sur le port 9090, qui est le port par défaut où Prometheus expose ses propres métriques.

```
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```
- job_name : Cette tâche est nommée node_exporter, ce qui indique que Prometheus va surveiller un autre service, ici le Node Exporter.
- static_configs : Liste les cibles statiques pour cette tâche.
 - targets: ['node_exporter:9100'] : La cible est le service node_exporter sur le port 9100, qui est le port par défaut pour exporter les métriques système (CPU, mémoire, etc.) à l'aide de Node Exporter.

### Datasource
#### A quoi sert une datasource ? 
Une datasource dans Grafana est une source de données externe à partir de laquelle Grafana extrait les informations pour visualiser et surveiller des métriques ou des logs. Elle permet de connecter Grafana à des systèmes comme Prometheus, Loki, ou Elasticsearch, afin de récupérer des données, les analyser et les afficher sous forme de graphiques, tableaux ou alertes dans des tableaux de bord interactifs.

Mais pourquoi faire un programme dessus ? Tout simplement ça va permettre de pouvoir configurer graphana automatiquement.  

```
apiVersion: 1
```
apiVersion: Indique la version de l'API utilisée pour la configuration de provisionnement des datasources. Ici, c'est la version 1.

```
datasources:
```
datasources: Cette section contient la définition de plusieurs sources de données. Chaque bloc sous cette section représente une datasource à configurer dans Grafana.

```
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090  # Vérifie que l'URL correspond à ton conteneur Prometheus
    isDefault: false
    editable: true
```
- name: Prometheus : Nom de la datasource. Ce nom apparaîtra dans Grafana lors de la sélection de cette source de données.
- type: prometheus : Spécifie que la datasource est de type Prometheus, qui est un outil de monitoring et d'alerte utilisé pour collecter et analyser des métriques.
- access: proxy : Définit comment Grafana accède à cette datasource. En mode proxy, Grafana agit comme intermédiaire pour faire les requêtes depuis le serveur vers la datasource.
- url: http://prometheus:9090 : URL à laquelle Grafana peut accéder à l'instance Prometheus. Elle doit correspondre à l'adresse du conteneur Prometheus.
- isDefault: false : Indique que cette datasource n'est pas définie comme la source par défaut.
- editable: true : Permet de rendre cette datasource modifiable via l'interface utilisateur de Grafana.

```
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100 
    isDefault: true
    jsonData:
      maxLines: 1000
```
- name: Loki : Nom de la datasource. Elle est définie pour Loki, un système de gestion des logs développé par Grafana Labs.
- type: loki : Spécifie que la datasource est de type Loki, utilisée pour la collecte et la recherche de logs.
- access: proxy : Comme pour Prometheus, Grafana agit en mode proxy pour accéder à Loki.
- url: http://loki:3100 : URL à laquelle Grafana accède au service Loki pour récupérer les logs. Ici, il s'agit de l'adresse du conteneur Loki.
- isDefault: true : Loki est défini comme la datasource par défaut, ce qui signifie que les requêtes qui ne spécifient pas de datasource utiliseront Loki par défaut.
- jsonData.maxLines: 1000 : Ce paramètre limite le nombre maximum de lignes de logs que Loki retourne dans une requête, ici fixé à 1000 lignes.

Avant de finir, les deux datasource ne sont pas toutes les deux par défaut pour une raison. Graphana accepte seulement une source par défaut. Donc ici, on met celle qui génére les logs par défaut car c'est la plus importante

### Dashboards

Comme précedent, ici on va configurer le dashboard. Ca va nous permettre d'automatiser la création de ce dernier lors de la création des conteneurs. On vera dans le chapitre d'après commant créer notre dashboard au pixel près. 

```
apiVersion: 1
```
apiVersion: Indique la version de l'API utilisée pour le provisionnement. Ici, c’est la version 1, qui spécifie le format du fichier de configuration pour les dashboards dans Grafana
```
providers:
  - name: "Projet_logs"
    folder: ""
    type: "file"
    options:
      path: /etc/grafana/provisioning/dashboards  
```
providers: : Cette section définit les "fournisseurs" de dashboards, c'est-à-dire comment Grafana va charger les dashboards à partir de fichiers ou d'autres sources externes.
- name: "Projet_logs" : C’est le nom du fournisseur de dashboards. Ce nom peut être utilisé pour identifier cette configuration dans Grafana. Ici, il est appelé "Projet_logs", ce qui reflète probablement le projet en cours.
- folder: "" : Définit le dossier où les dashboards seront stockés dans Grafana. Ici, un dossier vide ("") signifie que les dashboards ne seront pas classés dans un dossier spécifique dans l'interface de Grafana et apparaîtront dans la racine.
- type: "file" : Spécifie que les dashboards seront importés à partir de fichiers locaux. Le type "file" signifie que Grafana ira chercher les fichiers JSON de dashboards dans le système de fichiers.
- options: : Cette section contient des options supplémentaires pour le fournisseur de dashboards.
- path: /etc/grafana/provisioning/dashboards : Définit le chemin où Grafana va chercher les fichiers JSON de dashboards. Ici, il s’agit du dossier /etc/grafana/provisioning/dashboards. Grafana va parcourir ce dossier pour charger tous les dashboards au démarrage ou lors du rechargement.


### Information_logs

Maintenant qu'on a réaliser l'automatisation des datasources et la création du dashboard, on va créer le code qui va créer le dashboard où va afficher tout nos informations.
Donc ce fichier JSON représentera un dashboard Grafana du nom d'"Information et logs", avec plusieurs panneaux de visualisation pour suivre l'état du système. Voici la description de ce fochier :
```
{
  "id": null,
  "uid": "cLV5GDCkz",
  "title": "Information et logs",
  "tags": [],
  "timezone": "browser",
  "editable": true,
  "graphTooltip": 1,

```
1 Les Informations générales : 

- id : null (sera généré automatiquement lorsque le dashboard sera créé).
- uid : cLV5GDCkz (identifiant unique du dashboard utilisé dans l'URL).
- title : "Information et logs" (le titre du dashboard).
- tags : Liste vide, il n'y a pas de tags associés à ce dashboard.
- timezone : "browser" (le fuseau horaire est défini selon le navigateur de l'utilisateur).
- editable : true (le dashboard est modifiable).
- graphTooltip : 1 (affichage d'un tooltip lorsqu'on survole un graphique).
```

  "panels": [
```
2 Panneaux (Panels)
Le dashboard contient 3 panneaux, chacun ayant des objectifs spécifiques.
```
    {
      "type": "logs",
      "title": "Logs Nginx",
      "gridPos": {
        "h": 9,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "datasource": "Loki",
      "targets": [
        {
          "expr": "{job=\"nginx\"}",
          "refId": "A"
        }
      ],
      "options": {
        "showLabels": true,
        "wrapLogMessage": true,
        "sortOrder": "Descending"
      },
      "fieldConfig": {
        "defaults": {
          "custom": {}
        },
        "overrides": []
      }
    },
```
Panneau 1 : Logs Nginx
- type : "logs" (type de panneau pour afficher des logs).
- title : "Logs Nginx" (titre du panneau).
- gridPos : Position du panneau sur la grille (hauteur de 9, largeur de 24, position x=0, y=0).
- datasource : "Loki" (source de données Loki utilisée pour récupérer les logs).
- targets :
  - expr : "{job=\"nginx\"}" (expression de recherche pour les logs du job "nginx").
  - refId : "A" (identifiant de la requête).
- options :
  - showLabels: true (affiche les étiquettes des logs).
  - wrapLogMessage: true (ajoute un retour à la ligne dans le message de log si nécessaire).
  - sortOrder: "Descending" (les logs seront triés par ordre décroissant).
- fieldConfig : Cette section définit la configuration spécifique des champs de données dans le panneau. Elle permet de personnaliser l'affichage des valeurs et des unités dans les graphiques ou autres types de panneaux. 
- defaults : Cette sous-section permet de définir les valeurs par défaut de la configuration des champs.
- custom: {} : indique qu'il n'y a aucune configuration personnalisée par défaut. Si des personnalisations étaient nécessaires, elles seraient ajoutées dans ce bloc sous forme d'options spécifiques, mais ici ce champ est vide, ce qui signifie qu'aucune personnalisation n'est appliquée par défaut.
- overrides : Cette sous-section permet de définir des personnalisations spécifiques pour certains champs particuliers, en fonction de leurs caractéristiques ou de leurs valeurs.
Ici, la liste est vide ([]), ce qui signifie qu'il n'y a pas d'overrides (modifications spécifiques pour des champs particuliers) appliquées. 

```
    {
      "type": "graph",
      "title": "Utilisation de la RAM",
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 0,
        "y": 9
      },
      "datasource": "Prometheus",
      "targets": [
        {
          "expr": "node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes",
          "refId": "A"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bytes",
          "thresholds": {
            "steps": [
              { "color": "green", "value": null },
              { "color": "orange", "value": 0.75 },
              { "color": "red", "value": 0.9 }
            ]
          }
        }
      }
    },
```
Panneau 2 : Utilisation de la RAM
- type : "graph" (type de panneau graphique).
- title : "Utilisation de la RAM" (titre du panneau).
- gridPos : Position du panneau (hauteur de 9, largeur de 12, position x=0, y=9).
- datasource : "Prometheus" (source de données Prometheus pour les métriques système).
- targets :
  - expr : "node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes" (expression Prometheus pour calculer l'utilisation de la RAM en fonction de la mémoire totale et disponible).
  - refId : "A" (identifiant de la requête).
- fieldConfig : Cette section définit la configuration spécifique des champs de données dans le panneau. Elle permet de personnaliser l'affichage des valeurs et des unités dans les graphiques ou autres types de panneaux. 
- defaults : Cette sous-section permet de définir les valeurs par défaut de la configuration des champs.
- unit : "bytes" (l'unité utilisée est les octets).
- thresholds permet de définir des seuils de valeurs qui modifient l'apparence d'un graphique ou d'un tableau dans Grafana en fonction des valeurs mesurées. C'est une fonctionnalité qui permet d'appliquer une coloration conditionnelle sur un graphique ou un indicateur en fonction des valeurs de données.
- steps : Ce tableau contient une série de seuils, chaque seuil ayant deux propriétés principales :
  - color : La couleur qui sera appliquée lorsque la valeur atteindra ce seuil.
  - value : La valeur à partir de laquelle la couleur change. Cela détermine quand le seuil est atteint.
    - green: Pas de seuil défini, valeur initiale.
    - orange: Seuil à 75% d'utilisation de la RAM.
    - red: Seuil à 90% d'utilisation de la RAM.

```
    {
      "type": "graph",
      "title": "Utilisation du CPU",
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 12,
        "y": 9
      },
      "datasource": "Prometheus",
      "targets": [
        {
          "expr": "100 - (avg by (instance) (irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
          "refId": "A"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "thresholds": {
            "steps": [
              { "color": "green", "value": null },
              { "color": "orange", "value": 75 },
              { "color": "red", "value": 90 }
            ]
          }
        }
      }
    }
  ],
```
Panneau 3 : Utilisation du CPU
- type : "graph" (type de panneau graphique).
- title : "Utilisation du CPU" (titre du panneau).
- gridPos : Position du panneau (hauteur de 9, largeur de 12, position x=12, y=9).
- datasource : "Prometheus" (source de données Prometheus).
- targets :
  - expr : "100 - (avg by (instance) (irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)" (expression Prometheus pour calculer le pourcentage d'utilisation du CPU en fonction du temps d'inactivité).
  - refId : "A" (identifiant de la requête).
- fieldConfig : Cette section définit la configuration spécifique des champs de données dans le panneau. Elle permet de personnaliser l'affichage des valeurs et des unités dans les graphiques ou autres types de panneaux. 
- defaults : Cette sous-section permet de définir les valeurs par défaut de la configuration des champs.
- unit : "bytes" (l'unité utilisée est les octets).
- unit : "percent" (l'unité utilisée est le pourcentage).
- thresholds permet de définir des seuils de valeurs qui modifient l'apparence d'un graphique ou d'un tableau dans Grafana en fonction des valeurs mesurées. C'est une fonctionnalité qui permet d'appliquer une coloration conditionnelle sur un graphique ou un indicateur en fonction des valeurs de données.
- steps : Ce tableau contient une série de seuils, chaque seuil ayant deux propriétés principales :
  - color : La couleur qui sera appliquée lorsque la valeur atteindra ce seuil.
  - value : La valeur à partir de laquelle la couleur change. Cela détermine quand le seuil est atteint.
    - green: Pas de seuil défini, valeur initiale.
    - orange: Seuil à 75% d'utilisation du CPU.
    - red: Seuil à 90% d'utilisation du CPU.


```
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": ["5s", "10s", "30s", "1m", "5m"]
  },
```
3. Temps et Rafraîchissement
- time : Définit la plage de temps visible sur le dashboard (from: "now-6h", to: "now") — il affiche les données des 6 dernières heures.
- timepicker : Définit les intervalles de rafraîchissement disponibles pour le timepicker (options comme 5s, 10s, 30s, 1m, 5m).
  - refresh : Le dashboard se rafraîchira automatiquement toutes les 5 secondes.

```
  "templating": {
    "list": []
  },
  "annotations": {
    "list": []
  },
  "refresh": "5s",
  "schemaVersion": 17,
  "version": 1,
  "links": []
}
```
4. Autres Paramètres
- templating : Liste vide, il n'y a pas de variables définies pour ce dashboard.
- annotations : Liste vide, il n'y a pas d'annotations définies.
- schemaVersion : Version du schéma de configuration (ici 17).
- version : Version du dashboard (ici 1).
- links : Liste vide, il n'y a pas de liens associés au dashboard.

### Docker-compose : 

Maintenant que nous avions vu nos configurations, nous pouvions aller découvrir ce que nous avions configuré avec Docker-compose. Afin d'avoir un maximun de facilité et autres, une grande partie des créations d'image et de conteneur se trouve dans. 
```
version: '3'
```
Version 3 du format Docker Compose. Cette version est largement utilisée et offre un bon équilibre entre simplicité et fonctionnalités pour définir des services conteneurisés.
```
services:
  loki:
    image: grafana/loki:2.9.2
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - my_network
```
services : La section services: dans un fichier Docker Compose est utilisée pour définir et configurer plusieurs services (ou conteneurs Docker) qui vont tourner ensemble dans un même environnement orchestré. Chaque service représente un conteneur Docker indépendant qui peut être configuré, connecté aux autres services, et exposé avec des ports et volumes.
- Loki : création du contener loki pour pouvoir l'utiliser.
  - Image : Utilisation la version 2.9.2 de l'image officielle de Loki, qui est un agrégateur de logs conçu pour être efficace et léger par rapport à des solutions comme ELK.
  - Ports : Mappe le port 3100 du conteneur au port 3100 de l'hôte. C'est le port par défaut pour accéder à Loki via HTTP.
  - Command : Spécifie l'emplacement du fichier de configuration personnalisé de Loki, ici /etc/loki/local-config.yaml.
  - Networks : Le conteneur Loki est connecté à un réseau personnalisé appelé my_network, ce qui permet à tous les services de communiquer entre eux à l'intérieur du même réseau isolé.
```
  promtail:
    image: grafana/promtail:2.9.2
    volumes:
      - ./promtail-config.yaml:/etc/promtail/config.yml
      - /var/log/nginx/:/var/log/nginx
    command: -config.file=/etc/promtail/config.yml
    networks:
      - my_network
```
- promtail : création du contener promtail pour pouvoir l'utiliser.
  - Image : Utilisation la version 2.9.2 de Promtail, qui est l'agent qui collecte les logs à partir de fichiers (comme ceux de Nginx), les étiquette, et les envoie à Loki.
  - Volumes :
    - ./promtail-config.yaml:/etc/promtail/config.yml : Le fichier de configuration promtail-config.yaml est monté à l'intérieur du conteneur pour indiquer à Promtail où chercher les logs et comment les traiter.
    - /var/log/nginx/:/var/log/nginx : Le répertoire /var/log/nginx/ est monté pour que Promtail puisse accéder aux logs générés par Nginx et les envoyer à Loki.
  - Command : Indique à Promtail d'utiliser le fichier de configuration monté.
  - Networks : Connecté au même réseau personnalisé my_network que Loki et les autres services, permettant une communication sans problème avec Loki via son URL interne.
```
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./provisioning/dashboards:/etc/grafana/provisioning/dashboards  # Modifier le chemin pour éviter des conflits
      - ./provisioning/dashboards.yaml:/etc/grafana/provisioning/dashboards/dashboards.yaml
      - ./provisioning/datasource.yaml:/etc/grafana/provisioning/datasources/datasource.yaml

    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
    networks:
      - my_network
```
- grafana : création du contener grafana pour pouvoir l'utiliser.
  - Image : Utilisation l'image la plus récente de Grafana, un outil de visualisation des données. Grafana permet de visualiser les logs envoyés à Loki via des tableaux de bord.
  - Ports : Le port 3000 est mappé de l'intérieur du conteneur vers l'hôte, permettant d'accéder à l'interface de Grafana via localhost:3000.
  - Volumes : Le volume grafana-data est utilisé pour stocker les données persistantes de Grafana (paramètres, tableaux de bord, etc.).
  - Environment : Définit le mot de passe administrateur initial avec la variable GF_SECURITY_ADMIN_PASSWORD. Ici, le mot de passe est admin (par défaut).
  - Networks : Connecté à my_network, permettant à Grafana de communiquer avec Loki et Promtail directement via le réseau Docker.
```
 prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - my_network
```
- image : Utilise l'image prom/prometheus:latest pour Prometheus, l'outil de collecte de métriques.
- volumes :
  - Monte le fichier prometheus.yml pour fournir la configuration de Prometheus.
  - Monte un volume pour stocker les données de Prometheus de façon persistante.
- ports : Mappe le port 9090 du conteneur au port 9090 de l'hôte.
- networks : Prometheus fait partie du réseau my_network.
```
  node_exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    networks:
      - my_network
```
- image : Utilise l'image prom/node-exporter:latest pour exporter des métriques sur l'état du système.
- ports : Mappe le port 9100 du conteneur au port 9100 de l'hôte.
- networks : Node Exporter fait partie du réseau my_network.
```
  nginx:  # Nouveau service pour Nginx
    build: D:/Cours_3e_annee/SAE/projet2/version_final  # Remplace par le chemin vers ton Dockerfile Nginx
    ports:
      - "80:80"  # Mappe le port 80 du conteneur au port 80 de l'hôte
    volumes:
      - /var/log/nginx/:/var/log/nginx  # Monte le volume pour les logs Nginx
    networks:
      - my_network  # Connecte Nginx au réseau personnalisé
```
- nginx : création du contener nginx pour pouvoir l'utiliser.
  - Build : Indique que Nginx sera construit à partir d'un Dockerfile situé dans le répertoire spécifié (dans cet exemple, un chemin local sur ton système).
  - Ports : Le port 80 est exposé, permettant à Nginx de répondre aux requêtes HTTP via localhost:80.
  - Volumes : Monte le répertoire des logs Nginx (/var/log/nginx/) à la fois sur l'hôte et dans le conteneur, permettant à Promtail de lire les logs générés par Nginx.
  - Networks : Connecté à my_network pour que Nginx puisse interagir avec d'autres services sur le réseau, comme Promtail pour le transfert des logs.
```
volumes:
  loki-data:
  grafana-data:
  prometheus-data:
```
Déclare les volumes Docker utilisés pour stocker les données persistantes des services Loki, Grafana, et Prometheus. Ces volumes permettent de préserver les données même si les conteneurs sont arrêtés ou supprimés.

```
networks:
  my_network:   
```
networks : La section networks dans un fichier Docker Compose définit les réseaux que les conteneurs utiliseront pour communiquer entre eux.
- my_network: déclaration du réseau pour que les conteneurs communiquent entre eux. 

### Remarque de fin : 

Après avoir réalisé ce projet, nous avions bien sûr quelques idées d'amélioration de ce qu'on pourrait faire en plus. Ces « améliorations » sont surtout des fonctionnalités qu'on n'a pas pu mettre par manque de temps. Ces idées étaient de mettre plus d'information sur le système host, comme les ressources CPU utilisées, les paquets envoyés ou autre information. 

Mais comme vous le voyez pendant les heures en plus données, nous avions rajouter ces fonctionnalités. Même si on avait voulu en plus mettre le stockage utilisé sur le disque dur. Nous n’avions pas réussi à le faire correctement. Donc nous l'avions simplement retiré cette option.

Nous vous remercions d'avoir lu ce Markdown jusqu'au bout.

