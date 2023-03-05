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

DTM (GeoTIFF)

* [DTM GeoTIFF-Datei 2607000/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607000/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607500/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607500/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dtm.tif)

```
wget https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dtm.tif 
```

