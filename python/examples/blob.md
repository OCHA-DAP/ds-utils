---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.16.3
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

## Azure Storage Container I/O

This notebook walks through some basic examples of using the functions in `blob.py` to read and write data to Azure blob storage.

```python
import pandas as pd
import numpy as np
from src import blob_utils as blob
from shapely.geometry import Point
import xarray as xr
import geopandas as gpd

stage = "dev"
project_prefix = "ds-demo"
```

```python
blob.get_container_client()
```

### 1. Tabular data (csv, parquet)


Create a dummy dataframe

```python
df = pd.DataFrame(np.random.randn(6, 4), columns=list("ABCD"))
```

Write dataframe to blob as CSV and Parquet. By default, this will write to the `projects` container.

```python
blob.upload_csv_to_blob(
    df=df,
    blob_name=f"{project_prefix}/demo.csv",
    stage=stage)

blob.upload_parquet_to_blob(
    df=df,
    blob_name=f"{project_prefix}/demo.parquet",
    stage=stage)
```

Now read the files that we just uploaded

```python
df_csv = blob.load_csv_from_blob(
    blob_name=f"{project_prefix}/demo.csv",
    stage=stage
)
df_parquet = blob.load_parquet_from_blob(
    blob_name=f"{project_prefix}/demo.parquet",
    stage=stage
)
```

### 2. Geospatial vector data (shp)


Create a dummy geodataframe

```python
geometry = [
    Point(-122.4194, 37.7749),  # San Francisco
    Point(-74.0060, 40.7128),   # New York
    Point(-87.6298, 41.8781)    # Chicago
]
gdf = gpd.GeoDataFrame({
    'id': [1, 2, 3],
    'name': ['Point A', 'Point B', 'Point C'],
    'geometry': geometry
}, crs='EPSG:4326')
```

Write `gdf` to blob as Shapefile

```python
blob.upload_gdf_to_blob(
    gdf=gdf,
    blob_name=f"{project_prefix}/demo.shp",
    stage=stage
)
```

Read Shapefile from blob

```python
gdf_shp = blob.load_gdf_from_blob(
    blob_name=f"{project_prefix}/demo.shp",
    stage=stage
)
```

### 3. Geospatial gridded data (tiff)


Create a dummy xarray DataArray

```python
data = np.random.rand(4, 4) * 100  # 4x4 grid of random values
coords = {
    'y': range(4),
    'x': range(4)
}
da = xr.DataArray(
    data=data,
    dims=['y', 'x'],
    coords=coords,
    attrs={'units': 'random units', 'description': 'Sample 2D data'}
)
```

Write `da` to blob as a Cloud-Optimized-GeoTiff

```python
blob.upload_cog_to_blob(
    da=da,
    blob_name=f"{project_prefix}/demo.tif",
    stage=stage
)
```

Read COG from blob

```python
da_cog = blob.open_blob_cog(
    blob_name=f"{project_prefix}/demo.tif",
    stage=stage
)
```
