# Hardened PostgreSQL Container (Production-Grade Security)

Ce projet présente une configuration de conteneur PostgreSQL hautement sécurisée (hardened), conçue pour répondre aux exigences de production d'une infrastructure moderne (inspirée d'un cas réel pour la société PixelPunch Gaming). L'objectif est d'appliquer le principe du moindre privilège et d'assurer une défense en profondeur au niveau du noyau Linux et des conteneurs Docker.

---

## Sommaire

- [Architecture et Schéma de Sécurité](#architecture-et-schéma-de-sécurité)
- [Guide de Déploiement](#guide-de-déploiement)
- [Mesures de Sécurité Implémentées](#mesures-de-sécurité-implémentées)
- [Démonstration et Vérifications](#démonstration-et-vérifications)

---

## Architecture et Schéma de Sécurité

```text
+--------------------------------------------------------------------------+
|                  MACHINE HÔTE (Linux / Docker Host)                      |
+--------------------------------------------------------------------------+
       |
       +---> 🌐 Réseau Privé Isolé : [ pg_network ] (11.10.11.0/24)
       |     └── Isolation stricte et trafic cloisonné de la base de données
       |
       +---> 💾 Stockage Persistant : [ Volume Docker: pg_data ]
       |     └── Disque Hôte (Pérennité des données hors cycle de vie du conteneur)
       |
       +---> 🐳 CONTENEUR DOCKER : [ hardened_pg ] (Image: postgres:17.4)
             |
             +---> 👤 Exécution Non-Root : UID 999:999 (Utilisateur postgres)
             +---> 🔒 Sécurité Noyau : --security-opt no-new-privileges:true
             +---> ⚡ Capacités : --cap-drop=ALL (Toutes les capacités supprimées)
             +---> 📂 Montage : /var/lib/postgresql/data --> Lié à pg_data

Guide de Déploiement
1. Prérequis
Assure-toi d'avoir Docker installé sur ta machine hôte Linux.

2. Création du réseau isolé et du volume persistant

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

Mesures de Sécurité Implémentées

   -- Isolation Réseau (--network) : Rattachement à un sous-réseau dédié (11.10.11.0/24) avec une IP fixe (11.10.11.25), évitant toute exposition sur le pont réseau par défaut.

   --  Persistance des Données (-v) : Stockage externe via un volume Docker pour garantir qu'aucune donnée ne disparaisse lors de la suppression ou de la mise à jour du conteneur.

   -- Exécution Non-Root (--user 999:999) : Démarrage direct sous l'utilisateur non-privilégié postgres pour prévenir toute faille de type Container Breakout.

   -- Suppression des Capacités Noyau (--cap-drop=ALL) : Retrait de toutes les capacités d'administration Linux par défaut pour réduire drastiquement la surface d'attaque.

   -- Verrouillage des Privilèges (no-new-privileges) : Interdiction formelle d'élever ses privilèges au sein du conteneur (protection contre les binaires SUID).

   -- Fixation de Version (postgres:17.4) : Utilisation d'un tag explicite et figé (au lieu de latest) pour assurer la stabilité en production.

Démonstration et Vérifications
1. Vérification de l'utilisateur non-root

S'assurer que le conteneur s'exécute bien sous l'UID 999 (et non en root) :

docker exec hardened_pg id

(Résultat attendu : uid=999(postgres) gid=999(postgres))
2. Test de la connexion au réseau isolé

Connexion à l'instance PostgreSQL via l'adresse IP fixe du réseau privé :

psql -h 11.10.11.25 -p 5432 -U postgres

3. Validation de la persistance des données

Création d'une base de test, suppression du conteneur, puis redémarrage pour vérifier la persistance :

CREATE DATABASE test_database;
