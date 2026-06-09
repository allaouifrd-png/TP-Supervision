# TP-Supervision

# Exercice 1 — Prometheus (Module 1) 

**Objectif :** Lancer un seul conteneur Prometheus, accéder à l'interface web sur le port 9090 et vérifier que Prometheus se scrape lui-même.

Comme indiqué dans le TP, je vais suivre les étapes indiquées pour installer Prometheus et accéder à l’interface web sur le port 9090. Ci-dessous, les commandes que j’ai utilisées :

Récupération de l’image Prometheus (avec Docker) : 
```bash
sudo docker pull prom/prometheus:latest
```
Lancement du conteneur : 
```bash
sudo docker run --name prometheus -d -p 0.0.0.0:9090:9090 prom/prometheus
```
Accès à l’interface web : https://192.168.1.90:9090

Confirmation que la cible Prometheus est UP :

<img width="1887" height="348" alt="image" src="https://github.com/user-attachments/assets/69477fcb-a9cf-43ca-8582-fdb73aadafcb" />

Exécuter docker logs prometheus et lire la ligne de démarrage qui annonce le répertoire de stockage : 

Ci-dessous les logs indiquant le demarrage du répertoire de stockage : 

```bash
time=2026-04-27T10:00:55.585Z level=INFO source=main.go:1410 msg="Starting TSDB ..."
time=2026-04-27T10:00:55.586Z level=INFO source=head.go:698 msg="Replaying on-disk memory mappable chunks if any" component=tsdb
time=2026-04-27T10:00:55.586Z level=INFO source=head.go:784 msg="On-disk memory mappable chunks replay completed" component=tsdb duration=743ns
time=2026-04-27T10:00:55.586Z level=INFO source=head.go:792 msg="Replaying WAL, this may take a while" component=tsdb
time=2026-04-27T10:00:55.587Z level=INFO source=head.go:865 msg="WAL segment loaded" component=tsdb segment=0 maxSegment=0 duration=997.64µs
time=2026-04-27T10:00:55.587Z level=INFO source=head.go:902 msg="WAL replay completed" component=tsdb checkpoint_replay_duration=8.238µs wal_replay_duration=1.03504ms wbl_replay_duration=129ns chunk_snapshot_load_duration=0s mmap_chunk_replay_duration=743ns total_replay_duration=1.056979ms
time=2026-04-27T10:00:55.589Z level=INFO source=main.go:1431 msg="filesystem information" fs_type=EXT4_SUPER_MAGIC
time=2026-04-27T10:00:55.589Z level=INFO source=main.go:1434 msg="TSDB started"
time=2026-04-27T10:00:55.589Z level=INFO source=main.go:1632 msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
time=2026-04-27T10:00:55.589Z level=INFO source=main.go:1048 msg="TSDB retention updated" duration=15d size=0B percentage=0
```
# Exercice 2 : Écrire votre premier prometheus.yml

**Objectif :** Remplacer la configuration par défaut par votre propre prometheus.yml. Définir un intervalle de scrape global de 10s, un external label environment=lab, et recharger Prometheus sans le redémarrer.

comme indiqué dans le TP, je vais arrêter le conteneur précédent grâce à cette commande :
```bash
docker rm -f prometheus
```
Ci-dessous, le contenu du fichier prometheus.yml :

```bash
global:
  scrape_interval: 10s

  external_labels:
    environment: lab

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```
Grâce à cette configuration, un job nommé prometheus est créé. Celui-ci va scraper ses propres métriques (localhost:9090), les labelliser avec environment=lab, avec une fréquence de scrape de 10 secondes.

Pour vérifier que cette configuration est bien appliquée, je vais utiliser la commande suivante pour lancer le conteneur :

```bash
sudo docker run -d \
  --name prometheus \
  -p 0.0.0.0:9090:9090 \
  -v /home/ubuntu/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest \
  --config.file=/etc/prometheus/prometheus.yml \
  --web.enable-lifecycle \
  --web.listen-address=0.0.0.0:9090
```
Explique ces lignes :  
--web.enable-lifecycle \
--web.listen-address=0.0.0.0:9090
curl -X POST http://localhost:9090/-/reload

La capture d’écran ci-dessous indique que la configuration appliquée est bien prise en compte : 

<img width="1900" height="497" alt="image" src="https://github.com/user-attachments/assets/9af067b6-86d8-42e9-b2ee-02d644e39b40" />

