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
└── README.md
```

## 🛠️ Prérequis et Démarrage

### Prérequis
*   Docker et Docker Compose installés sur la machine hôte.
*   Ports `3000`, `9090`, `8080` et `9100` disponibles.

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

## 🌐 Accès aux interfaces

| Service | URL | Identifiants par défaut |
| :--- | :--- | :--- |
| **Grafana** | `http://localhost:3000` | `admin` / `admin` |
| **Prometheus** | `http://localhost:9090` | - |
| **cAdvisor** | `http://localhost:8080` | - |
| **Node Exporter** | `http://localhost:9100/metrics` | - |

## ⚙️ Configuration de Grafana

La source de données **Prometheus** est configurée automatiquement via le provisioning. Pour commencer à visualiser vos données :

1. Connectez-vous à Grafana (`admin`/`admin`).
2. Allez dans **Dashboards** > **New** > **Import**.
3. Utilisez les IDs suivants pour importer des vues professionnelles :
    *   **1860** : Dashboard complet pour l'hôte (Node Exporter).
    *   **19724**: Dashboard pour les conteneurs docker (cAdvisor).

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
> *   Ne publiez pas les ports de collecte (`9090`, `9100`, `8080`) sur votre interface publique ; laissez-les accessibles uniquement via le réseau interne de Docker ou l'hôte local.

## 📄 Licence

Ce projet est sous licence MIT.
