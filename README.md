# Spatial Join Condizionato

2023-08-25 Sfida di analisi spaziale

Collega ciascuna scuola all'università più vicina nella stessa regione amministrativa.

Thread: <https://treeverse.app/view/xlb1zSBR>

![](https://pbs.twimg.com/media/F4Xs3mOb0AAeO6F?format=jpg&name=large)

<!-- TOC -->

- [Spatial Join Condizionato](#spatial-join-condizionato)
- [Totò FIANDACA](#totò-fiandaca)
  - [PASSO 1:](#passo-1)
  - [PASSO 2:](#passo-2)
  - [DATI e PROGETTO](#dati-e-progetto)
  - [ALTRE ESPRESSIONI](#altre-espressioni)
  - [RELAZIONE N:N](#relazione-nn)
    - [TABELLA INTERMEDIA](#tabella-intermedia)
- [Reymar SANCHEZ](#reymar-sanchez)
  - [QUERY](#query)
- [Bert Temme](#bert-temme)
- [Postholer - GIS Resources](#postholer---gis-resources)
- [MAHESH KUMAR](#mahesh-kumar)
- [RIFERIMENTI](#riferimenti)
  - [video Youtube](#video-youtube)
- [DISCLAIMER](#disclaimer)

<!-- /TOC -->

# Totò FIANDACA

<https://twitter.com/totofiandaca/status/1695093176304812192>

Strumento:

1. QGIS desktop
2. Espressioni di QGIS

## PASSO 1:

Trasferire `fid` del layer `admin_boundaries` ai due layer puntuali `schools` e `colleges`:

aggiungere un attributo `IDp` numerico e popolarlo con:

```
overlay_within(
    layer:='admin_boundaries',
    expression:="fid"
)[0]
```
![](imgs/img_00.png)

![](imgs/img_02.png)

## PASSO 2:

per collegare (tramite un segmento) ciascuna scuola (schools) all'università (colleges) più vicina nelle stessa regione amministrativa, usare l'espressione:

```
make_line(
    eval('overlay_nearest(\'colleges\',
          $geometry,filter:=IDp='||"IDp"||')')[0],
          $geometry)
```

nel generatore di geometrie:

![](imgs/img_01.png)

## DATI e PROGETTO

[DATI](./dati/)

## ALTRE ESPRESSIONI

```
make_line(@geometry,
          geometry(
            array_filter( 
            overlay_nearest('colleges',@feature,limit:=-1), 
            attribute( @element, 'IDp' ) = "IDp" )[0]))
```
## RELAZIONE N:N

Creando due relazione di progetto su `colleges` e `schools` (perché la relazione tra loro è N:N)

![](imgs/img_03.png)

![](imgs/plugin.png)
<https://github.com/pyarchinit/selectbyrelationship_repo>

### TABELLA INTERMEDIA

Nella tabella degli attributi delle `schools` aggiungo un attributo `c_osm_id` e lo popolo con l'espressione:

```
eval('overlay_nearest(\'colleges\',
          osm_id,filter:=IDp='||"IDp"||')')[0]
```

![](imgs/demo.gif)

vedi `progetto_rel`

# Reymar SANCHEZ

Strumento:

1. PostgreSQL/PostGIS
2. SQL

<https://twitter.com/SanchezReymar/status/1695078310198280503/>

Dati caricati in PostgreSQL/PostGIS

## QUERY

```SQL
-- AGGIUNGO shapename DI admin_boundaries A colleges

ALTER TABLE colleges ADD COLUMN shapename varchar (100);

UPDATE colleges a SET shapename = b."shapeName"
FROM admin_boundaries b
WHERE ST_Contains (b.geom, a.geom);

-- AGGIUNGO shapename DI admin_boundaries A schools

ALTER TABLE schools ADD COLUMN shapename varchar (100);

UPDATE schools a SET shapename = b."shapeName" 
FROM admin_boundaries b
WHERE ST_Contains (b.geom, a.geom);

-- NEAREST

SELECT osm_id,schools,colleges,ST_Makeline(ageom,bgeom) geom
    FROM (
    SELECT a.osm_id,a.name schools, b.name colleges, 
        row_number() over (partition by a.osm_id ORDER BY ST_Distance (a.geom,b.geom)) r, 
        a.geom ageom, b.geom bgeom
    FROM schools a JOIN colleges b ON ST_DWithin (a.geom,b.geom,1)
    WHERE a."shapename" = b."shapename") t
WHERE r = 1;
```

![](imgs/pg.png)

# Bert Temme

<https://twitter.com/berttemme/status/1695090882247082239>

Strumento:

1. duckDB
2. SQL

repo: <https://github.com/bertt/spatial_analysis_challenge/blob/main/README.md>

PS: non ho verificato!!!

![](https://user-images.githubusercontent.com/538812/263313387-2685612d-c48c-43f9-83dd-4dd386d7478c.png)

# Postholer - GIS Resources

<https://twitter.com/postholer/status/1696047017490030692>

Strumento:

1. bash;
2. GDAL/OGR
3. SQL

```sh
ogr2ogr -nln paths -dialect sqlite -sql
"SELECT
  r.*
FROM
  (
    SELECT
      b.shapeID AS admin,
      c.osm_id AS college,
      s.osm_id AS school,
      makeline(c.geom, s.geom) AS linegeom,
      st_length (makeline(c.geom, s.geom)) AS linelen
    FROM
      admin boundaries b
      JOIN (
        SELECT
          (ROW_NUMBER() OVER()) AS fid,
          *
        FROM
          colleges
      ) c ON st_intersects(c.geom, b.geom)
      JOIN (
        SELECT
          (ROW_NUMBER() OVER()) AS fid,
          *
        FROM
          schools
      ) s ON st_intersects(s.geom, b.geom)
    GROUP BY
      b.shapeName,
      c.fid,
      s.fid
    ORDER BY
      linelen asc
  ) r
GROUP BY
  r.school"
newdata.gpkg layers.gpkg
```

![](https://pbs.twimg.com/media/F4mRlhyaAAAwjDx?format=png&name=large)

![](https://pbs.twimg.com/media/F4mRm7SacAANkD1?format=jpg&name=large)

# MAHESH KUMAR

<https://twitter.com/MAHESHK92842090/status/1696112436678734058>

Strumento:

1. QGIS desktop
2. plugin [RefFunction](https://plugins.qgis.org/plugins/refFunctions/) dreprecato
3. espressioni di QGIS

```
make_line ($geometry, geometry(get:feature('colleges', 'fid', array_filter(aggregate(layer:='colleges',aggregate:='array_agg',expression:=geomnearest('colleges,'fid'), filter:="admin" = attribute(@parent,'admin'),order_by:0distance($geometry,geometry(@parent)))))))
```
PS: non ho verificato!!!

![](https://pbs.twimg.com/media/F4nNNw9bEAAC4en?format=jpg&name=small)

![](https://pbs.twimg.com/media/F4nNPQ7aAAApwyh?format=jpg&name=900x900)

# RIFERIMENTI

- <https://twitter.com/spatialthinkies/status/1695021747177951435/>
- QGIS: https://www.qgis.org/it/site/
- ISSUE: <https://github.com/qgis/QGIS/issues/43146>
- SE: <https://gis.stackexchange.com/questions/391120/qgis-expression-with-overlay-fuction-filter-condition-based-on-comparison-of-at>
- Blog post Pigreconfinito: <https://pigrecoinfinito.com/2021/11/01/qgis-e-lo-spatial-join-condizionato/>

## video Youtube

[![](https://img.youtube.com/vi/G4e4bUxuMS0/0.jpg)](https://https://youtu.be/G4e4bUxuMS0 "Video")


# DISCLAIMER

Il presente contenuto è stato realizzato da _**Salvatore Fiandaca**_ nel mese di agosto 2023 utilizzando [QGIS 3.28 Firenze LTR](https://qgis.org/it/site/) e distribuito con licenza [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.it)