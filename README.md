
# Configuration Docker pour Nginx, Loki, Promtail et Grafana

Ce projet utilise Docker et Docker Compose pour déployer une pile complète de monitoring avec Nginx, Loki, Promtail et Grafana.

## Prérequis

- Docker installé sur votre machine
- Docker Compose installé

## Structure du projet

- `docker-compose.yml` : Fichier Docker Compose qui orchestre les différents services.
- `Dockerfile` : Fichier pour construire l'image Nginx personnalisée.
- `promtail-config.yaml` : Fichier de configuration pour Promtail.
- `loki-config.yaml` : Fichier de configuration pour Loki.

## Étapes pour utiliser ce projet

1. **Cloner le projet** : Téléchargez ou clonez ce répertoire sur votre machine.

2. **Configurer les fichiers** :
   - Vérifiez et adaptez les chemins si nécessaire dans le fichier `docker-compose.yml`.
   - Placez vos fichiers de configuration :
     - `promtail-config.yaml` pour Promtail.
     - `loki-config.yaml` pour Loki.

3. **Construire et lancer les services** :
   - Ouvrez un terminal dans le répertoire où se trouve votre `docker-compose.yml`.
   - Exécutez la commande suivante pour construire et démarrer tous les services :
     ```bash
     docker-compose up --build
     ```

4. **Accéder aux services** :
   - **Grafana** : Accédez à l'interface web via `http://localhost:3000`. Le mot de passe admin par défaut est `admin`.
   - **Nginx** : Votre serveur Nginx est accessible via `http://localhost:80`.
   - **Loki** : Loki écoute sur le port `3100` et est utilisé pour stocker les logs.
   - **Promtail** : Collecte les logs de Nginx et les pousse vers Loki.

## Personnalisation

- Si vous souhaitez changer la configuration de Nginx, modifiez le fichier `Dockerfile` et la configuration par défaut de Nginx.
- Vous pouvez également modifier les configurations de Loki et Promtail dans leurs fichiers respectifs.

## Arrêter les services

Pour arrêter et supprimer les conteneurs, exécutez la commande suivante :

```bash
docker-compose down
```

Pour  effacer tous les conteneurs et volumes, executez ce script dans le répertoire des conteneurs:

```bash
bash purge.sh
```

## Volumes et données persistantes

Les volumes Docker sont utilisés pour persister les données de Grafana et Loki, afin que vos logs et configurations soient conservés entre les redémarrages.
