
# G3W installazione via Docker

seguire la documentazione presente sul repository: https://github.com/g3w-suite/g3w-suite-docker  

 ## installare Docker su Ubuntu 22
 
 https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04  
 
 ```
 sudo apt install apt-transport-https ca-certificates curl software-properties-common
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 apt update
 apt-cache policy docker-ce
 sudo apt install docker-ce
 ```
 
### Installare docker-compose

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04  

```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose

```
## Installare G3W

https://github.com/g3w-suite/g3w-client
https://github.com/g3w-suite/g3w-suite-docker/tree/v3.5.x  

```
cd /opt/
git clone https://github.com/g3w-suite/g3w-suite-docker.git
git clone https://github.com/g3w-suite/g3w-admin.git --single-branch --branch dev ./g3w-admin
git clone https://github.com/g3w-suite/g3w-client.git --single-branch --branch dev ./g3w-client
```

### editare il file .env.example 

```
# Public hostmane
WEBGIS_PUBLIC_HOSTNAME=g3w.gis-ninja.eu

# Shared volume mount (docker internal: shared-volume)
# I suggest not to use the /tmp/ folder, /tmp/ folder is cleaned at each reboot
WEBGIS_DOCKER_SHARED_VOLUME=./shared-volume

# Docker internal DB
G3WSUITE_POSTGRES_USER_LOCAL=g3wsuite
G3WSUITE_POSTGRES_PASS='89#kL8y3D'
G3WSUITE_POSTGRES_DBNAME=g3wsuite
G3WSUITE_POSTGRES_HOST=postgis
G3WSUITE_POSTGRES_PORT=5432

# Set this to true to activate the frontend module
FRONTEND=True

# QGIS Server env variables
# ----------------------------------------------------
PGSERVICEFILE=/pg_service/pg_service.conf
# Put your pg service into ./scripts/pg_service.conf file, the conf file will be mounted into
# docker container at runtime a the PGSERVICEFILE

QGIS_SERVER_LOG_FILE=/shared_volume/QGIS/error.log
QGIS_SERVER_LOG_LEVEL=2

# Extra QGIS env variables.
# Every QGIS environment variables available as specified on manual
# https://docs.qgis.org/3.22/en/docs/server_manual/config.html#environment-variables
# can be set, important is remember to add thy to `docker-compose.yml` o  `docker-compose-consument.yml`.

# Openrouteservice
# ----------------------------------------------------
#ORS_API_ENDPOINT='https://api.openrouteservice.org/v2'
# Optional, can be blank if the key is not required by the endpoint
#ORS_API_KEY='<your API key goes here>'
# Max number of ranges (it depends on the server configuration)
#ORS_MAX_RANGES=6
# Max number of locations(it depends on the server configuration)
#ORS_MAX_LOCATIONS=2

# Gunicorn settings
# ---------------------------------------------------------
# Timeout for Gunicorn instances. In some cases you may consider increasing this value to 240 or higher to prevent 502 errors in nginx.
G3WSUITE_GUNICORN_TIMEOUT=120

# *Important*: remember to add this env vars also to docker-compose.yml or
# docker-compose-consumer.yml


G3WSUITE_LOCAL_CODE_PATH=../g3w-admin
G3WSUITE_DEBUG=True

```

### editare il file docker-compose.yml esponendo la porta 80