# Exercice 3 : Ajouter node_exporter et scraper les métriques système

**Objectif :** Lancer node_exporter et configurer Prometheus pour le scraper. Vérifier que la métrique node_cpu_seconds_total apparaît dans l'expression browser.

Comme indiqué dans le TP, à l’aide de cette commande, je vais lancer le conteneur node-exporter : 

```bash
docker run -d \
  --name node-exporter \
  -p 9100:9100 \
  prom/node-exporter:latest
```
La capture ci-dessous indique que le conteneur node-exporter est bien lancé :

<img width="1298" height="77" alt="image" src="https://github.com/user-attachments/assets/9e84a39f-8a62-4add-81fb-4e8cd8645693" />
</br>
Après avoir configuré le fichier prometheus.yml en y ajoutant le bloc ci-dessous, node-exporter apparaît bien dans les targets et est indiqué comme UP: 

```bash
  - job_name: 'node'
    static_configs:
      - targets: ['192.168.1.90:9100']
```

<img width="1885" height="685" alt="image" src="https://github.com/user-attachments/assets/8ac160a9-a855-45e3-9228-f6e4c784d3fc" />

Note : J’utilise la commande ci-dessous pour recharger la configuration de Prometheus :

```bash
curl -X POST http://localhost:9090/-/reload
```

La requête ci-dessous, exécutée sur Prometheus, me retourne bien un résultat :

```bash
node_cpu_seconds_total
```

<img width="1871" height="526" alt="image" src="https://github.com/user-attachments/assets/7f2afcbe-614c-4ba6-b213-0c0f889eea7e" />

Cette métrique est fournie par node-exporter et indique le temps CPU cumulé en secondes. Voir s’il est possible de rendre cette métrique plus lisible (pas obligatoire).

# Exercice 4 : Découverte de service : par fichier ou Kubernetes

**Objectif :** Remplacer les static_configs par un mécanisme de découverte. Sous Docker, utiliser la découverte par fichier ; sous Kubernetes, utiliser kubernetes_sd_configs avec un ServiceMonitor ou un bloc de découverte brut.

Créer un fichier targets.json contenant deux endpoints

Pour réaliser cela, j'ai créé un répertoire sd dans lequel j'ai placé le fichier targets.json. Ci-dessous figure le contenu du fichier targets.json. Deux endpoints y sont spécifiés : 172.17.0.1:9001 et 8.8.8.8:9100.

```bash
mkdir -p sd
nano sd/targets.json
```

```bash
[
  {
    "targets": ["172.17.0.1:9100, "8.8.8.8:9100"],
    "labels": {
      "job": "node"
    }
  }
]
```

Remplacer les static_configs d'un job par file_sd_configs pointant vers /etc/prometheus/sd/*.json

Pour m'assurer que Prometheus lit bien le fichier targets.json, j'ai retiré puis remis le conteneur Prometheus en montant le volume /home/Ubuntu/sd dans le conteneur sur /etc/prometheus/sd.

```bash
sudo docker run --name prometheus -d \
  -p 0.0.0.0:9090:9090 \
  -v /home/ubuntu/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v /home/ubuntu/sd:/etc/prometheus/sd \
  prom/prometheus \
  --config.file=/etc/prometheus/prometheus.yml
```
Les deux endpoints spécifiés dans le fichier "file_sd_configs" sont bien visibles sur Prometheus.

<img width="1912" height="547" alt="image" src="https://github.com/user-attachments/assets/9b2c12f1-cfa5-4e8d-9fba-712980103a04" />

La configuration statique est celle que j'ai commentée pour la remplacer par celle qui se trouve juste en dessous. Cette dernière lit les endpoints spécifiés dans le fichier targets.json. Un rafraîchissement toutes les 5 secondes a été mis en place dans le cadre de ce lab pour observer rapidement le résultat de la configuration.

```bash
#  - job_name: 'node'
#    static_configs:
#      - targets: ['192.168.1.90:9100']

  - job_name: 'node'
    file_sd_configs:
      - files:
          - /etc/prometheus/sd/*.json
        refresh_interval: 5s
```

Ajouter ou retirer une cible du JSON et confirmer que Prometheus la prend en compte sans rechargement

Après avoir retiré la cible 8.8.8.8:9100 du fichier targets.json, Prometheus la prend en compte 5 secondes après, sans que je recharge la configuration.

