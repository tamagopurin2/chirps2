---
title: "CHIRPS2 Precipitation Data Analysis and Visualization Script"
---

# CHIRPS2 Precipitation Data Analysis and Visualization Script

This Python script utilizes the `xarray`, `matplotlib`, and `cartopy` libraries to process and visualize CHIRPS2 annual precipitation data. It reads NetCDF files, extracts data for a specified region, calculates the average precipitation, generates maps, and exports the processed data as a GeoTIFF file.

## Table of Contents

1. [Introduction](#introduction)
2. [Requirements](#requirements)
3. [Usage](#usage)
4. [Code Explanation](#code-explanation)
5. [License](#license)
6. [Acknowledgments](#acknowledgments)

## Introduction

This script analyzes and visualizes CHIRPS2 annual precipitation data. It performs the following tasks:

* Reads precipitation data from a NetCDF file.
* Extracts data within a user-defined latitude and longitude range.
* Computes the average precipitation for each grid cell.
* Creates a precipitation map using `cartopy`.
* Exports the processed precipitation data as a GeoTIFF file.

## Requirements

Before running the script, set up a Python 3 environment with Jupyter Notebook (recommended, though optional). The script has been tested with Python 3.10.12.

Install the required libraries using `pip`:

```bash
pip install xarray matplotlib cartopy netCDF4 rasterio
```

For Conda users:

```bash
conda install xarray matplotlib cartopy netCDF4 rasterio
```

For `uv` users:

```bash
uv add xarray matplotlib cartopy netCDF4 rasterio
```

## Usage

1. **Download Data:** Obtain the CHIRPS2 annual precipitation NetCDF file (e.g., `chirps-v2.0.annual.nc`) and place it in the designated `work_dir` directory.  https://www.chc.ucsb.edu/data

2. **Run the Script:** Execute the script step by step. The script performs the following operations:

* Reads the NetCDF file.
* Extracts data based on a specified latitude and longitude range.
* Computes the average precipitation.
* Displays a precipitation map.
* Saves the average precipitation data as a GeoTIFF file (`average_prec.tif`) in `work_dir`.

## Code Explanation

```python
import os
import netCDF4
import xarray as xr

# Specify working directory and data
work_dir = "/home/nknazs/CHIRPS/data"
data = "chirps-v2.0.annual.nc"  # Input file name
output_file = "average_prec.tif"  # Output file

# Load file
fpath = os.path.join(work_dir, data)

# Open dataset
ds = xr.open_dataset(fpath)
```

### Data Path:

* `work_dir` specifies the directory containing the NetCDF file.
* `data` specifies the NetCDF file name.
* Ensure these settings match your environment. On Windows, use double backslashes:

```python
work_dir = "C:\path\to\file"
```

### Data Extraction:
Extracts the target region using latitude and longitude slicing:

```python
# Extract data
lat = slice(40, 45)  # Latitude
lon = slice(72, 82)  # Longitude

ds  = ds.sel(latitude=lat, longitude=lon)
```

### Data Aggregation:
Computes the average precipitation per grid cell:

```python
# Compute average precipitation per grid cell
mean_prec_grid = ds.groupby(['latitude', 'longitude'], squeeze=False).mean()
# Compute time-averaged precipitation
mean_prec_2d = mean_prec_grid['precip'].mean(dim='time')

print(mean_prec_grid)
```

### Visualization:
Uses `cartopy` and `matplotlib` to generate a precipitation map:

```python
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature
from cartopy.mpl.gridliner import LONGITUDE_FORMATTER, LATITUDE_FORMATTER

plt.figure(figsize=(10, 6))
ax = plt.axes(projection=ccrs.PlateCarree())
ax.add_feature(cfeature.COASTLINE)
ax.add_feature(cfeature.BORDERS, linestyle=':')

# Display precipitation using colormap
im = ax.contourf(mean_prec_grid.longitude, mean_prec_grid.latitude, mean_prec_2d, cmap='jet')

# Add color bar
plt.colorbar(im, orientation='vertical', label='Precipitation (mm/year)')

# Set display range
lat_min, lat_max = lat.start, lat.stop
lon_min, lon_max = lon.start, lon.stop
lat_buffer, lon_buffer = 2, 2
extent = [lon_min - lon_buffer, lon_max + lon_buffer, lat_min - lat_buffer, lat_max + lat_buffer]
ax.set_extent(extent)

# Display latitude/longitude grid
gl = ax.gridlines(crs=ccrs.PlateCarree(), draw_labels=True, linewidth=1, color='gray', alpha=0.5, linestyle='--')
gl.top_labels = False
gl.right_labels = False
gl.xformatter = LONGITUDE_FORMATTER
gl.yformatter = LATITUDE_FORMATTER

plt.title('Average Precipitation')
plt.show()
```

### GeoTIFF Export:
Exports the processed precipitation data as a GeoTIFF file using `rioxarray`:

```python
fpath_output = os.path.join(work_dir, output_file)
mean_prec_2d.rio.to_raster(fpath_output, crs="EPSG:4326")  # Set coordinate system to WGS84
```

## License
This script is provided under the MIT license.

## Acknowledgments
This script utilizes CHIRPS data.
