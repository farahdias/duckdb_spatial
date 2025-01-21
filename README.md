# Comparison GeoPandas and DuckDB on Spatial

Both GeoPandas and DuckDB can be used for spatial data manipulation, but their performance can vary based on the type of operation being performed, the size of the dataset, and the computational resources available. Here's a breakdown of their differences and how they perform in terms of time spent on spatial operations compared to traditional Pandas operations.

## Data Source

- Point of interest data on Indonesia area from OpenStreetMap
- Polygon city level from Indonesia Government

## Function
Both use source from pandas and result given on pandas to good comparison result

GeoPandas:

```python
def within_geopandas(df_point, df_polygon):
    # Create Point geometry from longitude and latitude
    geometry = [Point(lon, lat) for lon, lat in zip(df_point['longitude'], df_point['latitude'])]
    df_point = gp.GeoDataFrame(df_point, geometry=geometry, crs="EPSG:4326")  # assuming lat/lon is in WGS84 (EPSG:4326)
    gdf_joined = gp.sjoin(df_point, df_polygon, how="inner", predicate="within")
    return gdf_joined
```
DuckDB:

```python
def within_duckdb(df_point, df_polygon):
    
    points_df = df_point.copy()
    polygons_df = df_polygon.copy()
    
    # Convert polygons and points to WKT format (as text)
    polygons_df['geometry'] =  polygons_df['geometry'].apply(lambda geom: geom.wkt)
    
    # Connect to DuckDB
    con = duckdb.connect()
    
    # Register the DataFrames as tables in DuckDB
    con.register('points', points_df)
    con.register('polygons', polygons_df)
    
    query = """
        INSTALL spatial;
        LOAD spatial;
        SELECT *
        FROM points AS p
        JOIN polygons AS poly
        ON ST_Within(ST_Point(longitude, latitude), ST_GeomFromText(poly.geometry)) = TRUE
    """

    result = con.execute(query).fetchdf()
    return result
```
