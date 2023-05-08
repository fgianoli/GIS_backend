# WebGIS Backend

In questo corso tratteremo dell'installazione e della configurazione di un server e dell'installazione dei maggiori software per il WebGIS

- Installare Ubuntu su VM Virtualbox
- Configurare Ubuntu, i [Comandi Shell](https://github.com/fgianoli/GIS_backend/blob/main/00_Basic_commands.md)
- Installare [Docker e Docker Compose](https://github.com/fgianoli/GIS_backend/blob/main/01_Docker.md)
- Installare [PostgreSQL e PostGIS](https://github.com/fgianoli/GIS_backend/blob/main/02_PostgreSQL_PostGIS.md)
- Installare [Geoserver](https://github.com/fgianoli/GIS_backend/blob/main/03_Geoserver.md)
- Installare Geonode
- Installare Lizmap
- Installare G3W con Docker
- Geoserver e REST API
- Vector Tiles

## Risorse
- [Ubuntu download](https://www.ubuntu-it.org/download)
- [VirtualBox](https://www.virtualbox.org)
- [Putty](https://www.putty.org)
- [WinSCP](https://winscp.net/eng/download.php)

## Inoltro porte dalla  VM

```
http TCP 127.0.0.1 8000 10.0.2.15 80
ssh  TCP 127.0.0.1 2222 10.0.2.15 22
postgres TCP 127.0.0.1 5434 10.0.2.15 5432
geoserver 127.0.0.1 5000 10.0.2.15 8080
```
