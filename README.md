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

## 3D Szene öffnen und einrichten
Neue 3D Ansicht als separates Fenster oder dock hinzufügen über Menü "Ansicht" → "3D Kartenansichten" → "Neue 3D-Kartenansicht".

![image](https://user-images.githubusercontent.com/884476/224544908-b1e6b7f8-8475-492e-83c9-bb7047ba026a.png)


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

## Ausblick

### Konkret in der Pipeline
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
* Schönere Darstellung von Liniengeometrien (heute sind sie entweder pixelig dargestellt oder sie verschwinden tw unter dem Gelände). Workaround: Linien mit Offset knapp über der Oberfläche darstellen (z.B. 
* 3D Attributdatenabfrage: getroffenes Objekt sollte besser hervorgehben werden, z.B: mit Wireframe, andere Farbe oder Bounding box
* Besseres Messwerkzeug mit Snapping und visuellem Feedback
