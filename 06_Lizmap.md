# INSTALLARE LIZMAP

https://docs.lizmap.com/3.6/it/install/linux.html

## Installare PHP
```
sudo add-apt-repository -y ppa:ondrej/php && sudo apt update && apt install -y php7.4-mbstring php7.4-zip php7.4-xml

apt install php libapache2-mod-php
apt install php7.4-fpm php7.4-cli php7.4-bz2 php7.4-curl php7.4-gd php7.4-intl php7.4-json php7.4-mbstring php7.4-pgsql php7.4-sqlite3 php7.4-xml php7.4-ldap php7.4-redis
apt install xvfb
apt-get install php-sqlite3
sudo apt-get install libsqlite3-0 libsqlite3-dev
apt-get install php7.4-sqlite3
apt-get install php-xml

```

Controllare la versione del release di Lizmap da https://github.com/3liz/lizmap-web-client/releases/ 
```sh
cd /var/www
```
controlla
```sh
cd $LOCATION
wget https://github.com/3liz/lizmap-web-client/releases/download/3.6/lizmap-web-client-3.6.zip
# Unzip archive
unzip lizmap-web-client-3.6.zip

# virtual link for http://localhost/lizmap/
ln -s /var/www/lizmap-web-client-3.6/lizmap/www/ /var/www/html/lizmap
# Remove archive
rm lizmap-web-client-3.6.zip

cd /var/www/lizmap-web-client-3.6/
lizmap/install/set_rights.sh www-data www-data
```

Create lizmapConfig.ini.php, localconfig.ini.php e modificateli per impostare i parametri specifici della vostra installazione. Potete modificare lizmapConfig.ini.php per impostare l’url di qgis map server e altre cose.

```
cd lizmap/var/config
cp lizmapConfig.ini.php.dist lizmapConfig.ini.php
cp localconfig.ini.php.dist localconfig.ini.php
cp profiles.ini.php.dist profiles.ini.php

php lizmap/install/configurator.php
php lizmap/install/installer.php
```

Rieditare il file di apache

`nano /etc/apache2/sites-enabled/000-default.conf`
```
            Alias /lizmap /var/www/lizmap/lizmap-web-client/lizmap/www/
            <Directory "/var/www/lizmap/lizmap-web-client/lizmap/www/">
                Options -Indexes +FollowSymLinks +ExecCGI
                AllowOverride All
                Require all granted
            </Directory>
```
Riavviare apache `sudo service apache2 restart`
Assicurarsi di aver dato i permessi corretti alle cartelle

```
cd var/www/lizmap
chown -R www-data:www-data lizmap
```
