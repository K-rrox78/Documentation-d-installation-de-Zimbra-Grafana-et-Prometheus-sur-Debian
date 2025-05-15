# Guide d'Installation de Grafana et Prometheus sur Debian

Ce guide détaille les étapes pour installer Grafana et Prometheus (serveur et agent) sur une machine Debian.

## Table des matières

1. [Installation de Prometheus Server](#installation-de-prometheus-server)
2. [Installation de Prometheus Node Exporter](#installation-de-prometheus-node-exporter)
3. [Installation de Grafana](#installation-de-grafana)
4. [Configuration de Grafana avec Prometheus](#configuration-de-grafana-avec-prometheus)
5. [Configuration pour la surveillance de services additionnels](#configuration-pour-la-surveillance-de-services-additionnels)

## Prérequis

- Serveur Debian (10 ou supérieur)
- Accès root ou sudo
- Ports 9090 (Prometheus), 9100 (Node Exporter), 3000 (Grafana) disponibles

## Installation de Prometheus Server

### Étape 1: Créer un utilisateur Prometheus

```bash
# Créer l'utilisateur prometheus sans droits de connexion
sudo useradd --no-create-home --shell /bin/false prometheus

# Créer les répertoires nécessaires
sudo mkdir -p /etc/prometheus /var/lib/prometheus

# Définir les permissions
sudo chown prometheus:prometheus /var/lib/prometheus
```

### Étape 2: Télécharger et installer Prometheus

```bash
# Télécharger la dernière version de Prometheus (vérifiez la dernière version sur prometheus.io)
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz

# Extraire l'archive
tar -xvf prometheus-2.37.0.linux-amd64.tar.gz

# Copier les binaires
sudo cp prometheus-2.37.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.37.0.linux-amd64/promtool /usr/local/bin/

# Définir les permissions
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

# Copier les fichiers de configuration
sudo cp -r prometheus-2.37.0.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.37.0.linux-amd64/console_libraries /etc/prometheus
sudo cp prometheus-2.37.0.linux-amd64/prometheus.yml /etc/prometheus/

# Définir les permissions
sudo chown -R prometheus:prometheus /etc/prometheus
```

### Étape 3: Configurer Prometheus

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Configuration de base:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

### Étape 4: Créer un service systemd pour Prometheus

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Contenu:
```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

### Étape 5: Démarrer Prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

## Installation de Prometheus Node Exporter

### Étape 1: Créer un utilisateur pour Node Exporter

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

### Étape 2: Télécharger et installer Node Exporter

```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz

sudo cp node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### Étape 3: Créer un service systemd pour Node Exporter

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Contenu:
```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

### Étape 4: Démarrer Node Exporter

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

### Étape 5: Configurer Prometheus pour scraper Node Exporter

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Ajoutez à la section `scrape_configs`:
```yaml
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```

Puis redémarrez Prometheus:
```bash
sudo systemctl restart prometheus
```

## Installation de Grafana

### Étape 1: Ajouter le dépôt Grafana

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### Étape 2: Installer Grafana

```bash
sudo apt-get update
sudo apt-get install -y grafana
```

### Étape 3: Démarrer Grafana

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

## Configuration de Grafana avec Prometheus

### Étape 1: Accéder à l'interface web de Grafana

Ouvrez votre navigateur et accédez à:
```
http://adresse-ip-serveur:3000
```

Les identifiants par défaut sont:
- Utilisateur: `admin`
- Mot de passe: `admin`

Vous serez invité à changer le mot de passe lors de la première connexion.

### Étape 2: Ajouter Prometheus comme source de données

1. Dans le menu latéral de Grafana, cliquez sur "Configuration" (icône d'engrenage)
2. Sélectionnez "Data Sources"
3. Cliquez sur "Add data source"
4. Sélectionnez "Prometheus"
5. Dans le champ URL, entrez `http://localhost:9090`
6. Cliquez sur "Save & Test"

### Étape 3: Importer un tableau de bord

1. Dans le menu latéral, cliquez sur "+" puis "Import"
2. Entrez l'ID `1860` (Node Exporter Full dashboard) dans le champ "Import via grafana.com"
3. Cliquez sur "Load"
4. Sélectionnez votre source de données Prometheus dans le menu déroulant
5. Cliquez sur "Import"

Vous devriez maintenant voir un tableau de bord complet avec des métriques sur votre système.

## Configuration pour la surveillance de services additionnels

### Configuration pour la surveillance de MySQL

#### Installation de l'exportateur MySQL

```bash
cd /tmp
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.14.0/mysqld_exporter-0.14.0.linux-amd64.tar.gz
tar -xvf mysqld_exporter-0.14.0.linux-amd64.tar.gz

sudo cp mysqld_exporter-0.14.0.linux-amd64/mysqld_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/mysqld_exporter
```

#### Créer un utilisateur MySQL pour l'exportateur

```bash
sudo mysql -u root -p

CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'StrongPassword' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### Configurer les identifiants MySQL

```bash
sudo nano /etc/.mysqld_exporter.cnf
```

Contenu:
```ini
[client]
user=exporter
password=StrongPassword
```

```bash
sudo chown node_exporter:node_exporter /etc/.mysqld_exporter.cnf
sudo chmod 600 /etc/.mysqld_exporter.cnf
```

#### Créer un service systemd pour l'exportateur MySQL

```bash
sudo nano /etc/systemd/system/mysqld_exporter.service
```

Contenu:
```ini
[Unit]
Description=MySQL Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/mysqld_exporter --config.my-cnf=/etc/.mysqld_exporter.cnf

[Install]
WantedBy=multi-user.target
```

#### Démarrer l'exportateur MySQL

```bash
sudo systemctl daemon-reload
sudo systemctl start mysqld_exporter
sudo systemctl enable mysqld_exporter
```

#### Configurer Prometheus pour scraper l'exportateur MySQL

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Ajoutez à la section `scrape_configs`:
```yaml
  - job_name: 'mysql'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9104']
```

Redémarrez Prometheus:
```bash
sudo systemctl restart prometheus
```

### Configuration pour la surveillance d'autres serveurs (agents distants)

Pour surveiller plusieurs serveurs depuis votre instance Prometheus centrale:

1. Installez Node Exporter sur chaque serveur à surveiller
2. Configurez Prometheus pour scraper les Node Exporters distants

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Ajoutez:
```yaml
  - job_name: 'remote_nodes'
    scrape_interval: 5s
    static_configs:
      - targets: ['serveur1:9100', 'serveur2:9100', 'serveur3:9100']
        labels:
          environment: production
```

Redémarrez Prometheus:
```bash
sudo systemctl restart prometheus
```

## Vérification et dépannage

### Vérifier que Prometheus fonctionne

```bash
curl http://localhost:9090/metrics
```

### Vérifier que Node Exporter fonctionne

```bash
curl http://localhost:9100/metrics
```

### Logs Prometheus

```bash
sudo journalctl -u prometheus -f
```

### Logs Grafana

```bash
sudo journalctl -u grafana-server -f
```

### Problèmes de connexion

Si vous ne pouvez pas vous connecter aux interfaces web, vérifiez que les ports sont ouverts:

```bash
sudo apt install -y net-tools
sudo netstat -tuln | grep -E '9090|9100|3000'
```

Configurez le pare-feu si nécessaire:

```bash
sudo apt install -y ufw
sudo ufw allow 9090/tcp
sudo ufw allow 9100/tcp
sudo ufw allow 3000/tcp
```
