# Geospatial Analysis

## Visualizing geospatial data with GeoPandas
To visualize geospatial data by coding, we use the [GeoPandas](https://geopandas.org/en/stable/) library. We first load the data using the `gpd.read_file()` function and examine it using the `.head()` method. We can also use `loc`, `iloc`, and `isin` to select subsets of the data, and `value_counts()` method to see a list of unique feature values and how many times they appear in the data set. Finally, we can visualize the dataset using the `.plot()` method. To include multiple GeoDataFrames, we can define a basemap `ax` to ensure all the information are plotted on the same map.
```python
import geopandas as gpd

# Read in the data
full_data = gpd.read_file("../input/geospatial-learn-course-data/DEC_lands/DEC_lands/DEC_lands.shp")

# View the first five rows of the data
full_data.head()

# Subsetting data
data = full_data.loc[:, ["CLASS", "COUNTY", "geometry"]].copy()

# How many lands of each type are there?
data.CLASS.value_counts()

# Select lands that fall under the "WILD FOREST" or "WILDERNESS" category
wild_lands = data.loc[data.CLASS.isin(['WILD FOREST', 'WILDERNESS'])].copy()
wild_lands.head()

# Plotting
wild_lands.plot()

# Load multiple GeoDataFrames
# Campsites in New York state (Point)
POI_data = gpd.read_file("../input/geospatial-learn-course-data/DEC_pointsinterest/DEC_pointsinterest/Decptsofinterest.shp")
campsites = POI_data.loc[POI_data.ASSET=='PRIMITIVE CAMPSITE'].copy()

# Foot trails in New York state (LineString)
roads_trails = gpd.read_file("../input/geospatial-learn-course-data/DEC_roadstrails/DEC_roadstrails/Decroadstrails.shp")
trails = roads_trails.loc[roads_trails.ASSET=='FOOT TRAIL'].copy()

# County boundaries in New York state (Polygon)
counties = gpd.read_file("../input/geospatial-learn-course-data/NY_county_boundaries/NY_county_boundaries/NY_county_boundaries.shp")

# Plotting multiple GeoDataFrames
# Define a base map with county boundaries
ax = counties.plot(figsize=(10,10), color='none', edgecolor='gainsboro', zorder=3)

# Add wild lands, campsites, and foot trails to the base map
wild_lands.plot(color='lightgreen', ax=ax)
campsites.plot(color='maroon', markersize=2, ax=ax)
trails.plot(color='black', markersize=1, ax=ax)
```

## [Coordinate Reference Systems](https://www.kaggle.com/code/alexisbcook/coordinate-reference-systems)
**Coordinate reference systems (CRS)** show how the projected points correspond to real locations on Earth.

### Setting the CRS
In setting the CRS, we first create a Pandas DataFrame from a CSV file using `pd.read_csv()`. Then, we convert the Pandas DataFrame to a GeoPandas GeoDataFrame using `gpd.GeoDataFrame()`. From the columns containing latitude and longitude coordinates, we can create geometry (e.g., points) using `gpd.points_from_xy()`. Finally, we can set the appropriate CRS.
```python
import geopandas as gpd
import pandas as pd

# Create a DataFrame with health facilities in Ghana
facilities_df = pd.read_csv("../input/geospatial-learn-course-data/ghana/ghana/health_facilities.csv")

# Convert the DataFrame to a GeoDataFrame
facilities = gpd.GeoDataFrame(facilities_df, geometry=gpd.points_from_xy(facilities_df.Longitude, facilities_df.Latitude))

# Set the coordinate reference system (CRS) to EPSG 4326
facilities.crs = {'init': 'epsg:4326'}
```

### Re-Projecting
Sometimes, we have to deal with data that already has an existing CRS. When plotting multiple GeoDataFrames, it's important that they all use the same CRS. Changing the CRS is what we call **re-projecting** and we do this using the `to_crs()` method. The `to_crs()` method modifies only the "geometry" column: all other columns are left as-is.
```python
# Create a map
ax = regions.plot(figsize=(8,8), color='whitesmoke', linestyle=':', edgecolor='black')
facilities.to_crs(epsg=32630).plot(markersize=1, ax=ax)
```

### Attributes of geometric objects
All three types of geometric objects (i.e., points, lines, polygons) have built-in attributes that you can use to quickly analyze the dataset.
```python
# Get the x-coordinate of each point
facilities.geometry.head().x

# Calculate the area (in square meters) of each polygon in the GeoDataFrame 
regions.loc[:, "AREA"] = regions.geometry.area / 10**6

print("Area of Ghana: {} square kilometers".format(regions.AREA.sum()))
print("CRS:", regions.crs)
regions.head()
```

## Interactive Maps
