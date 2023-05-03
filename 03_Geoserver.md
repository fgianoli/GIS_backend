
# INSTALLARE GEOSERVER VIA DOCKER

Useremo l'immagine docker fornita da Kartoza disponibile qui: https://github.com/kartoza/docker-geoserver

Installiamo Geoserver sotto la cartella `/opt`

```
cd /opt
git clone https://github.com/kartoza/docker-geoserver.git
cd docker-geoserver
mkdir geoserver_data
```
### Editare il file `.env`

```
COMPOSE_PROJECT_NAME=kartozageoserver

IMAGE_VERSION=9.0.65-jdk11-openjdk-slim-buster
GS_VERSION=2.22.0
GEOSERVER_PORT=8600
# Build Arguments
JAVA_HOME=/usr/local/openjdk-11
WAR_URL=http://downloads.sourceforge.net/project/geoserver/GeoServer/2.22.0/geoserver-2.22.0-war.zip
STABLE_PLUGIN_BASE_URL=https://sourceforge.net/projects/geoserver/files/GeoServer
DOWNLOAD_ALL_STABLE_EXTENSIONS=1
DOWNLOAD_ALL_COMMUNITY_EXTENSIONS=1
FORCE_DOWNLOAD_STABLE_EXTENSIONS=false
FORCE_DOWNLOAD_COMMUNITY_EXTENSIONS=false
GEOSERVER_UID=1000
GEOSERVER_GID=10001
RECREATE_DATADIR=TRUE
# Generic Env variables
GEOSERVER_ADMIN_USER=admin
GEOSERVER_ADMIN_PASSWORD=geoserver
# Reset admin credentials on container restart
RESET_ADMIN_CREDENTIALS=FALSE
# https://docs.geoserver.org/latest/en/user/datadirectory/setting.html
GEOSERVER_DATA_DIR=/opt/geoserver/data_dir
# https://docs.geoserver.org/latest/en/user/data/raster/gdal.html#external-footprints-data-directory
FOOTPRINTS_DATA_DIR=/opt/footprints_dir
# https://docs.geoserver.org/latest/en/user/geowebcache/config.html#changing-the-cache-directory
GEOWEBCACHE_CACHE_DIR=/opt/geoserver/data_dir/gwc
# Show the tomcat manager in the browser
TOMCAT_EXTRAS=true
# Redirect to GeoServer web interface
ROOT_WEBAPP_REDIRECT=false
# https://docs.geoserver.org/stable/en/user/production/container.html#optimize-your-jvm
INITIAL_MEMORY=4G
# https://docs.geoserver.org/stable/en/user/production/container.html#optimize-your-jvm
MAXIMUM_MEMORY=4G
INITIAL_HEAP_OCCUPANCY_PERCENT=45
# Path where .ttf and otf font should be added
FONTS_DIR=/opt/fonts
# JVM Startup option for encoding
ENCODING='UTF8'
# JVM Startup option for timezone
TIMEZONE='GMT'
# Additional JVM startup options not specified in https://github.com/kartoza/docker-geoserver/blob/master/scripts/entrypoint.sh#L21
# Example ADDITIONAL_JAVA_STARTUP_OPTIONS='-Dorg.geotools.shapefile.datetime=true'
ADDITIONAL_JAVA_STARTUP_OPTIONS=
# DB backend to activate disk quota storage in PostgreSQL DB. Only permitted value is 'POSTGRES'
DB_BACKEND=
# https://docs.geoserver.org/latest/en/user/production/config.html#disable-the-auto-complete-on-web-administration-interface-login
LOGIN_STATUS=on
# https://docs.geoserver.org/latest/en/user/production/config.html#disable-the-geoserver-web-administration-interface
DISABLE_WEB_INTERFACE=false
# Rendering settings
ENABLE_JSONP=true
MAX_FILTER_RULES=20
OPTIMIZE_LINE_WIDTH=false
# Install the stable plugin specified in https://github.com/kartoza/docker-geoserver/blob/master/build_data/stable_plugins.txt
STABLE_EXTENSIONS=
# Install the community edition plugins specified in https://github.com/kartoza/docker-geoserver/blob/master/build_data/community_plugins.txt
COMMUNITY_EXTENSIONS=
# SSL Settings explained here https://github.com/AtomGraph/letsencrypt-tomcat
SSL=false
HTTP_PORT=8080
HTTP_PROXY_NAME=
HTTP_PROXY_PORT=
HTTP_REDIRECT_PORT=
HTTP_CONNECTION_TIMEOUT=20000
HTTPS_PORT=8443
HTTPS_MAX_THREADS=150
HTTPS_CLIENT_AUTH=
HTTPS_PROXY_NAME=
HTTPS_PROXY_PORT=
JKS_FILE=letsencrypt.jks
JKS_KEY_PASSWORD='geoserver'
KEY_ALIAS=letsencrypt
JKS_STORE_PASSWORD='geoserver'
P12_FILE=letsencrypt.p12
PKCS12_PASSWORD='geoserver'
LETSENCRYPT_CERT_DIR=/etc/letsencrypt
CHARACTER_ENCODING='UTF-8'
# Clustering  variables
# Activates clustering using JMS cluster plugin
CLUSTERING=False
# cluster env variables specified https://docs.geoserver.org/stable/en/user/community/jms-cluster/index.html
CLUSTER_DURABILITY=true
BROKER_URL=
READONLY=disabled
RANDOMSTRING=23bd87cfa327d47e
INSTANCE_STRING=ac3bcba2fa7d989678a01ef4facc4173010cd8b40d2e5f5a8d18d5f863ca976f
TOGGLE_MASTER=true
TOGGLE_SLAVE=true
EMBEDDED_BROKER=enabled
# kartoza/postgis env variables https://github.com/kartoza/docker-postgis
POSTGIS_VERSION_TAG=14-3.1
POSTGRES_DB=SIT_Vicenza
POSTGRES_USER=postgres
POSTGRES_PASS=db_vicenza2014
ALLOW_IP_RANGE=0.0.0.0/0
POSTGRES_PORT=5432


```


