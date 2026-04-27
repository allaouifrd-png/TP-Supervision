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

Après avoir configuré le fichier prometheus.yml en y ajoutant le bloc ci-dessous, node-exporter apparaît bien dans les targets et est indiqué comme UP: 

```bash
  - job_name: 'node-exporter'
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

<img width="1863" height="443" alt="image" src="https://github.com/user-attachments/assets/1268db53-7892-49a1-87f0-750aedd541f8" />

Cette métrique est fournie par node-exporter et indique le temps CPU cumulé en secondes. Voir s’il est possible de rendre cette métrique plus lisible (pas obligatoire).