<img width="1910" height="360" alt="image" src="https://github.com/user-attachments/assets/6022c0ac-7d86-4ee2-b06d-016a96884e27" />

Grâce à cet exercice, j'ai pu voir qu'il est possible, via un fichier de configuration, de mettre à disposition des cibles et qu'elles soient découvertes par Prometheus de manière automatique, sans qu'il soit nécessaire de recharger la configuration.

# Exercice 5 : Règles d'enregistrement (recording rules)

**Objectif :** Pré-calculer une requête coûteuse sous forme de règle d'enregistrement. Créer un fichier de règles qui enregistre job:http_requests:rate5m toutes les 30 secondes.

Après avoir copié le répertoire fourni dans le cadre de ce TP, je me suis rendu dans ce répertoire et ai construit l'image Docker demo-api.

```bash
cd /app
```
```bash
sudo docker build -t demo-api:1.0 .
```
Une fois l'image construite, je l'ai lancée.

```bash
sudo docker run -d \
  --name demo-api \
  -p 8000:8000 \
  demo-api:1.0
```
Ensuite, j'ai exécuté le script fourni.

```bash
cd /app/traffic.sh
chmod +x traffic.sh
./traffic.sh
```
L'objectif est de générer des requêtes HTTP afin qu'elles soient vues par Prometheus.

Ensuite, j'ai spécifié demo-api dans le fichier targets.json pour qu'il soit vu par Prometheus.

```bash
[
  {
    "targets": ["172.17.0.1:9100"],
    "labels": {
      "job": "node"
    }
  },
  {
    "targets": ["172.17.0.4:8000"],
    "labels": {
      "job": "demo-api"
    }
  }
]
```

J'ai créé le répertoire qui contiendra la règle.

```bash
mkdir -p ~/rules

nano /rules/api_rules.yml

groups:
  - name: api_rules
    interval: 30s
    rules:
      - record: job:http_requests:rate5m
        expr: rate(demo_http_requests_total[5m])
```

Dans prometheus.yml, j'ai ajouté le répertoire que je viens de créer.

```bash
global:
  scrape_interval: 10s

  external_labels:
    environment: lab

rule_files:
  - "/etc/prometheus/rules/*.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    file_sd_configs:
      - files:
          - /etc/prometheus/sd/*.json
        refresh_interval: 5s
```

Ensuite, j'ai lancé Docker en montant le volume incluant le fichier de configuration Prometheus qui inclut la règle.

```bash
sudo docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /home/ubuntu/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v /home/ubuntu/sd:/etc/prometheus/sd \
  -v /home/ubuntu/rules:/etc/prometheus/rules \
  prom/prometheus
```
<img width="1905" height="786" alt="image" src="https://github.com/user-attachments/assets/7c260ef5-4657-478b-abea-00727053cb8d" />

Dans Status > Rule Health, on peut voir que le groupe de règles api_rules a bien été chargé par Prometheus. La règle job:http_requests:rate5m est exécutée toutes les 30 secondes et son état est OK, ce qui confirme que le fichier api_rules.yml est correctement pris en compte et que la requête est évaluée.

<img width="1898" height="412" alt="image" src="https://github.com/user-attachments/assets/6c539b49-6b99-4c80-912f-808c85582762" />

Grâce à cet exercice, j'ai pu découvrir le fonctionnement des règles d'enregistrement dans Prometheus. J'ai constaté qu'il était possible de pré-calculer certaines requêtes utilisées régulièrement afin d'enregistrer leur résultat comme une nouvelle métrique. Cela permet d'éviter de recalculer la même requête à chaque fois, ce qui améliore les performances de Prometheus et rend l'affichage des données plus rapide dans les tableaux de bord.

# Exercice 5 : Règles d'alerte et Alertmanager

**Objectif :** Définir une alerte qui se déclenche lorsque le taux d'erreur de demo-api dépasse 5 % pendant 2 minutes, la router vers Alertmanager, puis observer l'alerte qui se déclenche.

Pour réaliser cet exercice, dans un premier temps, j'ai créé un répertoire alertmanager dans lequel je vais placer mon fichier de configuration alertmanager.yml.

```bash
mkdir -p /alertmanager
cd alertemanager 
nano alertmanager.yml
```
Dans le fichier alertmanager.yml, j'ai inclus le contenu ci-dessous :

