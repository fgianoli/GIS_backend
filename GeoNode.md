# INSTALLARE GEONODE VIA DOCKER

https://docs.geonode.org/en/master/install/basic/index.html#ubuntu-20-04-basic-setup  

## Preparare la macchina 

Per prima cosa installare sulla macchina le librerie

```sh
sudo add-apt-repository ppa:ubuntugis/ppa
sudo apt update -y

sudo apt install -y python3-gdal=3.3.2+dfsg-2~focal2 gdal-bin=3.3.2+dfsg-2~focal2 libgdal-dev=3.3.2+dfsg-2~focal2 ####### QUESTO NO
sudo apt install -y python3-pip python3-dev python3-virtualenv python3-venv virtualenvwrapper
sudo apt install -y libxml2 libxml2-dev gettext
sudo apt install -y libxslt1-dev libjpeg-dev libpng-dev libpq-dev
sudo apt install -y software-properties-common build-essential
sudo apt install -y git unzip gcc zlib1g-dev libgeos-dev libproj-dev
sudo apt install -y sqlite3 spatialite-bin libsqlite3-mod-spatialite
```

Successivamente

```
sudo add-apt-repository universe
sudo apt-get update -y
sudo apt-get install -y git-core git-buildpackage debhelper devscripts python3.10-dev python3.10-venv virtualenvwrapper
sudo apt-get install -y apt-transport-https ca-certificates curl lsb-release gnupg gnupg-agent software-properties-common vim

```

## Installare GeoNode

```sh
cd /opt

sudo git clone https://github.com/GeoNode/geonode-project.git -b 4.1.x


```
