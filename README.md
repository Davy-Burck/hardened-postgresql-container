# Hardened PostgreSQL Container (Production-Grade Security)

Ce projet présente une configuration de conteneur PostgreSQL hautement sécurisée (hardened), conçue pour répondre aux exigences de production d'une infrastructure moderne (inspirée d'un cas réel pour la société PixelPunch Gaming). L'objectif est d'appliquer le principe du moindre privilège et d'assurer une défense en profondeur au niveau du noyau Linux et des conteneurs Docker.

---

## Architecture et Schema de Securite

================================================================================
 [ MACHINE HÔTE (Linux / Docker Host) ]
================================================================================
  │
  ├─ Réseau Privé Isolé : [ pg_network ] (Sous-réseau : 11.10.11.0/24)
  │   └─ Isolation stricte du trafic de la base de données
  │
  ├─ Stockage Persistant : [ Volume Docker: pg_data ]
  │   └─ Disque Hôte (Garantit la pérennité des données hors cycle de vie du conteneur)
  │
  └─ CONTENEUR DOCKER : [ hardened_pg ] (Image: postgres:17.4)
      │
      ├─ Sécurité Noyau et Privilèges :
      │   ├─ Exécution UID : Utilisateur non-root [ 999:999 (postgres) ]
      │   ├─ Élévation de privilèges : Bloquée [--security-opt no-new-privileges:true]
      │   └─ Capacités du Noyau : Totalement supprimées [--cap-drop=ALL]
      │
      └─ Point de Montage Interne :
          └─ /var/lib/postgresql/data  ──>  Relié au Volume pg_data

---

## Guide de Deploiement

### 1. Prérequis
Assure-toi d'avoir Docker installé sur ta machine hôte Linux.

### 2. Création du réseau isolé et du volume persistant

# Création du réseau virtuel privé
docker network create --subnet=11.10.11.0/24 pg_network

# Création du volume pour la persistance des données sur l'hôte
docker volume create pg_data

3. Lancement du conteneur durci

docker run -d --rm \
  --name hardened_pg \
  -e POSTGRES_PASSWORD=MonSuperMotDePasseSecurise123! \
  --network pg_network --ip 11.10.11.25 \
  -v pg_data:/var/lib/postgresql/data \
  --user 999:999 \
  --security-opt no-new-privileges:true \
  --cap-drop=ALL \
  postgres:17.4

Mesures de Securite Implementees

    Isolation Réseau (--network) : Le conteneur est rattaché à un sous-réseau dédié (11.10.11.0/24) avec une IP fixe (11.10.11.25), évitant l'exposition inutile sur le pont réseau par défaut de Docker.

    Persistance des Données (-v) : Les données sont stockées sur l'hôte via un volume Docker, assurant qu'aucune donnée critique ne se perde lors de la suppression ou de la mise à jour du conteneur.

    Exécution Non-Root (--user 999:999) : Le conteneur démarre directement sous l'utilisateur non-privilégié postgres, prévenant toute faille de type Container Breakout vers la machine hôte.

    Suppression des Capacités Noyau (--cap-drop=ALL) : Toutes les capacités d'administration Linux par défaut sont retirées au processus pour réduire drastiquement la surface d'attaque.

    Verrouillage des Privilèges (no-new-privileges) : Empêche l'élévation de privilèges au sein du conteneur (notamment via des binaires SUID).

    Fixation de Version (postgres:17.4) : Utilisation d'un tag explicite et figé plutôt que latest pour garantir la stabilité et la reproductibilité de l'environnement de production.

Demonstration et Verifications
1. Vérification de l'utilisateur non-root

On s'assure que le conteneur s'exécute bien sous l'UID 999 (utilisateur postgres) et non en tant que root pour éviter toute élévation de privilèges :

docker exec hardened_pg id

(Résultat attendu : uid=999(postgres) gid=999(postgres))
2. Test de la connexion au réseau isolé

Connexion à l'instance PostgreSQL via l'adresse IP fixe définie sur notre réseau privé pg_network :
Bash

psql -h 11.10.11.25 -p 5432 -U postgres

3. Validation de la persistance des données

Création d'une base de données de test, suppression du conteneur, puis redémarrage pour prouver que les données survivent grâce au volume Docker pg_data :
SQL

CREATE DATABASE test_database;
