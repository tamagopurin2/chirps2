# CHIRPS2 Precipitation Data Analysis and Visualization Script

This Python script processes and visualizes CHIRPS2 annual precipitation data using the `xarray`, `matplotlib`, and `cartopy` libraries. It reads a NetCDF file, extracts data for a specific region, calculates average precipitation, creates a map, and exports the processed data as a GeoTIFF file.

## Table of Contents

1. [Introduction](#introduction)
2. [Features](#features)
3. [Getting Started](#getting-started)
4. [Usage](#usage)
5. [Code Explanation](#code-explanation)
6. [License](#license)
7. [Acknowledgements](#acknowledgements)

## 1. Introduction

This script is designed to analyze and visualize CHIRPS2 annual precipitation data. It performs the following operations:

* Reads precipitation data from a NetCDF file.
* Extracts data for a user-defined latitude and longitude range.
* Calculates the average precipitation for each grid cell.
* Creates a map of the average precipitation using `cartopy`.
* Exports the processed precipitation data as a GeoTIFF file.

## 2. Features

* Easy-to-use: Simple setup and intuitive workflow.
* Customizable: Users can specify the region of interest and customize the visualization.
* Open-source: Freely available and modifiable under the MIT license.

## 3. Getting Started

### Prerequisites

Before running the script, ensure you have the following:

* Python 3 installed on your system.
* The required Python libraries installed. You can install them using pip:

```bash
pip install xarray matplotlib cartopy netCDF4 rasterio
```

## 4. Data Acquisition
