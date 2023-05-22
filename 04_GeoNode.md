# INSTALLARE GEONODE VIA DOCKER

[GUIDA DOCUMENTAZIONE UFFICIALE](https://docs.geonode.org/en/master/install/basic/index.html)  

## Preparare la macchina 

Per prima cosa installare sulla macchina le librerie

```sh
sudo add-apt-repository universe
sudo apt-get update -y
sudo apt install -y git git-buildpackage debhelper devscripts python3.10-dev python3.10-venv virtualenvwrapper
sudo apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common vim
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Se non è installato docker installarlo

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
apt-cache policy docker-ce
sudo apt install -y docker-ce
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
docker compose version

```

## Installare GeoNode

```sh
cd /opt
## Controllare la release di GeoNode
git clone https://github.com/GeoNode/geonode-project.git -b 4.1.x
source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
mkvirtualenv --python=/usr/bin/python3 my_geonode
pip install Django==3.2.13
django-admin startproject --template=./geonode-project -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile my_geonode

cd my_geonode/
nano docker-build.sh #cambiare il comando da docker-compose a doker compose
python create-envfile.py
./docker-build.sh
docker ps -a
```

### Esempio file .env
```
COMPOSE_PROJECT_NAME=my_geonode
DOCKERHOST=
DOCKER_HOST_IP=
# See https://github.com/containers/podman/issues/13889
# DOCKER_BUILDKIT=0
DOCKER_ENV=production
# See https://github.com/geosolutions-it/geonode-generic/issues/28
# to see why we force API version to 1.24
DOCKER_API_VERSION="1.24"
BACKUPS_VOLUME_DRIVER=local

C_FORCE_ROOT=1
FORCE_REINIT=false
INVOKE_LOG_STDOUT=true

# LANGUAGE_CODE=pt
# LANGUAGES=(('en','English'),('pt','Portuguese'))

DJANGO_SETTINGS_MODULE=my_geonode.settings
GEONODE_INSTANCE_NAME=geonode

# #################
# backend
# #################
POSTGRES_USER=postgres
POSTGRES_PASSWORD=federico
GEONODE_DATABASE=my_geonode
GEONODE_DATABASE_PASSWORD=federico
GEONODE_GEODATABASE=my_geonode_data
GEONODE_GEODATABASE_PASSWORD=federico
GEONODE_DATABASE_SCHEMA=public
GEONODE_GEODATABASE_SCHEMA=public
DATABASE_HOST=db
DATABASE_PORT=5432
DATABASE_URL=postgis://my_geonode:federico@db:5432/my_geonode
GEODATABASE_URL=postgis://my_geonode_data:federico@db:5432/my_geonode_data
GEONODE_DB_CONN_MAX_AGE=0
GEONODE_DB_CONN_TOUT=5
DEFAULT_BACKEND_DATASTORE=datastore
BROKER_URL=amqp://guest:guest@rabbitmq:5672/
CELERY_BEAT_SCHEDULER=celery.beat:PersistentScheduler
ASYNC_SIGNALS=True

SITEURL=http://localhost/

ALLOWED_HOSTS="['django', '*', 'localhost']"

# Data Uploader
DEFAULT_BACKEND_UPLOADER=geonode.importer
TIME_ENABLED=True
MOSAIC_ENABLED=False
HAYSTACK_SEARCH=False
HAYSTACK_ENGINE_URL=http://elasticsearch:9200/
HAYSTACK_ENGINE_INDEX_NAME=haystack
HAYSTACK_SEARCH_RESULTS_PER_PAGE=200

# #################
# Jenkins
# CI/CD Server
# #################
JENKINS_HTTP_PORT=9080
JENKINS_HTTPS_PORT=9443

# #################
# nginx
# HTTPD Server
# #################
GEONODE_LB_HOST_IP=localhost
GEONODE_LB_PORT=80
PUBLIC_PORT=80

# IP or domain name and port where the server can be reached on HTTPS (leave HOST empty if you want to use HTTP only)
# port where the server can be reached on HTTPS
HTTP_HOST=localhost
HTTPS_HOST=

HTTP_PORT=80
HTTPS_PORT=443

# Let's Encrypt certificates for https encryption. You must have a domain name as HTTPS_HOST (doesn't work
# with an ip) and it must be reachable from the outside. This can be one of the following :
# disabled : we do not get a certificate at all (a placeholder certificate will be used)
# staging : we get staging certificates (are invalid, but allow to test the process completely and have much higher limit rates)
# production : we get a normal certificate (default)
LETSENCRYPT_MODE=disabled
# LETSENCRYPT_MODE=staging
# LETSENCRYPT_MODE=production

RESOLVER=127.0.0.11

# #################
# geoserver
# #################
GEOSERVER_WEB_UI_LOCATION=http://localhost/geoserver/
GEOSERVER_PUBLIC_LOCATION=http://localhost/geoserver/
GEOSERVER_LOCATION=http://geoserver:8080/geoserver/
GEOSERVER_ADMIN_USER=admin
GEOSERVER_ADMIN_PASSWORD=federico

OGC_REQUEST_TIMEOUT=30
OGC_REQUEST_MAX_RETRIES=1
OGC_REQUEST_BACKOFF_FACTOR=0.3
OGC_REQUEST_POOL_MAXSIZE=10
OGC_REQUEST_POOL_CONNECTIONS=10

# Java Options & Memory
ENABLE_JSONP=true
outFormat=text/javascript
GEOSERVER_JAVA_OPTS=-Djava.awt.headless=true -Xms2G -Xmx4G -Dgwc.context.suffix=gwc -XX:+UnlockDiagnosticVMOptions -XX:+LogVMOutput -XX:LogFile=/var/log/jvm.log -XX:PerfDataSamplingInterval=500 -XX:SoftRefLRUPolicyMSPerMB=36000 -XX:-UseGCOverheadLimit -XX:ParallelGCThreads=4 -Dfile.encoding=UTF8 -Djavax.servlet.request.encoding=UTF-8 -Djavax.servlet.response.encoding=UTF-8 -Duser.timezone=GMT -Dorg.geotools.shapefile.datetime=false -DGEOSERVER_CSRF_DISABLED=true -DPRINT_BASE_URL=http://localhost/geoserver/pdf -DALLOW_ENV_PARAMETRIZATION=true -Xbootclasspath/a:/usr/local/tomcat/webapps/geoserver/WEB-INF/lib/marlin-0.9.3-Unsafe.jar -Dsun.java2d.renderer=org.marlin.pisces.MarlinRenderingEngine

# #################
# Security
# #################
# Admin Settings
#
# ADMIN_PASSWORD is used to overwrite the GeoNode admin password **ONLY** the first time
# GeoNode is run. If you need to overwrite it again, you need to set the env var FORCE_REINIT,
# otherwise the invoke updateadmin task will be skipped and the current password already stored
# in DB will honored.

ADMIN_USERNAME=admin
ADMIN_PASSWORD=federico
ADMIN_EMAIL=gianoli.federico@gmail.com

# EMAIL Notifications
EMAIL_ENABLE=False
DJANGO_EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
DJANGO_EMAIL_HOST=localhost
DJANGO_EMAIL_PORT=25
DJANGO_EMAIL_HOST_USER=
DJANGO_EMAIL_HOST_PASSWORD=
DJANGO_EMAIL_USE_TLS=False
DJANGO_EMAIL_USE_SSL=False
DEFAULT_FROM_EMAIL='gianoli.federico@gmail.com' # eg Company <no-reply@company.org>

# Session/Access Control
LOCKDOWN_GEONODE=False
X_FRAME_OPTIONS="SAMEORIGIN"
SESSION_EXPIRED_CONTROL_ENABLED=True
DEFAULT_ANONYMOUS_VIEW_PERMISSION=True
DEFAULT_ANONYMOUS_DOWNLOAD_PERMISSION=True

CORS_ALLOW_ALL_ORIGINS=True
GEOSERVER_CORS_ENABLED=True
GEOSERVER_CORS_ALLOWED_ORIGINS=*
GEOSERVER_CORS_ALLOWED_METHODS=GET,POST,PUT,DELETE,HEAD,OPTIONS
GEOSERVER_CORS_ALLOWED_HEADERS=*

# Users Registration
ACCOUNT_OPEN_SIGNUP=True
ACCOUNT_EMAIL_REQUIRED=True
ACCOUNT_APPROVAL_REQUIRED=False
ACCOUNT_CONFIRM_EMAIL_ON_GET=False
ACCOUNT_EMAIL_VERIFICATION=none
ACCOUNT_EMAIL_CONFIRMATION_EMAIL=False
ACCOUNT_EMAIL_CONFIRMATION_REQUIRED=False
ACCOUNT_AUTHENTICATION_METHOD=username_email
AUTO_ASSIGN_REGISTERED_MEMBERS_TO_REGISTERED_MEMBERS_GROUP_NAME=True

# OAuth2
OAUTH2_API_KEY=
OAUTH2_CLIENT_ID=pRyvrrW1xJTgaWh
OAUTH2_CLIENT_SECRET=tfzjO41dZbUknPo

# GeoNode APIs
API_LOCKDOWN=False
TASTYPIE_APIKEY=

# #################
# Production and
# Monitoring
# #################
DEBUG=False

SECRET_KEY="V];VpY~/R5TQc,q2{eDBIa?{m>fD:D=VAOk>_d>pt+\TsH$izW"

STATIC_ROOT=/mnt/volumes/statics/static/
MEDIA_ROOT=/mnt/volumes/statics/uploaded/
GEOIP_PATH=/mnt/volumes/statics/geoip.db

CACHE_BUSTING_STATIC_ENABLED=False

MEMCACHED_ENABLED=False
MEMCACHED_BACKEND=django.core.cache.backends.memcached.MemcachedCache
MEMCACHED_LOCATION=127.0.0.1:11211
MEMCACHED_LOCK_EXPIRE=3600
MEMCACHED_LOCK_TIMEOUT=10

MAX_DOCUMENT_SIZE=2
CLIENT_RESULTS_LIMIT=5
API_LIMIT_PER_PAGE=1000

# GIS Client
GEONODE_CLIENT_LAYER_PREVIEW_LIBRARY=mapstore
MAPBOX_ACCESS_TOKEN=
BING_API_KEY=
GOOGLE_API_KEY=

# Monitoring
MONITORING_ENABLED=True
MONITORING_DATA_TTL=365
USER_ANALYTICS_ENABLED=True
USER_ANALYTICS_GZIP=True
CENTRALIZED_DASHBOARD_ENABLED=False
MONITORING_SERVICE_NAME=local-geonode
MONITORING_HOST_NAME=geonode

# Other Options/Contribs
MODIFY_TOPICCATEGORY=True
AVATAR_GRAVATAR_SSL=True
EXIF_ENABLED=True
CREATE_LAYER=True
FAVORITE_ENABLED=True

# Advanced Workflow
RESOURCE_PUBLISHING=False
ADMIN_MODERATE_UPLOADS=False

# LDAP
LDAP_ENABLED=False
LDAP_SERVER_URL=ldap://<the_ldap_server>
LDAP_BIND_DN=uid=ldapinfo,cn=users,dc=ad,dc=example,dc=org
LDAP_BIND_PASSWORD=<something_secret>
LDAP_USER_SEARCH_DN=dc=ad,dc=example,dc=org
LDAP_USER_SEARCH_FILTERSTR=(&(uid=%(user)s)(objectClass=person))
LDAP_GROUP_SEARCH_DN=cn=groups,dc=ad,dc=example,dc=org
LDAP_GROUP_SEARCH_FILTERSTR=(|(cn=abt1)(cn=abt2)(cn=abt3)(cn=abt4)(cn=abt5)(cn=abt6))
LDAP_GROUP_PROFILE_MEMBER_ATTR=uniqueMember

# CELERY

# expressed in KB
# CELERY__MAX_MEMORY_PER_CHILD="200000"
# ## 
# Note right autoscale value must coincide with worker concurrency value
# CELERY__AUTOSCALE_VALUES="15,10" 
# CELERY__WORKER_CONCURRENCY="10"
# ##
# CELERY__OPTS="--without-gossip --without-mingle -Ofair -B -E"
# CELERY__BEAT_SCHEDULE="/mnt/volumes/statics/celerybeat-schedule"
# CELERY__LOG_LEVEL="INFO"
# CELERY__LOG_FILE="/var/log/celery.log"
# CELERY__WORKER_NAME="worker1@%h"

# PostgreSQL
POSTGRESQL_MAX_CONNECTIONS=200

# Common containers restart policy
RESTART_POLICY_CONDITION="on-failure"
RESTART_POLICY_DELAY="5s"
RESTART_POLICY_MAX_ATTEMPTS="3"
RESTART_POLICY_WINDOW=120s

DEFAULT_MAX_UPLOAD_SIZE=5368709120
DEFAULT_MAX_PARALLEL_UPLOADS_PER_USER=5

```

### Esempio docker-compose.yml

```
version: '3.9'

# Common Django template for GeoNode and Celery services below
x-common-django:
  &default-common-django
  image: ${COMPOSE_PROJECT_NAME}_django:4.0
  restart: on-failure
  env_file:
    - .env
  volumes:
    # - './src:/usr/src/my_geonode'
    - statics:/mnt/volumes/statics
    - geoserver-data-dir:/geoserver_data/data
    - backup-restore:/backup_restore
    - data:/data
    - tmp:/tmp
  depends_on:
    db:
      condition: service_healthy
    geoserver:
      condition: service_healthy

services:

  # Our custom django application. It includes Geonode.
  django:
    << : *default-common-django
    build:
      context: ./
      dockerfile: Dockerfile
    container_name: django4${COMPOSE_PROJECT_NAME}
    healthcheck:
      test: "curl --fail --silent --write-out 'HTTP CODE : %{http_code}\n' --output /dev/null http://127.0.0.1:8001/"
      start_period: 60s
      interval: 60s
      timeout: 10s
      retries: 10
    environment:
      - IS_CELERY=False
    entrypoint: ["/usr/src/my_geonode/entrypoint.sh"]
    command: "uwsgi --ini /usr/src/my_geonode/uwsgi.ini"

  # Celery worker that executes celery tasks created by Django.
  celery:
    << : *default-common-django
    image: ${COMPOSE_PROJECT_NAME}_django:4.0
    container_name: celery4${COMPOSE_PROJECT_NAME}
    depends_on:
      - django
    environment:
      - IS_CELERY=True
    entrypoint: ["/usr/src/my_geonode/entrypoint.sh"]
    command: "celery-cmd"

  # Nginx is serving django static and media files and proxies to django and geonode
  geonode:
    image: geonode/nginx:4.0
    build: ./docker/nginx/
    container_name: nginx4${COMPOSE_PROJECT_NAME}
    environment:
      - HTTPS_HOST=${HTTPS_HOST}
      - HTTP_HOST=${HTTP_HOST}
      - HTTPS_PORT=${HTTPS_PORT}
      - HTTP_PORT=${HTTP_PORT}
      - LETSENCRYPT_MODE=${LETSENCRYPT_MODE}
      - RESOLVER=127.0.0.11
    ports:
      - "${HTTP_PORT}:80"
      - "${HTTPS_PORT}:443"
    volumes:
      - nginx-confd:/etc/nginx
      - nginx-certificates:/geonode-certificates
      - statics:/mnt/volumes/statics
    restart: on-failure

  # Gets and installs letsencrypt certificates
  letsencrypt:
    image: geonode/letsencrypt:4.0
    build: ./docker/letsencrypt/
    container_name: letsencrypt4${COMPOSE_PROJECT_NAME}
    environment:
      - HTTPS_HOST=${HTTPS_HOST}
      - HTTP_HOST=${HTTP_HOST}
      - ADMIN_EMAIL=${ADMIN_EMAIL}
      - LETSENCRYPT_MODE=${LETSENCRYPT_MODE}
    volumes:
      - nginx-certificates:/geonode-certificates
    restart: on-failure

  # Geoserver backend
  geoserver:
    image: geonode/geoserver:2.20.7
    build: ./docker/geoserver/
    container_name: geoserver4${COMPOSE_PROJECT_NAME}
    healthcheck:
      test: "curl --fail --silent --write-out 'HTTP CODE : %{http_code}\n' --output /dev/null http://127.0.0.1:8080/geoserver/ows"
      start_period: 60s
      interval: 60s
      timeout: 10s
      retries: 10
    env_file:
      - .env
    ports:
      - "8080:8080"
    volumes:
      - statics:/mnt/volumes/statics
      - geoserver-data-dir:/geoserver_data/data
      - backup-restore:/backup_restore
      - data:/data
      - tmp:/tmp
    restart: on-failure
    depends_on:
      db:
        condition: service_healthy
      data-dir-conf:
        condition: service_healthy

  data-dir-conf:
    image: geonode/geoserver_data:2.20.7
    container_name: gsconf4${COMPOSE_PROJECT_NAME}
    entrypoint: sleep infinity
    volumes:
      - geoserver-data-dir:/geoserver_data/data
    restart: on-failure
    healthcheck:
      test: "ls -A '/geoserver_data/data' | wc -l"

  # PostGIS database.
  db:
    # use geonode official postgis 13 image
    image: geonode/postgis:13
    command: postgres -c "max_connections=${POSTGRESQL_MAX_CONNECTIONS}"
    container_name: db4${COMPOSE_PROJECT_NAME}
    env_file:
      - .env
    volumes:
      - dbdata:/var/lib/postgresql/data
      - dbbackups:/pg_backups
    restart: on-failure
    healthcheck:
      test: "pg_isready -d postgres -U postgres"
    # uncomment to enable remote connections to postgres
    #ports:
    #  - "5432:5432"

  # Vanilla RabbitMQ service. This is needed by celery
  rabbitmq:
    image: rabbitmq:3.7-alpine
    container_name: rabbitmq4${COMPOSE_PROJECT_NAME}
    volumes:
      - rabbitmq:/var/lib/rabbitmq
    restart: on-failure

  jenkins:
    image: jenkins/jenkins:2.164-jdk11
    container_name: jenkins4${COMPOSE_PROJECT_NAME}
    user: jenkins
    ports:
      - '${JENKINS_HTTP_PORT}:${JENKINS_HTTP_PORT}'
      - '${JENKINS_HTTPS_PORT}:${JENKINS_HTTPS_PORT}'
      - '50000:50000'
    # network_mode: "host"
    volumes:
      - jenkins_data:/var/jenkins_home
      - backup-restore:/backup_restore
      - data:/data
    environment:
      - 'JENKINS_OPTS=--httpPort=${JENKINS_HTTP_PORT} --httpsPort=${JENKINS_HTTPS_PORT} --prefix=/jenkins'
    restart: on-failure

volumes:
  jenkins_data:
    driver: local
  statics:
    name: ${COMPOSE_PROJECT_NAME}-statics
  nginx-confd:
    name: ${COMPOSE_PROJECT_NAME}-nginxconfd
  nginx-certificates:
    name: ${COMPOSE_PROJECT_NAME}-nginxcerts
  geoserver-data-dir:
    name: ${COMPOSE_PROJECT_NAME}-gsdatadir
  dbdata:
    name: ${COMPOSE_PROJECT_NAME}-dbdata
  dbbackups:
    name: ${COMPOSE_PROJECT_NAME}-dbbackups
  backup-restore:
    name: ${COMPOSE_PROJECT_NAME}-backup-restore
  data:
    name: ${COMPOSE_PROJECT_NAME}-data
  tmp:
    name: ${COMPOSE_PROJECT_NAME}-tmp
  rabbitmq:
    name: ${COMPOSE_PROJECT_NAME}-rabbitmq

```
