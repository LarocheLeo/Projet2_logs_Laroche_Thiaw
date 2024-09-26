# LarocheLeo-SAE52_Collecte_traitement_des_logs

# Differents choix de logiciel de collecte de logs possibles 

## Introduction
Ce document fera le tours des differents outils de collecte de données que nous avons rencontré, quelle utilité nous leur avons trouvé, et enfin quelle a été notre choix et pourquoi.

## Outils 

### Elastic Stack

Dispose des outils Logstack/Beats pour la collecte et l'envoi de données. Dispose également d'Elasticsearch comme moteur de recherche, ce qui facilise sont utilisation. Très populaire dans le domaine, Elastic Stack est muni d'une grande communauté prête à accompgner les nouveux utilisateurs. Ceci dit, l'installation est assez compliqué à mettre en place

Concernant le prix, gratuit en version open-source, mais Elastic propose des fonctionnalités premium comme la sécurité avancée, les alertes et les fonctionnalités cloud.

### Graylog

GrayLog permet de centraliser les logs issus de plusieurs environnements grâce à son support de différents protocoles comme Syslog et GELF. L'interface est simple à utiliser et facile à prendre en main, ce qui le rend plus accessible que l'Elastic Stack. Il dispose également d'une communauté active pour aider les nouveaux utilisateurs. L'installation est plus facile que celle d'Elastic Stack.

La version open-source est gratuite, mais des forfaits payants existent pour les entreprises, avec des fonctionnalités avancées comme le clustering, la gestion d’alertes et le support prioritaire. Les prix varient selon la taille de l’infrastructure et les besoins spécifiques.



### Fluentd

Fluentd est un collecteur de logs très flexible qui permet de centraliser des données provenant de plusieurs sources et de les envoyer vers différents systèmes. Il est relativement facile à installer et à configurer pour des scénarios de base. Cependant, pour des configurations avancées, il peut être nécessaire d’avoir des connaissances plus approfondies. Fluentd ne propose pas de dashboard intégré, donc il doit être utilisé avec des outils comme Grafana ou Elasticsearch pour la visualisation.

Fluentd est entièrement gratuit et open-source, sans versions premium ni frais supplémentaires. Il est soutenu par une grande communauté, offrant une solution économique mais robuste pour les entreprises.

### Filebeat

Filebeat, faisant partie de l’Elastic Stack, collecte et envoie les logs vers Logstash ou Elasticsearch. Il est léger et simple à configurer pour des cas d’utilisation de base, mais des configurations avancées peuvent devenir plus difficiles à mettre en œuvre. Comme Fluentd, il n’a pas de dashboard intégré et doit être utilisé avec Kibana ou un autre outil de visualisation.

Filebeat est gratuit en version open-source, mais s’inscrit dans l’écosystème Elastic qui propose des versions payantes pour des fonctionnalités supplémentaires telles que le monitoring avancé ou des intégrations spécifiques.

### Promtail
### New Relic
### Splunk
### Graphana
### Solarwinds


## Source utilisées : 


