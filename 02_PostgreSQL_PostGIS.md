# INSTALLARE POSTGRESQL E POSTGIS

https://techviewleo.com/how-to-install-postgresql-database-on-ubuntu/

``` sh
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt -y update
sudo apt -y install postgresql-14

sudo -u postgres psql -c "SELECT version();"


```

## Installare PostGIS
https://www.vultr.com/docs/how-to-install-the-postgis-extension-for-postgresql/  


```sh
sudo apt install postgis postgresql-14-postgis-3

```

Ricordarsi di installare l'estensione PostGIS sul DB che si vuole utilizzare
https://www.postgresqltutorial.com/postgresql-administration/psql-commands/

Connettersi al database  
```sh
psql -d database -U  user -W
```
```sql
CREATE EXTENSION postgis;
```
