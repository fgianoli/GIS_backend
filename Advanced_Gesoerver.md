
#   Docker images

Geoserver v.2.23.0
postgis 15-3.3

docker-compose sulla base di

[SOURCE/DOC] https://github.com/kartoza/docker-geoserver

___

#  Geoserver REST API 

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
