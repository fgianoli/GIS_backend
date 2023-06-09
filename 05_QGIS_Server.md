# INSTALLARE QGIS SERVER

https://www.qgis.org/it/site/forusers/alldownloads.html

```
apt install apache2

sudo apt install gnupg software-properties-common

sudo mkdir -m755 -p /etc/apt/keyrings  # not needed since apt version 2.4.0 like Debian 12 and Ubuntu 22 or newer
sudo wget -O /etc/apt/keyrings/qgis-archive-keyring.gpg https://download.qgis.org/downloads/qgis-archive-keyring.gpg
```

editare `/etc/apt/sources.list.d/qgis.sources`

```
Types: deb deb-src
URIs: https://qgis.org/debian
Suites: jammy
Architectures: amd64
Components: main
Signed-By: /etc/apt/keyrings/qgis-archive-keyring.gpg
```
Suites, nelle righe precedenti, dipende dalla tua distribuzione. il comando `lsb_release -cs` mostrerà il nome della tua distribuzione.

```
sudo apt update
sudo apt-cache policy qgis-server

sudo apt install -y qgis-server python-qgis libapache2-mod-fcgid
```
Abilitare QGIS Server

```
a2enmod fcgid
a2enconf serve-cgi-bin
sudo a2enmod rewrite
sudo service apache2 restart
```

editare il file 000-default.conf di apache aggiungendo queste righe dopo la definizione della directory
`sudo nano /etc/apache2/sites-enabled/000-default.conf`

aggiungendo:
```
FcgidIOTimeout 120

ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
<Directory "/usr/lib/cgi-bin/">
  AllowOverride All
  Options +ExecCGI -MultiViews +FollowSymLinks
  AddHandler fcgid-script .fcgi
  Require all granted
</Directory>
```

Riavviare Apache2
`sudo service apache2 restart`

Per testare il corretto funzionamento 
http://localhost:8000/cgi-bin/qgis_mapserv.fcgi?SERVICE=WMS&VERSION=1.3.0&REQUEST=GetCapabilities