```bash
route:
  receiver: "default"

receivers:
  - name: "default"
```
La directive route définit le chemin pour savoir où envoyer une alerte et receiver est la liste des destinations spécifiées.

Ensuite, je vais lancer le conteneur Alertmanager via cette commande tout en veillant à monter le volume contenant mon fichier de configuration Alertmanager précédemment créé : 

```bash
sudo docker run -d \
  --name alertmanager \
  -p 9093:9093 \
  -v /home/ubuntu/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
  prom/alertmanager
```

Dans le fichier prometheus.yml, je vais ajouter Alertmanager. Voici les éléments à saisir :

```bash
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "172.17.0.2:9093"
```
Note : Cette commande me permet d'avoir l'ip exacte du conteneur Alertmanager : 

```bash
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' alertmanager
```
Ceci permettra de connecter Alertmanager à Prometheus et les alertes relevées sur Prometheus seront transmises à Alertmanager.

Une fois Alertmanager connecté à Prometheus, je vais créer un fichier d'alertes grâce à cette commande :

```bash
nano ~/rules/api_alerts.yml
```
et y inclure le contenu suivant :

```bash
groups:
  - name: api_alerts
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(demo_http_requests_total{status=~"5.."}[2m]))
            /
            sum(rate(demo_http_requests_total[2m]))
          ) > 0.05
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Taux d'erreur élevé"
          description: "Le taux d'erreur dépasse 5% depuis plus de 2 minutes."
```
Cette règle permet de surveiller le taux d'erreur de notre application demo-api et d'émettre une alerte si 5 % d'erreurs sont détectées en continu pendant au moins 2 minutes.

Ensuite, je vais redémarrer le conteneur Prometheus pour que la configuration appliquée puisse être rechargée : 

```bash
sudo docker restart prometheus
```
Sur l'interface Prometheus, je vois que l'alerte créée est bien prise en compte.

<img width="1902" height="627" alt="image" src="https://github.com/user-attachments/assets/2af4ae73-3b97-4fe9-a7af-535605a67ce6" />

Après avoir exécuté le script traffic.sh, dans la section Alerts, je vois bien les taux d'erreur qui remontent avec le flag Pending.

<img width="1898" height="783" alt="image" src="https://github.com/user-attachments/assets/def33925-6a0c-407b-9c98-b99dc2aba663" />

Sur cette capture d'écran, on peut voir que le taux d'erreur a bien dépassé les 5 % (6,22 %), mais le système attend 2 minutes pour vérifier si le problème persiste. Si le problème persiste encore au-delà de ces 2 minutes, l'alerte se déclenchera pour de bon, aura le flag Firing et sera envoyée à Alertmanager.

Dans la capture ci-dessous, nous pouvons voir que la condition de taux d'erreur de 5 % durant une période de 2 minutes est dépassée ; l'alerte possède désormais le flag Firing et est envoyée vers Alertmanager : 

<img width="1895" height="788" alt="image" src="https://github.com/user-attachments/assets/4f528522-eeff-40dd-99f9-4cfc60165497" />

Ci-dessous, l'alerte émise par Prometheus, présente sur Alertmanager, portant le nom HighErrorRate avec le label lab et la severity warning, ce qui correspond parfaitement aux éléments spécifiés dans les fichiers de configuration api_alerts.yml et prometheus.yml :

<img width="1510" height="635" alt="image" src="https://github.com/user-attachments/assets/e4fc0834-7e49-48bf-bdf7-5ac0d866817b" />

# Exercice 5 : PromQL - bases : vecteurs instantanés et vecteurs de plage

**Objectif :** Mettre en pratique la différence entre un vecteur instantané, un vecteur de plage et un scalaire. Répondre aux questions à partir des métriques de demo-api.

<img width="1878" height="370" alt="image" src="https://github.com/user-attachments/assets/4ebfa0ab-adde-4c3d-9891-feb0e9d8afcc" />

<img width="1873" height="708" alt="image" src="https://github.com/user-attachments/assets/5aacbb53-d6d9-48f5-9dcd-473db1090b5e" />

<img width="1875" height="357" alt="image" src="https://github.com/user-attachments/assets/e7609149-9f12-4c56-bba1-6c7e6ffc9fd3" />

<img width="1873" height="278" alt="image" src="https://github.com/user-attachments/assets/d30bd4fd-542e-467a-8062-42169a04d823" />






