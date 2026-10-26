# 🛡️ Hardened PostgreSQL Container (Production-Grade Security)

Ce projet présente une configuration de conteneur PostgreSQL hautement sécurisée (*hardened*), conçue pour répondre aux exigences de production d'une infrastructure moderne (inspirée d'un cas réel pour la société *PixelPunch Gaming*). L'objectif est d'appliquer le **principe du moindre privilège** et d'assurer une défense en profondeur au niveau du noyau Linux et des conteneurs Docker.

---

## 🏗️ Architecture et Schéma de Sécurité

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
