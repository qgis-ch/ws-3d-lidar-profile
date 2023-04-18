# ws-3d-lidar-profile
Workshop "QGIS 3D, LiDAR Punktwolken und Profilwerkzeug"

[Download Sample Data Solothurn](https://www.carto.net/neumann/qgis-3d-lidar-workshop/solothurn_data.zip) - contains terrain model, orthofoto, 3D buildings, landcover and LiDAR-data, as well as a QGIS project (.qgs).

## 3D data structures for the display and analysis in QGIS

* Raster Images: the height information (terrain model) is encoded in gray scale (16bit images). Each raster cell has a height value assinged. Can be used to store terrain models with regular sampling.
* Mesh-data: continuous surfaces consisting on irregular triangles, quadrilaterals (quads), or other simple convex polygons (n-gons).
* 3D vector polygon data: Each vertex can have a z-value assigned in the coordinates. This can be used to store 3d information on utility lines, road or river network data with 3D information, cave or tunnel models
* LiDAR point cloud data: captured by a laser scanner device (air based or terrestric) that sends out laser impulses, that are reflected from the surfaces suveyed one or multiple times. The distance to the scanner can be calculated via the transit time of the laser impulses from transmission to reception, and classifications of ground properties (buildings, trees, water, etc.) can be derived from the reflected pulses and their characteristics.

## Sample Data Solothurn

#### Download and data preparation
The required exercise data is provided in the ZIP folder above, but for the sake of completeness, here is a description of where the collected data came from and how it was compiled:

##### DOM (LAZ)

Download of the following files (tiles with 500m width and height, CRS: EPSG:2056), separate for DOM (point clouds) and DTM (TIFF raster file).

* [DOM LAZ-Datei 2607000/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dsm.laz)
* [DOM LAZ-Datei 2607000/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dsm.laz)
* [DOM LAZ-Datei 2607500/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dsm.laz)
* [DOM LAZ-Datei 2607500/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dsm.laz)

```
wget https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dsm.laz https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dsm.laz https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dsm.laz https://files.geo.so.ch/ch.so.agi.lidar_2019.dsm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dsm.laz

```
After that, all 4 LAZ files should be merged into a single COPC LAZ file (filename pdal-pipeline_merge_and_convert-to-copc.json - requires PDAL 2.4 or higher)
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

and the command can be started as follows
```
pdal pipeline pdal-pipeline_merge_and_convert-to-copc.json
```

or with the docker variant (the docker pull pdal/pdal has to be done only once; with the docker variant you have to prepend /opt/data to the file paths when using them as below, for all input files and output files - because the current directory is mounted in docker under /opt/data):

```
docker pull pdal/pdal
docker run -it -v $(pwd):/opt/data/ pdal/pdal pdal pipeline /opt/data/pdal-pipeline_merge_and_convert-to-copc.json
```


##### DTM (GeoTIFF)

* [DTM GeoTIFF-Datei 2607000/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607000/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607500/1228000](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dtm.tif)
* [DTM GeoTIFF-Datei 2607500/1228500](https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dtm.tif)

```
wget https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228000.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607000_1228500.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228000.ch.so.agi.lidar_2019.dtm.tif https://files.geo.so.ch/ch.so.agi.lidar_2019.dtm/aktuell/2607500_1228500.ch.so.agi.lidar_2019.dtm.tif 
```

After download, all 4 files are merged into a COG GeoTIFF (cloud optimized GeoTIFF with pyramids), using a command like the following

```
gdalwarp -ot FLOAT32 -of COG -t_srs EPSG:2056 -co COMPRESS=DEFLATE -co OVERVIEW_RESAMPLING=CUBIC *.tif solothurn_dtm_2019_25cm.tif
```

##### swissBuildings3D
The 3D building data can be downloaded from the swisstopo [swissBuildings3D-Website](https://www.swisstopo.admin.ch/de/geodata/landscape/buildings3d3.html#download)

We need the following tile:

```
wget https://data.geo.admin.ch/ch.swisstopo.swissbuildings3d_3_0/swissbuildings3d_3_0_2018_1127-12/swissbuildings3d_3_0_2018_1127-12_2056_5728.gdb.zip
```

The data is offered in ESRI FGDB format. It can be directly read by QGIS. The desired extent can be selected and the selected objects can then be saved in a GPKG file (Save selected objects as). Unfortunately, there is currently no reliable 3D clip algorithm in QGIS that can clip 3D buildings at tile boundaries.

##### swissTLM3D
The swissTLM3D contains land cover, individual objects, line geometries (e.g. traffic and other infrastructure) and points with Z coordinates. The data can be downloaded from swisstopo [swissTLM3D website](https://www.swisstopo.admin.ch/de/geodata/landscape/tlm3d.html#download). Only the whole of Switzerland can be downloaded in a multi-gigabyte total dataset. Different data formats are offered. We recommend the variant via Geopackage:

```
wget https://data.geo.admin.ch/ch.swisstopo.swisstlm3d/swisstlm3d_2023-03/swisstlm3d_2023-03_2056_5728.gpkg.zip
```

The dataset contains the whole of Switzerland. We can reduce the objects via selection to the desired extent: select graphically in QGIS and "Save as".


## Opening and configuring a new 3D scene in QGIS
A new 3D view can be added as a separate window or docked in the main QGIS windows using the menu "View" → "3D Map View" → "New 3D Map View".

![image](https://user-images.githubusercontent.com/884476/224544908-b1e6b7f8-8475-492e-83c9-bb7047ba026a.png)

The following tools are available for interaction with the 3D scene:

![image](https://user-images.githubusercontent.com/884476/224545483-3af8bee9-4e20-4db0-8676-6b8e1c3acb87.png)


## Navigation in 3D-Szene

### Mit Maus
* Linke Maustaste: Position im 3D-Raum verändern
* Ctrl-Linke Maustaste: Schwenken / Umherschauen / Ausschnitt Blickwinkel verändern
* Mittlere Maustaste (oder Shift-Linke Maustaste): Gesamte Szene rotieren. Mit 1. Klick wird der Rotationspunkt gesetzt, danach mit Maus-Drag rotieren.
* Rechte Maustaste: Zoomen

### Mit Tastatur
* Cursortasten oben/unten: vor- und zurückbewegen
* Cursortasten links/rechts: nach links/rechts bewegen
* Shift Cursor links/rechts: Szene rotieren
* Shift Cursor oben/unten: Szene kippen
* Ctrl Cursor links/rechts: Schwenken / Umherschauen / Ausschnitt Blickwinkel verändern
* Ctrl Cursor oben/unten: Blickrichtung nach oben / unten ändern
* Page up: Lift nach oben nehmen (Änderung Seehöhe Betrachter)
* Page down: Lift nach unten nehmen (Änderung Seehöhe Betrachter)

### Mit Navigations-Widget
* Rotation über Kompassrose
* Buttons hoch/runter für vorwärts/rückwärtsbewegen
* Buttons links und rechts für nach links/rechts bewegen
* Plus/Minus Buttons zum Ein/Auszoomen
* Buttons zum nach Kippen der Szene

![image](https://user-images.githubusercontent.com/884476/224546406-c54decda-6b73-4350-80df-4a590217fc79.png)

## Styling von 3D Daten

Die Einstellungen zur 3D-Grafik eines Layers erfolgt über die "Layereigenschaften" → "3D Ansicht":

![image](https://user-images.githubusercontent.com/884476/225347198-190a4518-c550-4332-803a-47e1c6977af5.png)

Falls ein Layer eine 3D-Darstellung zugewiesen hat, dann wird die 2D Repräsentation im 3D-View ignoriert, falls nicht, dann wird die 2D Repräsentation über das Geländemodell drauf drapiert.

Im Gegensatz zur 2D-Darstellung, gibt es bei der 3D Darstellung noch weniger Optionen und auch keine zusammengesetzten Symbole. Man kann aber mit der regelbasierten 3D-Darstellung ein Objekt auch mehrfach zeichnen, z.B. um einen Baum aus einen Zylinder (Stamm) und einer Kugel (Krone) zusammenzusetzen:

![image](https://user-images.githubusercontent.com/884476/225525097-c149637b-b03f-47c2-bbe4-2ad92a35d330.png)

Einstellungen zum Baumstamm (Zylinder):
![image](https://user-images.githubusercontent.com/884476/225525630-efc29f8f-53a0-48ee-887b-ae03a6068b81.png)

Einstellungen zur Baumkrone (Kugel):
![image](https://user-images.githubusercontent.com/884476/225526455-10bd4afe-d962-409c-bc26-a29a8abc6dad.png)

## Styling von LiDAR Point Cloud Daten

Punktwolkendaten können verschieden dargestellt werden:
* Einfarbig
* RGB Werte (falls vorhanden, im Solothurn Beispiel nicht der Fall)
* Klassierung nach Attributen (in Solothurn vorhanden)
* Attribute nach Verlauf, z.B. nach Intensität der Rückstrahlung (Dächer und Plätze haben mehr Intensität als z.B. Bäume bei der Rückstrahlung)

![image](https://user-images.githubusercontent.com/884476/225384014-2134a698-8364-40c4-8660-7d7a48378686.png)

Es werden in der Regel nicht alle Punkte dargestellt, sondern ausgedünnte Punkte. Das Punktbudget (z.B. 5 Millionen) legt fest wieviele Punkte gleichzeitig maximal dargestellt werden sollen.

## Geländehöhenprofil erstellen
Das Profilwerkzeug kann über das Menü "Ansicht" → "Geländehöhenprofil" gestartet werden. Ein eigenes Dockpanel geht auf:
![image](https://user-images.githubusercontent.com/884476/225532488-30c55b85-4d3a-4aac-922e-bbdf28c6a1b4.png)

Die Profillinie kann direkt in der Karte gezeichnet werden oder aus einer bestehenden Linie (z.B. Strassenachse) übernommen werden.

Ein Doppelklick auf einer Ebene öffnet die Darstellungseigenschaften bezüglich des Profils:
![image](https://user-images.githubusercontent.com/884476/225533778-bc823eb2-1c1f-467a-8cd6-c6417d56823b.png)

Im Beispiel sieht man die Profildarstellung des Layers "Eisenbahn" - ein Linienlayer - der aber beim Schneiden mit einer Profillinie als "Punkt" dargestellt wird. Man muss die Höhenbindung, Repräsentationsart und Punktdarstellung konfigurieren.

Bei Punktsymbolen empfiehlt es sich zudem, den vertikalen Ankerpunkt des Symbols auf "unten" zu setzen, damit das Symbol wirklich auf der Profillinie "aufsetzt":

![image](https://user-images.githubusercontent.com/884476/225534450-baa588ef-8971-40ae-8a8f-a8be43b6ad83.png)

![image](https://user-images.githubusercontent.com/884476/225534941-82faac1b-c07f-4858-a6dd-9113752a3caf.png)

Leider werden bei den Symbolen im Geländehöhenprofil die Längeneinheiten (Karteneinheiten) nicht respektiert, wodurch die Symbole beim Zoomen immer gleich gross dargestellt werden. Hoffentlich wird dies in zukünftigen Versionen verbessert!

## Geländehöhenprofil im Layout verwenden
Im Kartenlayout kann ein Geländehöhenprofile eingesetzt werden:
![image](https://user-images.githubusercontent.com/884476/225548078-7c8ab875-2a35-4b62-8969-22e0811731b6.png)

Im kontextsensitiven Einstellungs-Bereich können die Eigenschaften des Profils, wie Intervalle bei den Achsen, Beschriftungen, Gitterlinien, etc. eingestellt werden.

## Ausblick

### Konkret in der Pipeline (aus dem [letzten 3D crowdfunding-Projekt](https://www.lutraconsulting.co.uk/crowdfunding/pointcloud-processing-qgis/))
* [Verbesserung des Messwerkzeuges](https://github.com/qgis/QGIS/pull/52208) (Anzeige der Messlinie)

Punktwolken-Processing Provider (kommt mit QGIS 3.32 und benötigt PDAL 2.5, siehe [Github PR](https://github.com/qgis/QGIS/pull/52182))
* Metadaten aus Punktwolkendatensätzen extrahieren: Anzahl Punkte, Extents, KBS, etc.
* Punktwolkenformate konvertieren
* Punktwolkendatensätze umprojizieren
* KBS für Punktwolkendatensatz setzen
* Clip: Punktwolken durch Polygone beschneiden
* Filter: Subsets extrahieren gemäss PDAL Expressions (z.B. Kategorien, Z-Wert-Bereiche, Intensität, etc.)
* Merge: mehrere Punktwolkendatensätze in eine Datei zusammenfassen
* Tiling: Punktwolkendaten in Kacheln zerlegen
* Ausdünnen: ausgedünnte Punktwolkendatensätze erzeugen
* Ausdehnung: Polygone mit "Fussabdruck" (Ausdehnung) des Punktwolkendatensatzes erzeugen
* Density: Rasterdateien erzeugen mit "Anzahl Punkte" in jedem Rasterzellwert
* Export zu Raster: 2D Raster-Oberfläche erzeugen
* Export zu Raster via TIN: 2D Raster-Oberfläche erzeugen via TIN Interpolation
* Export zu Vektor: Export der 3D-Punkte in eine PointZ Vektordatei

Alle oben erwähnten Processing-Provider können über die Werkzeug-Box aufgerufen werden (einzelner Analyseschritt), im Batch-Modus verwendet werden oder Teil eines grafischen Processing-Modells sein.

### Persönliche Verbesserungswünsche
* Schönere Darstellung von Liniengeometrien (heute sind sie entweder pixelig dargestellt (aus dem darauf drapierten 2D-Kartenbild) oder sie verschwinden tw unter dem Gelände). Workaround: 3D-Linien mit Offset knapp über der Oberfläche darstellen (z.B. 0.5m über der Oberfläche), damit sie nicht partiell unter der 3D-Oberfläche verschwinden).
* 3D Attributdatenabfrage: getroffenes Objekt sollte besser hervorgehben werden, z.B: mit Wireframe, andere Farbe oder Bounding box
* Besseres Messwerkzeug mit Snapping und visuellem Feedback
