# Installation de Nextcloud
<!---
BEGIN
-->

## Sommaire
0. [Introduction](#introduction)
1. [Installation d'Apache2, PHP et MariaDB](#1-installation-dapache2-php-et-mariadb)
2. [Configuration de la base de données pour Nextcloud](#2-configuration-de-la-base-de-donnees-pour-nextcloud)
3. [Installation de Nextcloud](#3-installation-de-nextcloud)
4. [Configuration d'Apache pour NextCloud](#4-configuration-dapache-pour-nextcloud)
5. [Sécuriser Nextcloud avec Let's Encrypt](#5-securiser-nextcloud-avec-lets-encrypt)

## Introduction

__Nextcloud__ est un logiciel libre de site d'hébergement de fichiers et une plateforme de collaboration. C'est un outil très similaire aux service de cloud tel que ( __Google drive__, __Dropbox__, __iCloud__ ). Vous pourrez stocker et centraliser tous vos documents pour y avoir accès à tout moment. Avec __Nextcloud__ vous pourrez partager vos fichier, vos contacts et d'autres fichiers médias.

Cette installation se fera à partir d'une Debian fraichement installé et ce tutoriel fonctionne aussi sur la distribution raspbian qui tourne sur un raspberry.
Toute les commandes se feront en mode root.

## 1. Installation d'Apache2, PHP et MariaDB

__Nextcloud__ fonctionne sur un serveur web, qui est écrit en PHP et qui utilise MariaDB comme base de données.
Dans cette pemière partie, on va avoir besoins d'installer des paquets necessaires.

    apt install apache2 libapache2-mod-php mariadb-server wget unzip php php-{common,curl,gd,intl,xml,zip,mbstring,json,mysql,unzip,bcmath,gmp,imagick,apcu}

Si on veut masquer la version d'apache, il faut modifier le fichier

    nano /etc/apache2/conf-available/security.conf

et de modifier la ligne ``ServerSignature`` et de le mettre à off

Une fois les paquets installé, on ouvre le fichier ___php.ini___.

    nano /etc/php/7.3/apache2/php.ini

Et on modifie les paramètres suivantes:

    max_execution_time = 300
    memory_limit = 1024
    date.timezone = Europe/Paris
    opcache.enable=1
    opcache.enable_cli=1
    opcache.interned_strings_buffer=8
    opcache.max_accelerated_files=10000
    opcache.memory_consumption=128
    opcache.save_comments=1
    opcache.revalidate_freq=1
    session.cookie_httponly = True
    max_input_vars = 1740
    post_max_size=100M
    upload_max_filesize=500M

Après avois fait les modifications, on sauvegarde et on ferme le fichier. Ensuite on démarre les service d'Apache et de MariaDB, et on active le fait que les services démarre au démmarage du serveur avec les commandes suivantes:

    systemctl start apache2
    systemctl start mariadb
    systemctl enable apache2
    systemctl enable mariadb

Un fois que tous est fait, on peut passer à l'étape suivante.

## 2. Configuration de la base de données

Pour que __Nextcloud__ fonctionne on va avoir besoin de créer une base de données. Mais jsute avant on va lancer sécurisé mariaDB.
Pour cela, on lance la commande suivante.

    mysql_secure_installation

Et maintenant on peut se connecter à mariaDB de manière sécurisé.

    mysql -u root -p

Entrez le mot de passe quand ça sera demandé et ensuite on créer une base données avec la commande suivante. On créer ensuite un utilisateur qui sera utilisé pour la connection au nextcloud.

    CREATE DATABASE nextclouddb;
    CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'password';

On donne les droits à l'utilisateur sur la base de données

    GRANT ALL ON nextclouddb.* TO 'nextclouduser'@'localhost';

Et pour finir on applique les droits et on quitte

    FLUSH PRIVILEGES;
    EXIT;

Une fois que cette étape est faite, on peut passer à l'étape suivante.

## 3. Installation de Nextcloud

Maintenant que l'on a installée PHP,Apache et qu'on a configuré MariaDB, nous allons nous attaqué sur l'installation de Nextcloud. On commence par se positionner dans le dossier ``/var/www/``

    cd /var/www
    wget https://download.nextcloud.com/server/releases/nextcloud-19.0.2.tar.bz2

Après avoir télécharger l'archive de nextcloud, ont peut extraire le contenue dans le dossier ``html``. Pour cela on commence par supprimer le fichier index.html qui se trouve dans le dossier ``html``

    rm html/index.html
Après avoir supprimer le fichier ``index.html``, ont peut maintenant extraire l'archive

    tar -xf nextcloud-19.0.2.tar.bz2

Pour copier les fichiers et dossiers qui se trouvent dant le dossier ``nextcloud``, ont va utiliser la commande ``rsync``, pour que plus tard ça puisse nous faciliter une prochiane mise à jour de nextcloud.

    rsync -a nextcloud/ html/

Ont modifier les droits du dossier html et tous son contenue

    chown -R www-data:www-data html/
    chmod -R 755 html/

ont supprime le dossier nextcloud et l'archive
    rm -rf nextcloud
    rm -rf nextcloud-18.0.2.tar.bz2

Ont peut maintenant se connecter sur le nexcloud soit avec le nom de l'hôte (http://nom-de-l'hôte/) soit par adresse ip (http://192.168.1.X)

Une fois que cette étape est faite, on peut passer à l'étape suivante.

## 4. Configuration d'Apache pour NextCloud

Nous aurons besoin de créer un hôte vituel, pour cela on va devoir créer un fichier ```nextcloud.conf``` dans le dossier __/etc/apache2/sites-available/__. 

    nano /etc/apache2/sites-available/nextcloud.conf

Et on ajoutes les lignes suivantes:

    <VirtualHost *:80>
     ServerAdmin admin@example.com
     DocumentRoot /var/www/html/nextcloud/
     ServerName nextcloud.example.com

     Alias /nextcloud "/var/www/html/nextcloud/"

     <Directory /var/www/html/nextcloud/>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
          <IfModule mod_dav.c>
            Dav off
          </IfModule>
        SetEnv HOME /var/www/html/nextcloud
        SetEnv HTTP_HOME /var/www/html/nextcloud
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined

    </VirtualHost>

On sauvegarde et on ferme le fichier, en suite on active l'hôte virtuel Apache et d'autre module qui sont requis en utilisant les commandes suivantes:

    a2ensite nextcloud.conf
    a2enmod rewrite
    a2enmod headers
    a2enmod env
    a2enmod dir
    a2enmod mime

Et on termine par redémmarer le service Apache pour appliqué la nouvelle configuration.

    systemctl restart apache2

## 5. Sécurisation de Nextcloud avec Let's Encrypt

Maintenant que __NextCloud__  est correctement installé et configuré. Il est fortement recommandé de le sécurié avec Let's Encrypt. Pour ce faire, il faut d'abord installé le paquet``Certbot`` avec la commande suivante.

    apt-get install python-certbot-apache -y

Un fois ``Certbot`` installé, on peut lancer la commande suivante pour configurer le certificat de notre nom de domaine.

    certbot --apache -d nextcloud.example.com

Durant l'installation, il faudra entré une adresse mail et réponde au question.

    Saving debug log to /var/log/letsencrypt/letsencrypt.log
    Plugins selected: Authenticator apache, Installer apache
    Enter email address (used for urgent renewal and security notices) (Enter 'c' to
    cancel): admin@example.com

    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    Please read the Terms of Service at
    https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
    agree in order to register with the ACME server at
    https://acme-v02.api.letsencrypt.org/directory
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    (A)gree/(C)ancel: A

    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    Would you be willing to share your email address with the Electronic Frontier
    Foundation, a founding partner of the Let's Encrypt project and the non-profit
    organization that develops Certbot? We'd like to send you email about our work
    encrypting the web, EFF news, campaigns, and ways to support digital freedom.

    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    (Y)es/(N)o: Y
    Obtaining a new certificate
    Performing the following challenges:
    http-01 challenge for nextcloud.example.com
    Enabled Apache rewrite module
    Waiting for verification...
    Cleaning up challenges
    Created an SSL vhost at /etc/apache2/sites-available/nextcloud-le-ssl.conf
    Deploying Certificate to VirtualHost /etc/apache2/sites-available/nextcloud-le-ssl.conf
    Enabling available site: /etc/apache2/sites-available/nextcloud-le-ssl.conf

    Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    1: No redirect - Make no further changes to the webserver configuration.
    2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
    new sites, or if you're confident your site works on HTTPS. You can undo this
    change by editing your web server's configuration.
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
    Enabled Apache rewrite module
    Redirecting vhost in /etc/apache2/sites-enabled/nextcloud.conf to ssl vhost in /etc/apache2/sites-available/
    nextcloud-le-ssl.conf

    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    Congratulations! You have successfully enabled https://nextcloud.example.com

    You should test your configuration at:
    https://www.ssllabs.com/ssltest/analyze.html?d=nextcloud.example.com
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    IMPORTANT NOTES:
    - Congratulations! Your certificate and chain have been saved at:
    /etc/letsencrypt/live/example.com/fullchain.pem
    Your key file has been saved at:
    /etc/letsencrypt/live/example.com/privkey.pem
    Your cert will expire on 2019-10-22. To obtain a new or tweaked
    version of this certificate in the future, simply run certbot again
    with the "certonly" option. To non-interactively renew *all* of
    your certificates, run "certbot renew"
    - Your account credentials have been saved in your Certbot
    configuration directory at /etc/letsencrypt. You should make a
    secure backup of this folder now. This configuration directory will
    also contain certificates and private keys obtained by Certbot so
    making regular backups of this folder is ideal.
    - If you like Certbot, please consider supporting our work by:
    Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate
    Donating to EFF: https://eff.org/donate-le

Une fois que cette étape est faite, on peut passer à l'étape suivante.

## Configuration du HTTPS

Utilisé ___Nextcloud___ sans utilisé la sécurité du HTTPS 

Nous allons redirigé tous le traffic non crypté vers le HTTPS, pour cela il faut ajouté dans le fichier __nextcloud.conf__.

    <VirtualHost *:80>
        ServerName cloud.nextcloud.com
        Redirect permanent / https://cloud.nextcloud.com/
    </VirtualHost>


    <VirtualHost *:443>
        ServerName cloud.nextcloud.com
        <IfModule mod_headers.c>
            Header always set Strict-Transport-Security "max-age=15552000;preload; includeSubDomains"
        </IfModule>
    </VirtualHost>

Ajouter dans le fichier .htaccess

    Header always set Strict-Transport-Security "max-age=15552000;preload;

Configuration du cache mémoire

    opcache.enable=1
    opcache.interned_strings_buffer=8
    opcache.max_accelerated_files=10000
    opcache.memory_consumption=128
    opcache.save_comments=1
    opcache.revalidate_freq=1

Et ajouter dans le fichier config.php

    'memcache.local' => '\OC\Memcache\APCu',

## 6. Mise à jours de Nextcloud

Pour effectuer la mise à jour de __NextCloud__, avant cela il faudra avoir fait au préalable un backup du __NextCloud__. 


## Installation de Docker

    apt install apt-transport-https ca-certificates curl software-properties-common gnupg2


    curl -fsSl https://download.docker.com/linux/debian/gpg | apt-key add -

    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

    apt update

    apt install docker-ce

    systemctl status docker
