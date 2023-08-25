# spatial join condizionato

2023-08-25 Sfida di analisi spaziale

Collega ciascuna scuola all'università più vicina nella stessa regione amministrativa.

![](https://pbs.twimg.com/media/F4Xs3mOb0AAeO6F?format=jpg&name=large)

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

# RIFERIMENTI

- <https://twitter.com/spatialthinkies/status/1695021747177951435/>
- QGIS: https://www.qgis.org/it/site/


# DISCLAIMER

Il presente contenuto è stato realizzato da _**Salvatore Fiandaca**_ nel mese di agosto 2023 utilizzando [QGIS 3.28 Firenze LTR](https://qgis.org/it/site/) e distribuito con licenza [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.it)