# NOAA GFS Weather Visualizer

This project is a Python-based web application that downloads forecast data from the National Oceanic and Atmospheric Administration (NOAA) Global Forecast System (GFS), extracts specific weather parameters for a given location, and visualizes the data using a 3D graphical panel.

> ** Disclaimer: Personal Project & Estimation Model**
> This application is a personal project developed for educational and visualization purposes only. The metrics presented-including the Air Quality Index (AQI) and RealFeel calculations-are **estimates based on heuristic models** and should not be used for critical decision-making, health safety, or as a substitute for official meteorological data. The GFS data ingestion is subject to network stability and server availability; please consult official weather services for reliable, production-grade information.

---

## Features

* **Data Ingestion:** Downloads raw GRIB2 format forecast files directly from NOAA's public Amazon S3 buckets.
* **Fallback Logic:** Automatically checks and shifts to the previous day's data if the current day's data is not yet fully uploaded to NOAA servers.
* **Data Processing:** Uses `xarray` and `cfgrib` to parse specific atmospheric layers for temperature, cloud cover, precipitation, and wind vectors.
* **3D Visualization:** Generates a 6-panel 3D Matplotlib graphic displaying temperature gradients, volumetric cloud shapes, and rain vectors based on wind speed.
* **Calculated Metrics:** Provides an **estimated** Atmospheric Clarity Proxy (AQI) and a custom RealFeel temperature calculation based on meteorological heuristics.
* **Web Interface:** Provides a local web interface built with Gradio to input locations and view outputs.

---

## System Requirements

Because the application parses raw binary GRIB2 files, it requires an external system-level dependency alongside Python packages.

### 1. System Dependency
The Python library `cfgrib` requires the **ECMWF ecCodes** binary package to be installed on your operating system.

* **macOS (via Homebrew):**
    brew install eccodes

* **Linux (Ubuntu/Debian):**
    sudo apt-get install libeccodes-dev

* **Windows:**
    Installation via Anaconda/Conda is recommended to handle the binary compilation:
    conda install -c conda-forge eccodes

### 2. Python Libraries
Install the required Python packages using pip:
    pip install numpy xarray cfgrib matplotlib geopy gradio

---

## Project Structure and Logic

* **Geocoding:** Converts user-input address strings into precise latitude and longitude values via the `geopy` library.
* **Download Pipeline:** Downloads target `.grib2` files corresponding to 4-hour intervals for a 24-hour period.
* **Extraction:** Coordinates are converted to the 0 to 360 degree grid array mapping format used by global weather models. The script reads temperature at 2 meters above ground, total cloud cover, surface precipitation, and u/v wind components.
* **Rendering:** Generates 3D boxes representing each 4-hour window, rendering custom shapes for weather conditions.
* **Cleanup:** Automatically deletes the downloaded `.grib2` data and temporary `.idx` index files after processing to prevent disk storage build-up.

---

## How to Run

1. Save the application script to a local file, for example, `main.py`.
2. Execute the script from your terminal:
    python main.py
3. Open your web browser and navigate to the local address provided in the terminal, which defaults to: http://localhost:8080

---

## Technical Considerations

* **File Deletion:** The pipeline handles cleanups via a `finally` block to verify files are unlinked even if data extraction fails mid-loop.
* **API Constraints:** Geocoding relies on OpenStreetMap's Nominatim server, which requires a stable internet connection and may time out under heavy server requests.
* **Estimated Metrics Formulation:** The "AQI" displayed is a proprietary estimation formula derived from temperature, wind speed, and precipitation density to proxy atmospheric conditions. It is not an official measurement of airborne particulate matter.
    Raw Proxy = 65 + (Latitude % 7) * 18 + (Max_Temp * 1.3) + (Max_Wind * -1.8)
    Final Proxy = max(12, min(380, Raw Proxy * (1.0 - (Max_Rain * 0.22))))
