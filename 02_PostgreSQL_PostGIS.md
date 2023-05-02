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


**Creare Username e Database**

```sh
su - postgres
createuser --interactive --pwprompt
```

Replace user with the name of the user that you want to own the database, and replace dbname with the name of the database that you want to create:
```
createdb -O user dbname 
```

### TIPS

Per impostare le configurazioni di postgres perchè sia accessibile da altri IP bisogna editare i settaggi nel file `pg_hba.conf` e nel file `postgresql.conf` presenti nella cartella `etc/postgresql/14/main`

```
cd etc/postgresql/14/main
nano pg_hba.conf 
```
aggiungere:  

`host 	all	 all	 0.0.0.0/0	  md5`
              
nel file `postgresql.conf` andare ad editare nel seguente modo:

```
nano postgresql.conf  

listen_addresses = '*'  
```

Riavviare il servizio  Postgres con `service postgresql restart`


To tune up postgresql see [here](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)


