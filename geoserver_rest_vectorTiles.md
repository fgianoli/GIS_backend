#   Docker images

Geoserver v.2.23.0
postgis 15-3.3

docker-compose sulla base di

[SOURCE/DOC] https://github.com/kartoza/docker-geoserver

___

#   VIDEO 1 - Geoserver REST API 

[DOC] https://docs.geoserver.org/latest/en/user/rest/index.html#rest

##  curl e Postman REST API test

Test chiamate http ai REST API di Geoserver con comando curl o usando l'applicaizone Postman
[SOURCE/DOC] https://www.postman.com/downloads/

NOTA:
Come riportato nella docuemntazione di Geoserver relativa ai rest lo swagger e gli esempi di chiamate REST potrebbero non funzionare esattamente come riportato per mancaza di aggiornamenti continui su questa parte di doc.
Si dovrà proceder eun po' a tentativi nel ricostruire correttamente url path e payload delle chiamate da effettuare. Gli esempi riportati di seguito sono stati testati in locale con le verisoni di Geoserver indicate nel corso.

Esempi di chiamata alle API di Geoserver:

1 - Workspaces: lista esistenti e crearne uno nuovo

[DOC]  
* https://docs.geoserver.org/latest/en/api/#1.0.0/workspaces.yaml
* https://docs.geoserver.org/latest/en/user/rest/workspaces.html

```
curl -v -u admin:myawesomegeoserver -XGET http://localhost:8600/geoserver/rest/workspaces 

curl -v -u admin:myawesomegeoserver -XPOST http://localhost:8600/geoserver/rest/workspaces -d '{"workspace":{"name":"test"}}' -H "accept: application/json" -H "content-type: application/json"

curl -v -u admin:myawesomegeoserver -XGET http://localhost:8600/geoserver/rest/workspaces 
```

In una nuova installaizone di Geoserver il primo comando restituirà una lista vuota di workspaces

```
{"workspaces":""}
```

il secondo comando tramite chiamata POST creerà un nuovo workspace **test**

mentre lanciando l'ultimo comando otterremo la lista aggiornata con il nuovo workspace

```
{"workspaces":{"workspace":[{"name":"test","href":"http://localhost:8600/geoserver/rest/workspaces/test.json"}]}}
```

infine possiamo provare anche ad eliminare il workspace **test** con la chiamata DELETE

```
curl -v -u admin:myawesomegeoserver -XDELETE "http://localhost:8600/geoserver/rest/workspaces/test?recurse=true"
```

2 - DataStores: esistenti e nuovo datastore

[DOC] 
* https://docs.geoserver.org/latest/en/api/#1.0.0/datastores.yaml
* https://docs.geoserver.org/latest/en/user/rest/stores.html

Interroghiamo la lista di datastore disponibili nel workspace creato in precedenza:

```
curl -v -u admin:myawesomegeoserver -XGET http://localhost:8600/geoserver/rest/workspaces/test/datastores
```

che ci restituirà una json vuoto:

```
{"dataStores":""}
```

Creiamo ora quindi un nuovo datastore per collegarci al database postgres con estensioni postgis locale (vedi docker-compose.yaml)
In questo caso come vedremo i parametri e il body della chiamata POST da effettuare iniziano a diventare più corposi, usiamo **Postman** per poter lavorare più comodamente con input più complessi

Datastore POST request url con Basic Authentication
```
localhost:8600/geoserver/rest/workspaces/test/datastores/
```

Datastore POST request body come raw application/json
```
{
            "dataStore":
                {"name": "local_postgis",
                "connectionParameters": {
                    "entry": [
                      {"@key":"host","$":"db"},
                          {"@key":"port","$":5432},
                          {"@key":"database","$":"gis"},
                          {"@key":"user","$":"docker"},
                          {"@key":"passwd","$":"docker"},
                          {"@key":"dbtype","$":"postgis"},
                          {"@key":"schema","$":"public"}
                        ]
                    }
                }
            }
```

3 - Layers: Pubblicare un nuovo layer


[DOC] https://docs.geoserver.org/latest/en/api/#1.0.0/layers.yaml

Ottenere lista layer pubblicati in un Datastore

Url path API - GET request senza body
```
localhost:8600/geoserver/rest/workspaces/test/datastores/local_postgis/featuretypes.json

o versione html

localhost:8600/geoserver/rest/workspaces/test/datastores/local_postgis/featuretypes

o ancora 

localhost:8600/geoserver/rest/workspaces/test/layers
```

Pubblicare un nuovo layer da datastore PostGis

Url path API - POST request
```
localhost:8600/geoserver/rest/workspaces/test/datastores/local_postgis/featuretypes
```

body 
```
{
    "featureType": {"name": "popolazione_italia"}
}
```

[Opzionale]

4 - Upload SLD style

