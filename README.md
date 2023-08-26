# Spatial Join Condizionato

2023-08-25 Sfida di analisi spaziale

Collega ciascuna scuola all'università più vicina nella stessa regione amministrativa.

![](https://pbs.twimg.com/media/F4Xs3mOb0AAeO6F?format=jpg&name=large)

<!-- TOC -->

- [Spatial Join Condizionato](#spatial-join-condizionato)
  - [PASSO 1:](#passo-1)
  - [PASSO 2:](#passo-2)
  - [DATI e PROGETTO](#dati-e-progetto)
  - [ALTRE ESPRESSIONI](#altre-espressioni)
  - [RELAZIONE N:N](#relazione-nn)
    - [TABELLA INTERMEDIA](#tabella-intermedia)
- [RIFERIMENTI](#riferimenti)
  - [video Youtube](#video-youtube)
- [DISCLAIMER](#disclaimer)

<!-- /TOC -->

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