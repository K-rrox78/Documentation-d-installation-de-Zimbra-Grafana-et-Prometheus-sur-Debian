# Guide d'Installation du Serveur de Messagerie Zimbra sur Debian

Ce guide détaille les étapes pour installer Zimbra Collaboration Suite sur un serveur Debian.

## Prérequis

- Serveur Debian (10 ou supérieur) avec au moins:
  - 4 Go de RAM (8 Go recommandés)
  - 30 Go d'espace disque
  - 2 CPUs / vCPUs
- Un nom de domaine configuré avec les enregistrements DNS appropriés:
  - Enregistrement A pour mail.votredomaine.com pointant vers l'IP de votre serveur
  - Enregistrement MX pour votredomaine.com pointant vers mail.votredomaine.com
  - Enregistrements SPF, DKIM et DMARC (recommandés)
- Un accès root ou sudo
- Ports nécessaires ouverts sur votre pare-feu

## Étape 1: Préparation du système

```bash
# Mettre à jour le système
apt update && apt upgrade -y

# Installer les dépendances
apt install -y netcat sudo libidn11 libpcre3 libgmp10 libexpat1 libstdc++6 libperl5.32 libaio1 unzip pax sysstat sqlite3
```

## Étape 2: Configurer le nom d'hôte

```bash
# Définir le nom d'hôte
hostnamectl set-hostname mail.votredomaine.com

# Vérifier que le nom d'hôte est correctement configuré
hostname -f
```

## Étape 3: Configurer le fichier hosts

Modifiez le fichier `/etc/hosts`:

```bash
nano /etc/hosts
```

Ajoutez la ligne suivante (remplacez avec votre IP et votre nom de domaine):
```
192.168.1.100 mail.votredomaine.com mail
```

## Étape 4: Désactiver Postfix (s'il est installé)

```bash
# Vérifier si postfix est installé
dpkg -l | grep postfix

# Si installé, arrêter et désactiver le service
systemctl stop postfix
systemctl disable postfix
```

## Étape 5: Télécharger Zimbra

```bash
# Créer un répertoire pour l'installation
mkdir -p /opt/zimbra-install
cd /opt/zimbra-install

# Télécharger la dernière version de Zimbra (vérifiez la dernière version sur zimbra.com)
wget https://files.zimbra.com/downloads/8.8.15_GA/zcs-8.8.15_GA_4179.DEBIAN10_64.20211118033954.tgz
tar -xzvf zcs-*.tgz
```

## Étape 6: Installer Zimbra

```bash
cd zcs-*

# Exécuter le script d'installation
./install.sh
```

Pendant l'installation:

1. Acceptez la licence en tapant `Y`
2. Choisissez les packages à installer (généralement tous)
3. À la question "Modify system for Zimbra MTA?" répondez `Y`
4. Validez les informations DNS, nom d'hôte et domaine
5. Configurez un mot de passe administrateur lorsque demandé (pour le compte admin@votredomaine.com)

## Étape 7: Finaliser l'installation

Une fois l'installation terminée, vous pourrez accéder à l'interface d'administration via:
- https://mail.votredomaine.com:7071

Et à l'interface webmail pour les utilisateurs via:
- https://mail.votredomaine.com

## Commandes utiles post-installation

```bash
# Vérifier le statut de tous les services Zimbra
su - zimbra -c "zmcontrol status"

# Redémarrer tous les services Zimbra
su - zimbra -c "zmcontrol restart"

# Arrêter tous les services Zimbra
su - zimbra -c "zmcontrol stop"

# Démarrer tous les services Zimbra
su - zimbra -c "zmcontrol start"
```

## Étape 8: Configuration de base après installation

### Créer un compte utilisateur

```bash
su - zimbra
zmprov ca utilisateur@votredomaine.com MotDePasse
```

### Configurer une liste de diffusion

```bash
su - zimbra
zmprov cdl liste@votredomaine.com
```

### Mettre en place des règles anti-spam avancées

Connectez-vous à l'interface d'administration (https://mail.votredomaine.com:7071) et configurez les règles anti-spam selon vos besoins.

## Dépannage

### Problèmes de DNS
Vérifiez vos entrées DNS avec:
```bash
dig MX votredomaine.com
dig A mail.votredomaine.com
```

### Erreurs de certificat SSL
Si vous rencontrez des problèmes de certificat, vous pouvez générer un nouveau certificat avec:
```bash
su - zimbra
zmcertmgr createca
zmcertmgr deployca
zmcertmgr createcrt
zmcertmgr deploycrt
zmcontrol restart
```

### Logs à consulter
Les logs principaux se trouvent dans:
```
/var/log/zimbra.log
/opt/zimbra/log/
```
