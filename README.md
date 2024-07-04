# final_project
Here's an example of a `README.txt` file for your project:

---

# Paris Neighborhoods and Traffic Counts Visualization Project

## Introduction

This project aims to visualize traffic counting points and Paris neighborhoods on a map, using CSV files as the data source and creating GeoJSON files for import into Tableau.

## Project Steps

### 1. Data Preparation

#### a. Import Libraries and Load CSV Files

```python
import pandas as pd
import geopandas as gpd
from shapely.geometry import Point, Polygon
import json

# Load the CSV files
df_comptage = pd.read_csv("comptages-routiers-permanents.csv", sep=";")
df_quartiers = pd.read_csv("quartier_paris.csv", sep=";")
```

#### b. Convert Traffic Count Points Data to GeoDataFrame

```python
# Extract latitude and longitude coordinates
df_comptage[['latitude', 'longitude']] = df_comptage['geo_point_2d'].str.split(', ', expand=True)
df_comptage['latitude'] = df_comptage['latitude'].astype(float)
df_comptage['longitude'] = df_comptage['longitude'].astype(float)

# Select necessary columns and remove duplicates
df_unique_points = df_comptage[['Identifiant arc', 'Libelle', 'latitude', 'longitude']].drop_duplicates()

# Convert points to GeoDataFrame
geometry_points = [Point(lon, lat) for lon, lat in zip(df_unique_points['longitude'], df_unique_points['latitude'])]
gdf_points = gpd.GeoDataFrame(df_unique_points, geometry=geometry_points, crs="EPSG:4326")
```

#### c. Convert Neighborhood Data to GeoDataFrame

```python
def parse_polygon(polygon_str):
    coordinates = json.loads(polygon_str)['coordinates'][0]
    coordinates = [(float(lon), float(lat)) for lon, lat in coordinates]
    return Polygon(coordinates)

df_quartiers['geometry'] = df_quartiers['Geometry'].apply(parse_polygon)

# Convert neighborhoods to GeoDataFrame
gdf_quartiers = gpd.GeoDataFrame(df_quartiers, geometry='geometry', crs="EPSG:4326")
```

#### d. Spatial Join to Find the Corresponding Neighborhood for Each Point

```python
# Perform a spatial join to find the neighborhood for each point
joined = gpd.sjoin(gdf_points, gdf_quartiers, predicate='within', how='left')

# Add the 'Quartier' column to the points
df_unique_points['Quartier'] = joined['L_QU']
```

#### e. Save Results to CSV and GeoJSON Files

```python
# Save the result to a CSV file
df_unique_points.to_csv("df_unique_points_with_quartiers.csv", index=False)

# Save the result to a GeoJSON file
gdf_points['Quartier'] = joined['L_QU']
gdf_points.to_file("df_unique_points_with_quartiers.geojson", driver="GeoJSON")
```

### 2. Importing Files into Tableau

1. **Open Tableau:**
   - Click on "Connect" in the top left.
   - Select "File" and choose "GeoJSON".
   - Import the files `df_unique_points_with_quartiers.geojson` and `quartier_paris.geojson`.

2. **Add Neighborhood Polygons:**
   - Go to a new worksheet.
   - Drag the `quartier_paris.geojson` file into the view.
   - Tableau will automatically detect the polygon geometries.

3. **Add Traffic Count Points:**
   - Stay in the same worksheet.
   - Drag the `df_unique_points_with_quartiers.geojson` file into the view.
   - Tableau will automatically detect the point geometries.

4. **Customize the Map:**
   - Click on the "Marks" card and ensure polygons are displayed as "Polygons" and points as "Points".
   - Adjust the colors and sizes of the marks to easily differentiate between neighborhood polygons and traffic count points.

5. **Synchronize Axes (if needed):**
   - If the points and polygons do not align correctly, synchronize the axes by right-clicking on the axis and selecting "Synchronize Axis".

## Conclusion

This project demonstrates how to prepare, transform, and visualize spatial data using Python and Tableau. The provided steps guide you through data preparation, conversion to appropriate formats, and visualization in Tableau.

---

Feel free to edit and expand this README as necessary for your project.