```
version: "3"
services:
  postgis:
    image: g3wsuite/postgis:11.0-2.5
    ports:
      - "5439:5432"
    environment:
      - POSTGRES_DBNAME=${G3WSUITE_POSTGRES_DBNAME},data_testing,data_production
      - POSTGRES_USER=${G3WSUITE_POSTGRES_USER_LOCAL}
      - POSTGRES_PASS=${G3WSUITE_POSTGRES_PASS}
      - ALLOW_IP_RANGE=0.0.0.0/0
    restart: on-failure
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
    volumes:
      - ${WEBGIS_DOCKER_SHARED_VOLUME}:/var/lib/postgresql
    healthcheck:
      interval: 60s
      timeout: 30s
      retries: 3
      test: "pg_isready"
    networks:
      internal:

  g3w-suite:
    image: g3wsuite/g3w-suite:dev
    environment:
      - DISPLAY=:99
      - G3WSUITE_TILECACHE_PATH
      - G3WSUITE_POSTGRES_DBNAME
      - G3WSUITE_POSTGRES_USER
      - G3WSUITE_POSTGRES_USER_LOCAL
      - G3WSUITE_POSTGRES_PASS
      - G3WSUITE_POSTGRES_HOST
      - G3WSUITE_POSTGRES_PORT
      - G3WSUITE_ORS_API_ENDPOINT
      - G3WSUITE_ORS_API_KEY
      - TILESTACHE_CACHE_BUFFER_SIZE
      - TILESTACHE_CACHE_TOKEN
      - G3WSUITE_GUNICORN_NUM_WORKERS
      - G3WSUITE_GUNICORN_MAX_REQUESTS
      - G3WSUITE_GUNICORN_TIMEOUT
      - FRONTEND
      - PGSERVICEFILE
      - QGIS_OPTIONS_PATH=/shared-volume/
      - QGIS_SERVER_LOG_FILE
      - QGIS_SERVER_LOG_LEVEL
    expose:
      - "8000"
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"

    depends_on:
      - postgis
    volumes:
      - ${WEBGIS_DOCKER_SHARED_VOLUME}:/shared-volume
      - ${WEBGIS_DOCKER_SHARED_VOLUME}/node_modules:/code/node_modules
      - ./config/g3w-suite/overrides/static:/code/static:ro
      - ./config/g3w-suite/overrides/templates:/code/templates:ro
      - ./config/g3w-suite/settings_docker.py:/code/g3w-admin/base/settings/local_settings.py
      - ./secrets/pg_service.conf:${PGSERVICEFILE}
      - ./config/qgis/QGIS3.ini:/shared-volume/QGIS/QGIS3.ini

    networks:
      internal:

  nginx:
    image: nginx
    ports:
      - "80:8080"
      - "443:443"
    expose:
      - "8080"
    volumes:
      - ${WEBGIS_DOCKER_SHARED_VOLUME}:/shared-volume
      - ${WEBGIS_DOCKER_SHARED_VOLUME}/var/www/.well-known:/var/www/.well-known
      - ${WEBGIS_DOCKER_SHARED_VOLUME}/certs/letsencrypt:/etc/letsencrypt:ro
      - ./config/g3w-suite/overrides/static:/shared-volume/static/overrides:ro
      - ./config/nginx:/etc/nginx/conf.d:ro
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
    restart: always
    networks:
      internal:

  # Letsencrypt certs
  certbot:
    image: certbot/certbot
    volumes:
      - ${WEBGIS_DOCKER_SHARED_VOLUME}/var/www/certbot:/var/www/certbot
      - ${WEBGIS_DOCKER_SHARED_VOLUME}/certs/letsencrypt:/etc/letsencrypt

volumes:
  shared-volume:
  g3wsuite-pg-data:

networks:
  internal:

```

### editare il file django.conf in /config/nginx

editare il file andando ad inserire il dominio corretto

## Installare

```
docker compose -f docker-compose.yml up -d

```


### HTTPS

check the domain name in the `.env` file and in `config/nginx/django_ssl.conf`  
nel file `django_ssl.conf` controllare il percorso dei file di letsencrypt

```
run: docker pull certbot/certbot
launch ./run_certbot.sh
activate 301 redirect into config/nginx/django.conf
move config/_nginx/django_ssl.conf to config/nginx/django_ssl.conf
docker compose down
docker compose -f docker-compose.yml up -d

```

## TIPS
Per installare font aggiuntivi per la simbologia entrare nel container della g3w suite

`docker exec -ti  g3wsuite/g3w-suite:dev bash`
 
 ```
  cd usr/local/share/
  cd fonts/
  apt install fontconfig
  fc-cache -f -v
  fc-list | grep "ESRI SDS 1.95 1 Regular"
  cd usr/local/share/fonts/
  git clone https://github.com/fgianoli/fonts.git
  cd fonts/
  fc-cache -f -v
  fc-list
  exit
```