Per assegnare uno style ad un layer dobbiamo innanzi tutto creare e caricare lo style su geoserver.

Prima ancora possiamo fare un GET degli stili pubblicati per assicurarci di caricare poi uno stile con nome nuovo univoco.


a. Lista stili Geoserver

Url GET request
```
localhost:8600/geoserver/rest/styles
```

b. Definire un nuovo style

NOTA: il parametro "workspace" nel body della request è opzionale se non indicato caricherà lo stile senza associarlo ad alun workspace.

Url POST request
```
localhost:8600/geoserver/rest/styles
```

Body
```
{
    "style": { 
    "name": "popolazione_italia", 
    "filename": "popolazione_italia.sld",
    "workspace": "test"
    }
}
```

c. Caricare l'XML del file SLD nello stile creato

NOTA: in questo caso dovremo modificare l'header della chiamata e inoltre se l'SLD è fromattato con tag di stile

\<SvgParameter\>

vanno sostituiti con tag

\<CssParameter\>

    ContentType = application/vnd.ogc.sld+xml

Url PUT request
```
localhost:8600/geoserver/rest/styles/pop_ita.xml
```

Body (XML)
```
<... inserire il testo del file demo sld allegato>
```

5 - Associare lo stile al layer caricato in precedenza

Per associare lo stile ad un layer esistente come Default style o come style altenrativo dobbimao modificare e quindi fare una richiesta POST al layer stesso

a. ottenere info del layer esistente

URL GET request
```
localhost:8600/geoserver/rest/layers/test:popolazione_italia
```

che ci restituirà un json con le informaizoni incluso se presente il default style e altenrativi del layer eg.
```
{
    "layer": {
        "name": "popolazione_italia",
        "type": "VECTOR",
        "defaultStyle": {
            "name": "generic",
            "href": "http://localhost:8600/geoserver/rest/styles/generic.json"
        },
        "resource": {
            "@class": "featureType",
            "name": "test:popolazione_italia",
            "href": "http://localhost:8600/geoserver/rest/workspaces/test/datastores/local_postgis/featuretypes/popolazione_italia.json"
        },
        "attribution": {
            "logoWidth": 0,
            "logoHeight": 0
        },
        "dateCreated": "2023-05-04 07:48:04.746 UTC"
    }
}
```

b. modificare il default style e gli style alternativi per il layer

Ora prendiamo il json ottenuto in precedenza e modifichiamo come segue nel body
spostando il generic.sld negli style alternativi e impostando lo style creato e caricato come default

URL PUT request
```
localhost:8600/geoserver/rest/layers/test:popolazione_italia
```

Body
```
{
    "layer": {
        "name": "popolazione_italia",
        "type": "VECTOR",
        "defaultStyle": {
            "name": "popolazione_italia",
            "href": "http://localhost:8600/geoserver/rest/styles/popolazione_italia.json"
        },
        "styles": {
            "@class": "linked-hash-set",
            "style": [
                {
                    "name": "generic",
                    "href": "http://localhost:8600/geoserver/rest/styles/generic.json"
                }
            ]
        },
        "resource": {
            "@class": "featureType",
            "name": "test:popolazione_italia",
            "href": "http://localhost:8600/geoserver/rest/workspaces/test/datastores/local_postgis/featuretypes/popolazione_italia.json"
        },
        "attribution": {
            "logoWidth": 0,
            "logoHeight": 0
        },
        "dateCreated": "2023-05-04 07:48:04.746 UTC"
    }
}
```

6 - Eliminare un layer e in cascata gli stili usati dal solo layer

NOTA: il parametro recurse se true eliminerà il layer anche da tutti i layer group più i layer group in cui è presente il solo layer

URL DELETE request
```
localhost:8600/geoserver/rest/workspaces/test/layers/popolazione_italia?recurse=true
```


##  Python HTTP script REST API automatization

[Opzionale] magari qui invece facco solo vedere un esempio di applicaizone python per fare requests
e spiego possibili script implementabili ... invece di implementarlo che siamo già sui 20 minuti belli pieni.

Configurare geoserver tramite yaml file e chiamate http con python

1 - creare un virtual environment e attivarlo, quindi installare i relativi requirements

Linux / MacOS
```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Windows
```
python3 -m venv venv
venv/bin/activate.bat
pip install -r requirements.txt
```

2 - scriviamo un codice Python 3.x che faccia richieste http get e post con il modulo requests per automatizzare le chiamate alle API

[DOC] https://requests.readthedocs.io/en/latest/

```
# GET request eg.
response = requests.get(url, params=payload)

# dove payload è un diizonario di parametri eg. payload = {"param1": "text", "param2":1, ...}


# POST request eg.
post_response = requests.post(url_post, json=payload)

# dove payload è un json eg. payload = {"param1": "text", "param2":1, ...}

