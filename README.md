# ws-3d-lidar-profile
FOSSGIS 2023 Workshop "QGIS 3D, LiDAR Punktwolken und Profilwerkzeug"

Hier entsteht die Anleitung zum Workshop. Sie finden hier später auch die Download-Links für die Übungsdaten.

## 3D Datenstrukturen für Darstellung und Analyse in QGIS

* Raster-Bilder: die  Höheninformation ist im Graustufenkanal (16bit Bilder) codiert. z.B. 16bit GeoTIFF-Dateien. jede Rasterzelle hat einen Höhenwert zugewiesen. So können z.B. Geländemodelle mit regelmässigem Sampling gespeichert werden.
* Mesh-Daten: kontinuierliche Oberflächen bestehend aus unterschiedlich grossen Vielecken. 
* 3D-Vektor-Polygondaten: Jeder Stützpunkt kann eine 3D-Koordinate beinhalten. Damit können z.B. 3D Leitungen, Strassenzüge, Gewässernetze mit 3D-Verlauf oder auch Gebäude- oder Tunnel/Höhlenmodelle abgebildet werden.

## LiDAR Punktwolken (Point Clouds)

### Beispiel Solothurn

#### Download und Datenaufbereitung LiDAR Daten
Download der folgenden Dateien (Kacheln mit je 500m Kantenlänge, KBS: EPSG:2056), separat für DOM (Punktwolken) und DTM (Rasterdatei).

DOM (LAZ)

* [DOM LAZ-Datei 2607000/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dsm.laz)
* [DOM LAZ-Datei 2607000/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dsm.laz)
* [DOM LAZ-Datei 2607500/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dsm.laz)
* [DOM LAZ-Datei 2607500/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dsm.laz)

```
wget https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dsm.laz https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dsm.laz https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dsm.laz https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dsm.laz

```
Danach alle 4 LAZ-Dateien per pdal pipeline (Dateiname: pdal-pipeline_merge_and_convert-to-copc.json) in eine Datei mergen und nach copc konvertieren (braucht mindestens PDAL 2.4):
```
[
    "2607000_1228000.ch.so.agi.lidar_2019.dsm.laz",
    "2607000_1228500.ch.so.agi.lidar_2019.dsm.laz",
    "2607500_1228000.ch.so.agi.lidar_2019.dsm.laz",
    "2607500_1228500.ch.so.agi.lidar_2019.dsm.laz",
    {
        "type":"filters.merge"
    },
    {
        "type":"writers.copc",
        "filename":"solothurn.copc.laz"
    }
]
```

und Aufruf Kommando mit

```
pdal pipeline pdal-pipeline_merge_and_convert-to-copc.json
```

oder mit der docker-Variante (das docker pull pdal/pdal muss nur 1x gemacht werden; bei der docker-Variante muss bei den Dateipfaden bei der Verwendung wie unten jeweils ein /opt/data vorangestellt werden, bei sämtlichen input Files und output files - weil das aktuelle Directory im docker unter /opt/data gemountet ist):

```
docker pull pdal/pdal
docker run -it -v $(pwd):/opt/data/ pdal/pdal pdal pipeline /opt/data/pdal-pipeline_merge_and_convert-to-copc.json
```


DTM (GeoTIFF)

* [DTM GeoTIFF-Datei 2607000/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607000/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607500/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607500/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dtm.tif)

```
wget https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dtm.tif 
```

Danach werden alle 4 Datein in ein COG (cloud optimized GeoTIFF mit Pyramiden) zusammengefasst, z.B. über den folgenden Befehl:

```
gdalwarp -ot FLOAT32 -of COG -t_srs EPSG:2056 -co COMPRESS=DEFLATE -co OVERVIEW_RESAMPLING=CUBIC *.tif solothurn_dtm_2019_25cm.tif
```