### Editare il file `docker-compose.yml`

```yml

version: '3.9'

services:
   geoserverfg:
      image: kartoza/geoserver:${GS_VERSION}
      volumes:
        - ./geoserver-data:/opt/geoserver/data_dir
        
      # connettere il docker network con il network della macchina host (localhost)
      # https://stackoverflow.com/questions/70958724/connect-to-postgres-as-systemd-service-from-docker-compose
      network_mode: host
      restart: on-failure
      environment:
        - GEOSERVER_DATA_DIR=${GEOSERVER_DATA_DIR}
        - GEOWEBCACHE_CACHE_DIR=${GEOWEBCACHE_CACHE_DIR}
        - GEOSERVER_ADMIN_PASSWORD=${GEOSERVER_ADMIN_PASSWORD}
        - GEOSERVER_ADMIN_USER=${GEOSERVER_ADMIN_USER}
        - INITIAL_MEMORY=${INITIAL_MEMORY}
        - MAXIMUM_MEMORY=${MAXIMUM_MEMORY}
        - STABLE_EXTENSIONS=${STABLE_EXTENSIONS}
        - COMMUNITY_EXTENSIONS=${COMMUNITY_EXTENSIONS}
        # Aggiungere
        - DISK_QUOTA_SIZE=5
        - DISK_QUOTA_FREQUENCY=600
        - EXISTING_DATA_DIR=true
        #           
        - HOST=localhost
      healthcheck:
        test: "curl --fail --silent --write-out 'HTTP CODE : %{http_code}\n' --output /dev/null -u ${GEOSERVER_ADMIN_USER}:'${GEOSERVER_ADMIN_PASSWORD}' http://localhost:8080/geoserver/rest/about/version.xml"
        interval: 1m30s
        timeout: 10s
        retries: 3
```

## Lanciare installazione

```
sudo docker-compose -f docker-compose.yml up -d
```

