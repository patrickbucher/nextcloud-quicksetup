# Nextcloud-Installation

Ausgangslage: Debian 12 "Bookworm" (Standardinstallation)

## Apache

Den Apache-Webserver installieren:

    $ sudo apt install -y apache2

Überprüfen:

    $ systemctl is-active apache2

Zum Testen auf [http://localhost](http://localhost) zugreifen.

## PHP

Die Standardseite deaktivieren:

    $ sudo a2dissite 000-default.conf

PHP installieren:

    $ sudo apt install -y php8.2

Das Verzeichnis `/var/www/demo` erstellen und berechtigen:

    $ sudo mkdir /var/www/demo
    $ sudo chown -R www-data:www-data /var/www/demo

Eine PHP-Infoseite erstellen (`/var/www/demo/index.php`):

    <?php
        phpinfo();
    ?>

Eine neue Testseite einrichten (`/etc/apache2/sites-available/demo.conf`):

    <VirtualHost *:80>
        DocumentRoot /var/www/demo
        ServerName localhost
    </VirtualHost>

Die PHP-Infoseite aktivieren:

    $ sudo a2ensite demo.conf

Die Serverkonfiguration neu laden:

    $ sudo systemctl reload apache2.service

Zum Testen auf [http://localhost](http://localhost) zugreifen (Server API: Apache 2.0 Handler).

## PHP-FPM

PHP-FPM mit passendem Apache-Modul (FastCGI) installieren:

    $ sudo apt install -y php8.2-fpm libapache2-mod-fcgid

Das Apache-Modul `mod_php` deaktivieren:

    $ sudo a2dismod php8.2

Das Apache-Modul `proxy_fcgi` mit der dazugehörigen Konfiguration aktivieren:

    $ sudo a2enmod proxy_fcgi
    $ sudo a2enconf php8.2-fpm

Apache neu starten:

    $ sudo systemctl restart apache.service

Zum Testen auf [http://localhost](http://localhost) zugreifen (Server API: FPM/FastCGI).

Memory-Zuordnung für PHP-Skripte erhöhen (`/etc/php/8.2/fpm/php.ini`):

    memory_limit = 256M

PHP-FPM neu starten:

    $ sudo systemctl restart php8.2-fpm.service

Zum Testen auf [http://localhost](http://localhost) zugreifen (Memory-Limite: 256M).

Infoseite deaktivieren:

    $ sudo a2dissite demo.conf

## MariaDB

MariaDB installieren:

    $ sudo apt install -y mariadb-server

MariaDB härten:

    $ sudo mariadb-secure-installation

Optionen:

    [Enter]
    n
    n
    Y
    Y
    Y
    Y

Datenbank erstellen, Benutzer erstellen und für die Datenbank berechtigen:

    $ sudo mariadb
    > CREATE USER nextcloud@localhost IDENTIFIED BY 'topsecret';
    > CREATE DATABASE nextcloud CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
    > GRANT ALL PRIVILEGES ON nextcloud.* TO nextcloud@localhost;
    > FLUSH PRIVILEGES;
    > EXIT

## Nextcloud

Apache-Konfiguration erstellen (`/etc/apache2/sites-available/nextcloud.conf`):

    <VirtualHost *:80>
        DocumentRoot /var/www/nextcloud
        ServerName localhost
        <Directory /var/www/nextcloud>
            Require all granted
            AllowOverride All
            Options FollowSymLinks MultiViews
            <IfModule mod_dav.c>
                Dav off
            </IfModule>
        </Directory>
    </VirtualHost>

Nextcloud herunterladen:

- [nextcloud.com](https://nextcloud.com)
- Get Nextcloud
- Nextcloud server
- Community Projects
- Archive
    1. archive: `.tar.bz2` 
    2. SHA256 checksum: für `.tar.bz2` 
    3. PGP Signature: für `.tar.bz2`
    4. PGP Key: `.asc`

Integrität prüfen:

    $ cd Downloads/
    $ sha256sum -c latest.tar.bz2.sha256
    OK

Schlüssel importieren:

    $ gpg --import nextcloud.asc

Signatur prüfen:

    $ gpg --verify latest.tar.bz2.asc latest.tar.bz2

Nextcloud-Archiv nach `/var/www` entpacken:

    $ sudo tar xf latest.tar.bzw -C /var/www

Verzeichnis berechtigen:

    $ sudo chown -R www-data:www-data /var/www/nextcloud

Seite aktivieren:

    $ sudo a2ensite nextcloud.conf

Apache-Konfiguration neu laden:

    $ sudo systemctl reload apache2.service

Nextcloud-Seite unter [http://localhost](http://localhost) laden (Fehlermeldungen).

Fehlermeldungen beheben:

    $ sudo apt install -y php8.2-mysql php8.2-zip php8.2-dom php8.2-mbstring php8.2-gd php8.2-curl

Einstellungen:

- Benutzername: `admin`
- Passwort: `mostsecret`
- Speicherort: [belassen]
- Datenbank-Benutzer: `nextcloud`
- Datenbank-Passwort: `topsecret`
- Datenbank-Name: `nextcloud`
- Datenbank-Name: [belassen]
- Installieren: [betätigen]