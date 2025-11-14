# **GIS-Projekt** WS2025/26 @ HS Mainz
Prof. Dr.-Ing. Yu Feng

---

## Übung 1: Fahrzeugerkennung mit YOLO

**Ziele der Übung**

- Ein vortrainiertes Modell auf Luftbilder anwenden und **Fahrzeuge detektieren**.

- Auf [Hugging Face Models](https://huggingface.co/models) nach **„VisDrone“** oder **„UAV“** suchen, **weitere Modelle** herunterladen und vergleichen. e.g., https://huggingface.co/erbayat; https://huggingface.co/Mahadih534/YoloV8-VisDrone

- **WMS**-Kartenausschnitte in **verschiedenen Maßstäben** abrufen, die Detektion ausführen und bewerten, **welcher Maßstab für ein GIS‑Projekt** am besten geeignet ist.

---

## Übung 2: Parkplatzanalyse und Dichteschätzung

### Aufgabe 1: Fahrzeugstatistik

In dieser Aufgabe werden Punktdaten (Fahrzeugdetektionen) und Polygondaten (Parkflächen) mit **Pandas** und **GeoPandas** verarbeitet, um **Frequency** (Häufigkeit) und **Intensity** (Dichte pro m²) zu berechnen.

### Verwendung von *Pandas* und *GeoPandas*

Für die Analyse kommen insbesondere folgende **GeoPandas-Funktionen** und **Pandas-Methoden** zum Einsatz:

- **`geopandas.sjoin()`**  
  → räumlicher Join (Point-in-Polygon), weist jedem Punkt die Zone zu, in der er liegt.

- **`GeoDataFrame.area`**  
  → berechnet die Fläche jeder Parkzone (Polygon).

- **`DataFrame.groupby().size()`**  
  → zählt, wie viele Punkte pro Zone liegen (Frequency).

### Vorgehen zur Berechnung von Frequency & Intensity

1. **Spatial Join (GeoPandas)**  
   Weist Fahrzeugpunkte ihren Parkzonen zu:  
   ```python
   joined = gpd.sjoin(points_gdf, zones_gdf, predicate="within")
   ```

2. **Frequency berechnen (Pandas)**
   Anzahl der Fahrzeugdetektionen pro Zone:

   ```python
   freq = joined.groupby("zone_id").size().reset_index(name="frequency")
   ```

3. **Fläche der Polygone bestimmen (GeoPandas)**

   ```python
   zones_gdf["area"] = zones_gdf.geometry.area
   ```

4. **Intensity berechnen**

   ```python
   zones_gdf = zones_gdf.merge(freq, on="zone_id", how="left")
   zones_gdf["intensity"] = zones_gdf["frequency"] / zones_gdf["area"]
   ```

5. **Kartografische Darstellung (GeoPandas)**
   Choroplethenkarte:

   ```python
   zones_gdf.plot(column="intensity", cmap="OrRd", legend=True)
   ```

---

### Aufgabe 2: Kernel-Dichteschätzung (KDE)

Ziel ist die Erzeugung einer kontinuierlichen Dichtekarte eines bestimmten Parkplatzes.

#### Relevante Python-Bibliotheken

* **GeoPandas**
  – Selektion der Punkte im gewünschten Parkplatzpolygon.
* **SciPy**
  – Kernel Density Estimation über `scipy.stats.gaussian_kde`.
* **NumPy**
  – Erstellung von Analyse-Grids.

### Kritische Parameter

* **Bandwidth**
  steuert die Glättung der Dichteoberfläche.
* **Grid Resolution (Zellengröße)**
  kleiner als Bandwidth → feinere Dichtekarte.

### Vorgehen

1. Aufbereitung der Punktkoordinaten als numerische Arrays  
   (Extraktion der X-/Y-Werte aus den Geometrien mittels NumPy).

2. Durchführung der Kernel-Dichteschätzung  
   mit `scipy.stats.gaussian_kde` unter Angabe einer geeigneten Bandbreite.

3. Erstellung eines regelmäßigen Analyse-Grids  
   (NumPy: gleichmäßige Gitterpunkte innerhalb des Untersuchungsgebiets).

4. Berechnung der Dichtewerte  
   durch Auswertung der KDE-Funktion für alle Rasterzellen.

5. Kartografische Darstellung  
   als kontinuierliche Dichtekarte bzw. Heatmap zur Visualisierung der Parkaktivität.


---

### Übersicht: Relevante *GeoPandas* / *Pandas* Funktionen

| Schritt                 | Funktion                   | Bibliothek    |
| ----------------------- | -------------------------- | ------------- |
| Punkt-Polygon-Zuordnung | `gpd.sjoin()`              | **GeoPandas** |
| Polygonefläche          | `GeoDataFrame.area`        | **GeoPandas** |
| Frequency zählen        | `groupby().size()`         | **Pandas**    |
| Dichte berechnen        | `frequency / area`         | —             |
| KDE                     | `scipy.stats.gaussian_kde` | SciPy         |