# Basic Authentication eg.
private_url_response = requests.get(
    url='http://localhost:8600/geoserver/rest/...',
    auth=HTTPBasicAuth('username', 'password')
)

# Print dello status code della risposta
private_url_response.status_code

```


___

#   VIDEO 2 - Vector Tiles

[DOC] https://www.maptiler.com/news/2019/02/what-are-vector-tiles-and-why-you-should-care/

Cosa sono e a cosa servono.

##  Postgis ST_AsMVT

Creare Vector Tiles direttamente da Postgis 
[DOC] https://postgis.net/docs/ST_AsMVT.html

Esempio di query per ottenere pbf tile con zoom, x, y da una tabella con geometria
```
WITH mvtgeom AS
    (
    SELECT ST_AsMVTGeom(ST_Transform(geom,3857), ST_TileEnvelope(1,1, 1), extent => 4096, buffer => 256) AS geom, 
    den_reg, popolazione, ROW_NUMBER() OVER () AS feature__id
    FROM popolazione_italia
    WHERE ST_Transform(geom,3857) && ST_TileEnvelope(1, 1, 1, margin => (256.0 / 4096))
    )
    SELECT ST_AsMVT(mvtgeom.*,'popolazione_italia',4096,'geom','id')
FROM mvtgeom;
```

Vediamolo in azione con un server python parametrizzando zoom, x, y e nome tabella per cui generare i vector tiles nel seguente paragrafo.

##  Minimal Python Vector Tiles Server

Servire Vector Tiles create da postgis con un minimal web server
[DOC] https://www.crunchydata.com/blog/dynamic-vector-tiles-from-postgis

In questo articolo e relativo repository troviamo un esempio pratico di implementazione di un server in grado di interrogare e distribuire vecotr tiles interrogando un db postgis.

Proviamo a clonare e configurare il repository con il nostro database postgis e il layer popolazione_italia utilizzato in precedenza nel corso dei REST di Geoserver.

[REPO] https://github.com/pramsey/minimal-mvt.git

```
git clone https://github.com/pramsey/minimal-mvt.git
cd minimal-mvt
python3 -m venv venv
pip install --upgrade pip
pip install -r requirements.txt
```

nota: se pip install dovesse fallire provare a sostituire la libreria psygopg2 con psycopg2-binary

Ora configuriamo la nostra connessione al database e la tabella da pubblicare nel minimal-mvt.py

```
# Database to connect to
DATABASE = {
    'user':     'docker',
    'password': 'docker',
    'host':     'localhost',
    'port':     '32767',
    'database': 'gis'
    }

# Table to query for MVT data, and columns to
# include in the tiles.
TABLE = {
    'table':       'popolazione_italia',
    'srid':        '4326',
    'geomColumn':  'geom',
    'attrColumns': 'id, den_reg, popolazione'
    }  
```

quindi lanciamo il server python con il comando

```
 python minimal-mvt.py 
