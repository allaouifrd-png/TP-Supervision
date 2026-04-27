# TP-Supervision

# Exercice 1 — Prometheus (Module 1) 

**Objectif :** Lancer un seul conteneur Prometheus, accéder à l'interface web sur le port 9090 et vérifier que Prometheus se scrape lui-même.

Comme indiqué dans le TP, je vais suivre les étapes indiquées pour installer Prometheus et accéder à l’interface web sur le port 9090. Ci-dessous, les commandes que j’ai utilisées :

Récupération de l’image Prometheus (avec Docker) : sudo docker pull prom/prometheus:latest

Lancement du conteneur : sudo docker run --name prometheus -d -p 0.0.0.0:9090:9090 prom/prometheus

Accès à l’interface web : https://192.168.1.90:9090

Confirmation que la cible Prometheus est UP :
