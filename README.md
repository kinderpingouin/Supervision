# 📊 Stack de Supervision Docker + Host

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)

Ce projet met en place une infrastructure de monitoring complète pour superviser un hôte Linux et l'ensemble de ses conteneurs Docker en temps réel. Cette stack utilise les standards de l'industrie pour garantir une observabilité maximale.

## 🚀 Composants de la solution

La stack est composée des services suivants :

*   **Prometheus** : Système de collecte et base de données pour les métriques temporelles.
*   **Grafana** : Interface de visualisation permettant de créer des tableaux de bord dynamiques.
*   **Node Exporter** : Agent collectant les métriques de l'hôte physique (CPU, RAM, Disques, Réseau).
*   **cAdvisor** : Outil d'analyse de l'utilisation des ressources et des performances des conteneurs actifs.

## 📂 Structure du projet

```text
.
├── docker-compose.yml       # Définition des services et volumes
├── prometheus/
│   └── prometheus.yml       # Configuration du scraping des cibles
├── grafana/
│   └── provisioning/        # Configuration automatique (datasources & dashboards)
├── ansible/                 # Déploiement distant automatisé (playbook unique)
└── README.md
```

## 🛠️ Prérequis et Démarrage

### Prérequis
*   Docker et Docker Compose installés sur la machine hôte.
*   Port `3000` (Grafana) disponible — les autres services communiquent
    uniquement via le réseau interne Docker.

### Démarrage
1. Clonez ce dépôt sur votre machine et accédez au dossier du projet :
   ```bash
   git clone https://github.com/votre-utilisateur/Supervision.git
   cd Supervision
   ```
2. Lancez l'ensemble des services en mode détaché :
   ```bash
   docker-compose up -d
   ```

## ☁️ Déploiement distant (Ansible)

Un playbook unique ([`ansible/playbook.yml`](./ansible/playbook.yml)) déploie la
stack sur un serveur distant :
1. active le **cgroup mémoire** si nécessaire (Raspberry Pi OS le désactive par
   défaut, ce qui fait remonter des métriques RAM à 0 dans cAdvisor) — cette
   étape modifie la ligne de boot et **redémarre le serveur** ;
2. installe **Docker Engine** + le plugin **Compose** via le script officiel
   [get.docker.com](https://get.docker.com) ;
3. copie les fichiers de configuration du projet dans `/opt/supervision` ;
4. lance la stack avec `docker compose up -d`.

### Prérequis

*   **Machine de contrôle** (votre poste) : Ansible installé
    (`pipx install ansible` ou `apt install ansible`).
*   **Serveur cible** : une distribution supportée par get.docker.com
    (Debian, Ubuntu, Fedora, etc.), un accès **SSH** avec droits **sudo**,
    et le port `3000` disponible.

> Aucune collection Ansible externe n'est requise : seuls les modules du cœur
> (`ansible.builtin`) sont utilisés.

### Configuration

1. **Inventaire** — copiez le modèle puis renseignez l'IP/hostname et
   l'utilisateur SSH de votre serveur (`inventory.ini` est ignoré par git,
   vos coordonnées ne seront jamais commitées) :
   ```bash
   cp ansible/inventory.ini.example ansible/inventory.ini
   ```
   ```ini
   [supervision]
   monitoring-server ansible_host=VOTRE_IP ansible_user=VOTRE_USER
   ```

2. **Variables** (optionnel) — en tête de `ansible/playbook.yml`, personnalisez
   le répertoire de déploiement et surtout le **mot de passe Grafana** :
   ```yaml
   grafana_admin_user: admin
   grafana_admin_password: "un-mot-de-passe-solide"
   ```
   Pour ne pas stocker le mot de passe en clair, utilisez Ansible Vault :
   ```bash
   ansible-vault encrypt_string 'MonMotDePasse' --name grafana_admin_password
   ```

### Déploiement

```bash
cd ansible

# Test de connexion
ansible supervision -m ping

# Déploiement (ajoutez --ask-become-pass si sudo requiert un mot de passe)
ansible-playbook playbook.yml
```

Une fois terminé, Grafana est accessible sur `http://VOTRE_IP:3000`.

Le playbook est **idempotent** : relancez-le après une modification des
fichiers de configuration pour les appliquer. `docker compose up -d` ne recrée
que les conteneurs dont la configuration a changé.

## 🌐 Accès aux interfaces

Seul **Grafana** est publié sur l'hôte : `http://localhost:3000`
(identifiants par défaut `admin` / `admin`).

Prometheus (`9090`), cAdvisor (`8080`) et Node Exporter (`9100`) ne sont
accessibles que depuis le réseau interne Docker.

## ⚙️ Configuration de Grafana

Le provisionnement est entièrement automatisé. Au démarrage de la stack, un conteneur d'initialisation (`download-dashboards`) télécharge les fichiers JSON officiels et applique les corrections nécessaires (via `sed`) pour lier la source de données.

Pour visualiser vos métriques :

1. Connectez-vous à Grafana (`admin` / `admin`).
2. Rendez-vous directement dans le menu **Dashboards**.
3. Les tableaux de bord **Node Exporter Full** et **y0nei's cAdvisor dashboard** sont immédiatement disponibles et opérationnels.

##  Gestion des données

Les données de Prometheus et Grafana sont stockées dans des **volumes nommés** Docker (`prometheus_data` et `grafana_data`). Cela garantit que vos métriques et configurations ne sont pas perdues lors des redémarrages.

*   **Arrêter la stack en conservant les données** :
    ```bash
    docker-compose down
    ```
*   **Réinitialiser la stack (suppression totale des données)** :
    ```bash
    docker-compose down -v
    ```

##  Maintenance
Si vous rencontrez une erreur de type "no such file or directory" liée aux volumes, tentez de réinitialiser les volumes spécifiques :
```bash
docker volume rm supervision_grafana_data    # Pour Grafana
docker volume rm supervision_prometheus_data # Pour Prometheus
```

## 🔒 Sécurité

> [!IMPORTANT]
> Cette configuration est optimisée pour un usage local ou un réseau privé. Pour une exposition sur internet :
> *   Changez immédiatement le mot de passe administrateur de Grafana.
> *   Utilisez un Reverse Proxy (comme Nginx ou Traefik) avec terminaison SSL.
> *   Les ports de collecte (`9090`, `9100`, `8080`) ne sont pas publiés sur l'hôte ; ils restent accessibles uniquement via le réseau interne de Docker.

## 📄 Licence

Ce projet est sous licence MIT.
