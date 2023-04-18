# ws-3d-lidar-profile
FOSSGIS 2023 Workshop "QGIS 3D, LiDAR Punktwolken und Profilwerkzeug"

[Download Übungsdaten Solothurn](https://www.carto.net/neumann/qgis-3d-lidar-workshop/solothurn_data.zip) - enthält Höhenmodell, Orthofoto, 3D Gebäude, Bodenbedeckung und LiDAR-Daten, sowie ein QGIS Projekt (.qgs).

## 3D Datenstrukturen für Darstellung und Analyse in QGIS

* Raster-Bilder: die  Höheninformation ist im Graustufenkanal (16bit Bilder) codiert. z.B. 16bit GeoTIFF-Dateien. jede Rasterzelle hat einen Höhenwert zugewiesen. So können z.B. Geländemodelle mit regelmässigem Sampling gespeichert werden.
* Mesh-Daten: kontinuierliche Oberflächen bestehend aus unterschiedlich grossen Vielecken. 
* 3D-Vektor-Polygondaten: Jeder Stützpunkt kann eine 3D-Koordinate beinhalten. Damit können z.B. 3D Leitungen, Strassenzüge, Gewässernetze mit 3D-Verlauf oder auch Gebäude- oder Tunnel/Höhlenmodelle abgebildet werden.
* LiDAR Punktwolkendaten: von einem Laserscanner (flugzeuggestützt oder terrestrisch) werden Laserimpulse ausgesendet, von der Oberfläche reflektiert und ein oder mehrmals wieder empfangen. Über die Laufzeit vom Senden bis zum Empfang kann die Entfernung zum Scanner ausgerechnet werden, aus den reflektierten Impulsen und deren Charakteristik können Klassierungen der Bodeneigenschaften (Gebäude, Bäume, Wasser, etc.) abgeleitet werden.

## Beispiel Solothurn

#### Download und Datenaufbereitung
Die benötigten Übungsdaten befinden sich im obigen ZIP-Ordner zusammengefasst, aber der Vollständigkeit halber soll hier noch beschrieben werden, woher die gesammelten Daten stammen und wie sie zusammengefasst wurden:

##### DOM (LAZ)

Download der folgenden Dateien (Kacheln mit je 500m Kantenlänge, KBS: EPSG:2056), separat für DOM (Punktwolken) und DTM (Rasterdatei).

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


##### DTM (GeoTIFF)

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

##### swissBuildings3D
Die 3D-Gebäuddaten können von der swisstopo [wwissBuildings3D-Website](https://www.swisstopo.admin.ch/de/geodata/landscape/buildings3d3.html#download) heruntergeladen werden.

Es wird die folgende Kachel benötigt:

```
wget https://data.geo.admin.ch/ch.swisstopo.swissbuildings3d_3_0/swissbuildings3d_3_0_2018_1127-12/swissbuildings3d_3_0_2018_1127-12_2056_5728.gdb.zip
```

Die Daten werden im ESRI FGDB-Format angeboten. Diese können von QGIS gelesen werden. Der gewünschte Ausschnitt kann selektiert und die selektierten Objekte danach in einer GPKG-Datei gespeichert werden (Gewählte Objekte Speichern als). Leider existiert im QGIS derzeit kein zuverlässiger 3D Clip-Algorithmus der 3D Gebäude an Kachelgrenzen schneiden kann.

##### swissTLM3D
Der swissTLM3D enthält Bodenbedeckungen, Einzelobjekte, Liniengeometrien (z.B. Verkehr und andere Infrastruktur) und Punkte mit Z-Koordinaten. Die Daten können von der swisstopo [swissTLM3D Website](https://www.swisstopo.admin.ch/de/geodata/landscape/tlm3d.html#download) heruntergeladen werden. Es kann nur die gesamte Schweiz in einem Multi-Gigabyte Gesamtdatensatz heruntergeladen werden. Es werden verschiedene Datenformate angeboten. Wir empfehlen die Variante via Geopackage:

```
wget https://data.geo.admin.ch/ch.swisstopo.swisstlm3d/swisstlm3d_2023-03/swisstlm3d_2023-03_2056_5728.gpkg.zip
```

Im Datensatz ist die gesamte Schweiz enthalten. Wir können die Objekte via Auswahl auf den gewünschten Ausschnitt reduzieren: im QGIS grafisch selektieren und "Speichern unter".


## 3D Szene öffnen und einrichten
Neue 3D Ansicht als separates Fenster oder dock hinzufügen über Menü "Ansicht" → "3D Kartenansichten" → "Neue 3D-Kartenansicht".

![image](https://user-images.githubusercontent.com/884476/224544908-b1e6b7f8-8475-492e-83c9-bb7047ba026a.png)

Verfügbare Interaktionswerkzeuge in der 3D-Szene:

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