```

Testiamo con una chiamata al server che dovrebbe quindi farci scaricare il file binario .mvt

```
http://localhost:8080/0/0/0.mvt
```

Per visualizzare il risultato vedremo poi come inserire queste Vector Tiles in una mappa eg. MapBox o Maplibre

[opzionale]
Velocizzare il serving delle Vecotr Tiles con un layer di caching eg. Redis
Usare flask o fastapi per documentare e standardizzare le api di pubblicaizone delel vector tiles.

##  Geoserver Vector Tiles

Pubblicare Vector Tiles con geoserver
[DOC] https://docs.geoserver.org/latest/en/user/extensions/vectortiles/tutorial.html

L'immagine di kartoza che usiamo per il corso include già l'estensione per la distribuzioe del servizio con tile caching in formato vector tiles, vediamo ora come abilitare tale serviizo per singolo layer o di default nel pubblicare nuovi layers.

Per singolo layer troviamo l'opzione nella lista di formati di pubblicazione nella tab Tile Caching quando andiamo ad editare il layer. Per abilitare quindi i Vector Tiles ci basterà spuntare nella lista delle **Tile Image Formats**

**application/vnd.mapbox-vector-tile**

In basso nella lista di gridset di pubblicazione vedremo di default i due più comunemente usati EPSG:4326 e EPSG:900913 per le Vector Tiles. In questa sezione inoltre è possibile definire oltre ai livelli di zoom pubblicati e cachabili anche altri gridset personalizzati.

In alternativa, per abilitare il formato come default di pubblicazione sia singolo vector layer che group layer, è possibile selezionare lo stesso formato anche nella pagina delle Caching Defaults del main menu. Stessa cosa nche per il default grid set disponibile per i servizi.


### Preview Vector Tiles di Geoserver

La preview dei formati Vector Tiles è visualizzabile con rendering mappa da Geoserver stesso navigando nella sezione 
**Tile Layers**

Nella colonna preview è possibile selezionare per i diversi grid set disponibili vari formati di tile layers inclusi i default

**EPSG:4326 / pbf**

**EPSG:900913 / pbf**

selezionado questi si aprirà un renderer mappa con la preview del layer formato pbf ovvero Vectro Tiles.


### Visualizzare Vector Tiles pubblicate tramite Geoserver e minimal web server con MapBox e Maplibre

Ora che abbiamo sia Geoserver che il minimal web server in grado di distribuire Vector Tiles proviamo quindi a visualizzarne le geometrie.

Nel repository del tutorial del minimal web server troveremo già una demo Mapbox in cui i basterà modificare nel file index.html il centro della mappa per poter visualizzare e testare il nostro layer popolazione_italia.

```
var map = new mapboxgl.Map({
    'container': 'map',
    'zoom': 5, // Italia estensione
    'center': [12, 42], // Italia centroide "circa a spanne"
    ...
```

lanciamo dalla cartella /minimal-mvt/map-mapboxgl un local server con python 

```
python3 -m http.server
```

quindi andiamo su browser all'indirizzo loclahost:8000 e dovremmo infine vedere le nostre belle tiles.

Ora proviamo a stilizzare le tiles lato client tramite le expressions di mapboxgl.
Utilizzeremo il campo popolazione per creare la stessa scala colori dello stile .sld di geoserver

Aggiungiamo quindi nella lista dei layer un altro layer di tipo fill con stops che possiamo copiare dall'.sld creato per geoserver

```
        {
            'id': 'postgis-tiles-layer-fill',
            'type': 'fill',
            'source': 'postgis-tiles',
            // ST_AsMVT() uses 'default' as layer name
            'source-layer': 'default', 
            'minzoom': 0,
            'maxzoom': 22,
            'paint': {
                'fill-color': {
                    property: 'popolazione',
                    stops: [[570365, '#abdda4'], [4448841, '#ffffbf'],[5898124,'#fdae61'],[10018806,'#d7191c']]
                },
                'fill-opacity': 0.85
            }
        }
```

modifichiamo ora la source delle vector tiles con quella di Geoserver e controlliamo in mappa di ottenere il medesimo risultato

```
            'postgis-tiles': {
                'type': 'vector',
                'tiles': [
//                    "http://localhost:8080/{z}/{x}/{y}.mvt"
                    "http://localhost:8600/geoserver/test/gwc/service/wmts?REQUEST=GetTile&SERVICE=WMTS&VERSION=1.0.0&LAYER=test:popolazione_italia&STYLE=&TILEMATRIX=EPSG:900913:{z}&TILEMATRIXSET=EPSG:900913&FORMAT=application/vnd.mapbox-vector-tile&TILECOL={x}&TILEROW={y}"
                ]

```

e modifichiamo il nome del source-layer

```
'source-layer': 'popolazione_italia',//'default', 
```

Mentre il minmal web server distribuisce un unico layer Vector Tile con nome default Goeserver distribuisce n layer e il source-layer deve coincidere con il nome del layer in Geoserver (case-sensitive).

Verifichiamo su web browser che il layer è ancora visibile infine.

[Opzionale]

Carichiamo il layer con servizio wms da Geoserver per confrontare i due servizi e rendering.

aggiungiamo quindi una nuova source nella configurazione della classe mappa:

```
...
'sources': {
    ...
    'geoserver-wms':{
                'type':'raster',
                tiles:[
                    "http://localhost:8600/geoserver/wms?service=WMS&request=GetMap&version=1.1.1&width=256&height=256&CRS=EPSG:3857&layers=popolazione_italia&bbox={bbox-epsg-3857}&opacities=255&transparent=true&format=image/png&tiled=true"
                ]
            }
    ...
```

e aggiungiamo il layer popolazione_italia da questa source:

```
...
        'layers': [{
        ...
        {
            'id': 'geoserver-wms',
            'type': 'raster',
            'source': 'geoserver-wms',
            // ST_AsMVT() uses 'default' as layer name
            'source-layer': 'popolazione_italia',//'default', 
        }
    ...
```

Ricaricando la mappa nel browser vedremo quindi come ultimo layer sovrapposto il layer servito da geoserver con serviizo wms.
Salta già subito all'occhio la qualità inferiore del rendering del layer.

### Consideraizoni finali Vector Tiles

Pros

+ stilizzazione lato client più flessibile
+ rendering grafico migliore, più fluido nel passaggio a diversi livelli di zoom e con minore rendering delay
+ le tiles hanno una dimensione ridotta rispetto ai servizi raster il che le rende ideali per lo streaming e le mappe offline  

Cons

- Il rendering avviene lato client e potrebbe avere problemi di performance su dispositivi datati o comunque per mappe con molti layer e stilizzazioni con formattazioni condizionali (expression) complesse. 
