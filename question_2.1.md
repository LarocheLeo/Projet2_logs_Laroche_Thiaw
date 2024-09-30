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


### New Relic

New Relic offre une plateforme complète pour le monitoring et l'analyse en temps réel des logs. L’interface intuitive et les puissantes fonctionnalités de visualisation permettent de créer des dashboards dynamiques. La solution est idéale pour surveiller des environnements cloud à grande échelle, avec des capacités d’alertes et de suivi des performances en temps réel.

New Relic est une solution payante, avec une tarification basée sur la quantité de données collectées et le nombre d'utilisateurs. Les plans incluent des fonctionnalités de base pour le monitoring, mais des options plus avancées (comme les alertes personnalisées et le suivi détaillé des logs) sont disponibles dans des forfaits supérieurs.


### Splunk

Splunk est l’une des solutions les plus puissantes pour la gestion des logs, capable de traiter de grands volumes de données provenant de multiples sources. L'installation est plus complexe, mais il offre une des meilleures performances pour l’analyse et la gestion des logs à grande échelle. Les dashboards sont hautement personnalisables et permettent une surveillance en temps réel.

Splunk est une solution payante avec une tarification basée sur le volume de données ingérées. Les forfaits varient selon les fonctionnalités et les besoins de l'entreprise, allant de la gestion de base des logs jusqu’à des options avancées pour l'analyse des données et la sécurité.

### Graphana

Grafana, lorsqu’il est utilisé avec Loki, offre une solution complète pour la centralisation et la visualisation des logs. L’avantage de cette combinaison est la capacité de Grafana à afficher non seulement les logs, mais aussi d'autres métriques dans des dashboards très personnalisés. Facile à installer, cette solution est particulièrement populaire auprès des entreprises qui cherchent une approche unifiée pour surveiller à la fois les métriques et les logs.

Bien que Grafana et Loki soient gratuits et open-source, les offres de Grafana Cloud permettent aux entreprises de bénéficier d’un hébergement géré avec des options de mise à l’échelle, le tout basé sur des tarifs compétitifs en fonction du volume de logs et des utilisateurs

### Promtail

Promtail est utilisé pour envoyer des logs à Loki, une base de données conçue pour être utilisée avec Grafana. L'installation est simple et intuitive, surtout si l’environnement Grafana est déjà en place. Les logs peuvent être visualisés dans Grafana via des dashboards personnalisables, ce qui rend cet ensemble parfait pour ceux qui recherchent une solution légère.

La version open-source de Loki et Promtail est gratuite, mais Grafana propose des offres cloud payantes avec des fonctionnalités premium comme l'hébergement géré, des options de mise à l’échelle, et un support technique. Ces forfaits sont basés sur la quantité de logs traités et les fonctionnalités utilisées.


### Solarwinds

SolarWinds est une solution tout-en-un qui se distingue par sa capacité à surveiller des environnements complexes. Elle permet de centraliser les logs tout en offrant des fonctionnalités de monitoring d’infrastructure. Les dashboards permettent de visualiser les performances en temps réel, ce qui en fait un outil précieux pour les administrateurs système. Cependant, l’installation peut s’avérer difficile dans des environnements de grande taille.

Les solutions SolarWinds sont payantes, avec des forfaits basés sur le nombre de nœuds et les fonctionnalités choisies. Ces forfaits incluent souvent des options de support technique, ainsi que des services avancés de surveillance des infrastructures et d'analyse des logs.

## Source utilisées : 

- https://www.syloe.com/collecte-et-traitement-des-logs/

- https://blog.wescale.fr/comment-mettre-en-place-une-solution-de-centralisation-de-logs
- https://newrelic.com/fr/resources/white-papers/log-management-best-practices
-https://www.splunk.com/fr_fr/data-insider/what-is-it-monitoring.html
- https://www.youtube.com/watch?v=dMzlclnDJLw
- https://documentation.solarwinds.com/en/success_center/observability/content/configure/configure-logs.htm