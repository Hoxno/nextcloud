# Installation Nextcloud
<!---
BEGIN
-->

### Sommaire
0. [Introduction](#introduction)
1. [Installation d'Apache2 et PHP](#1-installation-dapache2-et-php)
2. [Installation de Nextcloud](#2-installation-de-nextcloud)


## Introduction

Cette installation se fera sur une installation d'une debian fraichement installer et ce tutoriel fonctionne aussi sur la distribution raspbian qui fonctionne sur une raspberry.
Toute les commande se feront en mode root.

## 1. Installation d'Apache2 et PHP
On commence par vérifier si le système st à jour

    apt update && apt dist-upgrade

Ensuite ont va installer des paquets de base
    apt install nmon htop

Et ont termine par l'installation d'Apache et de PHP (tout sur une même ligne)

    apt install apache2 php php-{sqlite3,curl,gd,intl,xml,zip,mbstring,json,mysql}  mariadb-server

masquer la version d'apache
il faut modifier le fichier /etc/apache2/conf-available/security.conf
et modifier la ligne ou se trouve la ligne ServerSignature et le mettre à off

## 2. Installation de Nextcloud

On commence par se positionner dans le dossier ``/var/www/``

    cd /var/www
    wget https://download.nextcloud.com/server/releases/nextcloud-18.0.2.tar.bz2

Après avoir télécharger l'archive de nextcloud, ont peut extraire le contenue dans le dossier ``html``.

On commence par supprimer le fichier index.html qui se trouve dans le dossier ``html``

    rm html/index.html
Après avoir supprimer le fichier ``index.html``, ont peut maintenant extraire l'archive

    tar -xf nextcloud-18.0.2.tar.bz2

Pour copier les fichiers et dossiers qui se trouvent dant le dossier ``nextcloud``, ont va utiliser la commande ``rsync``, pour que plus tard ça puisse nous faciliter une prochiane mise à jour de nextcloud.

    rsync -a nextcloud/ html/

Ont modifier les droits du dossier html et tous son contenue

    chown -R www-data html/

ont supprime le dossier nextcloud et l'archive
    rm -rf nextcloud
    rm -rf nextcloud-18.0.2.tar.bz2

Ont peut maintenant se connecter sur le nexcloud soit avec le nom de l'hôte (http://nom-de-l'hôte/) soit par adresse ip (http://192.168.1.X)

