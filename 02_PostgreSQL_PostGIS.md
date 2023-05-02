# INSTALLARE POSTGRESQL E POSTGIS

[https://techviewleo.com/how-to-install-postgresql-database-on-ubuntu/
](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-22-04-quickstart)

``` sh
sudo apt update
sudo apt install postgresql postgresql-contrib

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
