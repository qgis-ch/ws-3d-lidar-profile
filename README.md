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

![image](https://user-images.githubusercontent.com/884476/233046481-53f58ce7-24e6-4164-aeb2-91bf321e71e6.png)

The following tools are available for interaction with the 3D scene:

![image](https://user-images.githubusercontent.com/884476/233043748-7d8ce262-d77c-4ef9-89ba-7e72d44e94cd.png)

## Navigation in the 3D-Scene

### With the mouse
* Left mouse button: change position in the 3D space
* Ctrl-left mouse button: looking around / change of the field of view
* Middle mouse button (or Shift-left mouse button): Rotate the whole scene. With the first click you set the rotation center then with dragging you rotate the whole scene
* Right mouse button: zooming in and out

### With the keyboard
* Cursor keys up/down: move forward and backward
* Cursor keys left/right: move left and right
* Shift and cursor left/right: rotate the whole scene
* Shift and cursor up/down: tilt the whole scene
* Ctrl Cursor left/right: looking around / change of the field of view
* Ctrl Cursor oben/unten: looking up and down
* Page up: take the elevator/helicopter up while remaining in the current position (change the altitude of the viewer)
* Page down: take the elevator/helicopter down while remaining in the current position (change the altitude of the viewer)

### With the Navigations-Widget
* Rotation using the compass rose
* Buttons up/down to move forwards/backwards
* Buttons left and right to move left/right
* Plus/Minus Buttons for zooming in/out
* Extra Buttons for tilting the whole scene

![image](https://user-images.githubusercontent.com/884476/224546406-c54decda-6b73-4350-80df-4a590217fc79.png)

## Styling of 3D Data

The styling properties of the 3D graphics of a layer can be found in the "Layer Properties" → "3D View":

![image](https://user-images.githubusercontent.com/884476/225347198-190a4518-c550-4332-803a-47e1c6977af5.png)

If a layer has a 3D representation assigned, then the 2D representation in the 3D view is ignored, if not, then the 2D representation is draped over the terrain model.

In contrast to the 2D representation, there are fewer options in the 3D representation and also no composite symbols. However, with the rule-based 3D representation, you can draw an object several times, e.g. to compose a tree from a cylinder (trunk) and a sphere (crown):

![image](https://user-images.githubusercontent.com/884476/225525097-c149637b-b03f-47c2-bbe4-2ad92a35d330.png)

Settings for the tree trunk (cylinder):
![image](https://user-images.githubusercontent.com/884476/225525630-efc29f8f-53a0-48ee-887b-ae03a6068b81.png)

Settings for the tree crown (sphere):
![image](https://user-images.githubusercontent.com/884476/225526455-10bd4afe-d962-409c-bc26-a29a8abc6dad.png)

## Styling of LiDAR Point Cloud Data

Point cloud data can be represented in different ways:
* single color
* RGB values (if present in the LiDAR data, not present in the case of the Solothurn data)
* Classification attributes, e.g. building, roof, water, trees, etc. (present in the Solothurn data)
* Color ramps based on a single attribute, e.g. a ramp according to intensity of the received signal after reflection (Roofs or sealed surfaces like streets have a higher value then e.g. trees in the signal strength)

![image](https://user-images.githubusercontent.com/884476/225384014-2134a698-8364-40c4-8660-7d7a48378686.png)

Normally not all points are displayed, but thinned out points. The point budget (e.g. 5 million) defines the maximum number of points to be displayed simultaneously.

## Creating an Elevation Profile
The elevation profile panel can be enabled in menu "View" → "Elevation Profile". A separate dock panel is displayed:
![image](https://user-images.githubusercontent.com/884476/225532488-30c55b85-4d3a-4aac-922e-bbdf28c6a1b4.png)

The profile line can be drawn directly in the map or taken from an existing line (e.g. road axis).

Double-clicking on a layer opens the display properties related to the profile:
![image](https://user-images.githubusercontent.com/884476/225533778-bc823eb2-1c1f-467a-8cd6-c6417d56823b.png)

In the example, you can see the profile representation of the "Railway" layer - a line layer - but when it is cut with a profile line, it is represented as a "Point". It is necessary to configure the height constraint, representation type and point representation.

For point symbols, it is also recommended to set the vertical anchor point of the symbol to " bottom" so that the symbol really "touches down" on the profile line:

![image](https://user-images.githubusercontent.com/884476/225534450-baa588ef-8971-40ae-8a8f-a8be43b6ad83.png)

![image](https://user-images.githubusercontent.com/884476/225534941-82faac1b-c07f-4858-a6dd-9113752a3caf.png)
Unfortunately, the symbols in the terrain height profile do not respect the length units (map units), which means that the symbols are always displayed the same size when zooming in and out. Hopefully this will be improved in future versions!

## Using the elevation profile in the layout
A terrain height profile can be used in the map layout:
![image](https://user-images.githubusercontent.com/884476/225548078-7c8ab875-2a35-4b62-8969-22e0811731b6.png)
In the context-sensitive settings section, the properties of the profile, such as intervals for axes, labels, grid lines, etc. can be set.

## Outlook

### In the Pipeline (from the [previous 3D crowdfunding-Project](https://www.lutraconsulting.co.uk/crowdfunding/pointcloud-processing-qgis/))
* [Improvements of the 3D measurement tool](https://github.com/qgis/QGIS/pull/52208) (Display of the Measurement line)

Point cloud processing provider (comes with QGIS 3.32 and requires PDAL 2.5, see [Github PR](https://github.com/qgis/QGIS/pull/52182))
* Extract metadata from point cloud datasets: Number of points, extents, KBS, etc.
* Convert point cloud formats.
* Reproject point cloud datasets
* Set CRS for point cloud dataset.
* Clip: Clip point clouds by polygons.
* Filter: extract subsets according to PDAL expressions (e.g. categories, Z-value ranges, intensity, etc.)
* Merge: combine multiple point cloud datasets into one file
* Tiling: break point cloud data into tiles
* Thin: create thinned point cloud datasets
* Extent: create polygons with "footprint" (extent) of point cloud dataset
* Density: generate raster files with "number of points" in each raster cell value
* Export to raster: generate 2D raster surface
* Export to Raster via TIN: generate 2D raster surface via TIN interpolation
* Export to vector: export the 3D points to a PointZ vector file

All processing providers mentioned above can be called via the tool box (single analysis step), used in batch mode or be part of a graphical processing model.

### Personal wishes for improvements
* Various improvements in the elevation profile tool - e.g. better choice of scale in the profile tool, fixing scale of both axis, support of map units for point and line symbology
* Nicer display of line geometries (today they are either displayed pixelated (from the 2D map image draped on them) or they disappear partially under the terrain). Workaround: display 3D lines with offset just above the surface (e.g. 0.5m above the surface), so they don't partially disappear under the 3D surface).
* 3D attribute data query: hit object should be better highlighted, e.g.: with wireframe, different color or bounding box.
* Better measurement tool with snapping and visual feedback
