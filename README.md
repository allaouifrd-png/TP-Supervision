# TP-Supervision

# Exercice 1 — Prometheus (Module 1) 

**Objectif :** Lancer un seul conteneur Prometheus, accéder à l'interface web sur le port 9090 et vérifier que Prometheus se scrape lui-même.

Comme indiqué dans le TP, je vais suivre les étapes indiquées pour installer Prometheus et accéder à l’interface web sur le port 9090. Ci-dessous, les commandes que j’ai utilisées :

Récupération de l’image Prometheus (avec Docker) : sudo docker pull prom/prometheus:latest

Lancement du conteneur : 
```bash
sudo docker run --name prometheus -d -p 0.0.0.0:9090:9090 prom/prometheus
```
Accès à l’interface web : https://192.168.1.90:9090

Confirmation que la cible Prometheus est UP :

<img width="1887" height="348" alt="image" src="https://github.com/user-attachments/assets/69477fcb-a9cf-43ca-8582-fdb73aadafcb" />

Exécuter docker logs prometheus et lire la ligne de démarrage qui annonce le répertoire de stockage : 

Ci-dessous les logs indiquant le demarrage du répertoire de stockage [à expliquer]: 

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
