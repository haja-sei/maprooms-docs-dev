
# 4. Installation

This chapter describes how to install Maprooms for local development, prepare its datasets, configure country-specific files, and verify the application.

Complete the requirements in [chapter 3](3-requirement.md) before starting.

The paths shown below retain the example directory structure used in this documentation. Adapt them to your computer.

##  4.1. Installation of the Maproom
###     4.1.1. Prepare application and dataset directories

First, prepare the directories that will contain the application and its datasets.

Keeping the `DATA/` and `MAPROOMS/` directories side by side is recommended because it makes their responsibilities easier to distinguish. They can also be stored elsewhere, provided all configured paths are correct.

For example:

```text
ENACTS/
├── DATA/
│   ├── ENACTS/
│   ├── CMIP6/
│   ├── SHP/
│   ├── SOILGRIDS/
│   └── FORECASTS/
├── MAPROOMS/
│   ├── maprooms/
│   ├── venv/
│   └── maprooms_data/
└── installation_doc/
    ├── install_maprooms.sh
    └── create_data.sh
```

The application uses three distinct types of location:

| Location | Purpose |
|---|---|
| Maprooms source directory | Application code, templates, static files, and configuration |
| MAPROOMS_DATA_DIR in config.py | Runtime cache, user files, and SQLite databases |
| Dataset paths in app/yaml/datasets-config.yaml | Climate datasets, geographic boundaries, forecasts, and other supporting data |

The datasets-config.yaml file connects the application to its datasets.

Changing MAPROOMS_DATA_DIR does not automatically update the dataset paths. Check both configuration locations.


###      4.1.2. Use the installation Bash script

Installation files are supplied separately to help developers install Maprooms.

Change only the path settings identified in those scripts to match your computer. Changes to installation logic, package selection, Python version, repository source, or service configuration should follow the maintainer's instructions.

The supplied local installation package contains a script named install_maprooms.sh. Use the actual filename provided in your package.

Its opening section is similar to:

```bash
#!/bin/bash

# sudo ufw status
sudo ufw allow from any to any port 8484 proto tcp
sudo apt install python3-pip -y
sudo apt install python3-venv -y

#BRANCH="modal_customization_3"
#FORK="https://github.com/Kaleab1914/maprooms.git"

ROOT_DIR="/home/aranaivoson/ENACTS/MAPROOMS"
PYTHON_VER="3.14"

...

```
Update ROOT_DIR to the directory that will contain maprooms/, venv/, and maprooms_data/.

The selected Python interpreter must already be installed. For the supplied configuration, verify that python3.14 is available.

The installer performs the following main steps:

1. Runs the supplied Ubuntu package and firewall commands.
2. Creates the Python virtual environment.
3. Clones the Maprooms repository.
4. Initializes the dst_api submodule.
5. Copies dev- YAML templates to active local YAML files.
6. Installs Python dependencies from requirements.txt.
7. Applies the supplied Flask-JSGlue compatibility adjustment.
8. Creates the initial runtime-data directory structure.

Run the installation script from the directory containing it:

```bash
cd /home/aranaivoson/ENACTS/installation_doc
bash install_maprooms.sh
```

The script is intended for a fresh installation. If an application checkout already exists, follow the maintainer's procedure for that checkout rather than repeating the fresh-clone process.

The YAML copy step can overwrite existing active configuration files. Preserve customized local settings when working with an existing installation.

The script creates a virtual environment named venv. To use it after installation, start from the directory containing venv/ and maprooms/:

```bash
cd /home/aranaivoson/ENACTS/MAPROOMS
source venv/bin/activate
cd maprooms
```

Keep the environment active when running Maproom Python commands.


###     4.1.3. Prepare local configuration files for development

During installation, files beginning with dev- are copied to corresponding active files without that prefix.

      For example:

      app/yaml/dev-datasets-config.yaml
                       ↓ copy
      app/yaml/datasets-config.yaml

The contents are copied; paths and country-specific values are not automatically adapted to the developer's computer.

The application reads the active files without the dev- prefix.

The main local configuration resources include:

```html
config.py
app/yaml/*.yaml
app/<domain>/<component>/yaml/*.yaml
```
Some country-specific files are supplied separately.

The installer does not create config.py or run.py. Obtain these from the configuration package and place them in the top-level application directory:

        maprooms/config.py
        maprooms/run.py

In config.py, set the runtime data directory to the appropriate local path:

```bash
MAPROOMS_DATA_DIR = "/home/aranaivoson/ENACTS/MAPROOMS/maprooms_data"
```

Retain the other required settings in the supplied configuration file.

The language lists must describe the same languages in the same order:

      LANGUAGES_CODE
      LANGUAGES_NAME
      LANGUAGES_LABEL
      LANGUAGES_LOCALE

Complete the dataset configuration in [section 4.2.1](#421-configure-datasets-configyaml) and the country-specific configuration in [section 4.3](#43-preparation-of-local-files) before running data preparation.


##  4.2. Data preparation

###    4.2.1. Configure *datasets-config.yaml*

The active *app/yaml/datasets-config.yaml* file is the main connection between the dataset directories and the Maproom application.

Begin with the supplied configuration and adapt its paths and metadata to the available local datasets.

The following examples are excerpts. Preserve the other configuration sections required by your implementation.


####       4.2.1.1. Dataset identifiers and variable mappings

The configuration identifies dataset groups and maps descriptive variable names to internal identifiers.

For example:

```yaml
datasets:
    ALL:
        variables:
            rainfall: precip
            maximum_temperature: tmax
            minimum_temperature: tmin
            reference_evapotranspiration: et0
            potential_evapotranspiration: pet
            mean_relative_humidity: rhmean
            maximum_relative_humidity: rhmax
            minimum_relative_humidity: rhmin
        daily:
            netcdf:
                precip: 
                    dir: /data/maprooms/DATA/ENACTS/NETCDF/ALL/rr_daily
                    format: chirps-v2_%Y%m%d.nc
                    name: Daily rainfall
                    varid: precip
                    lon: Lon
                    lat: Lat
                    time: Time
                    prec: float
                    units: mm
                    missval: -99
                    compute: False
...

```
ENACTS uses identifiers such as **ALL**, **MON**, **ADJ**, and **CLM** to distinguish dataset types. Refer to the ENACTS dataset-type documentation for their definitions, and use the identifiers supported by your implementation.

Variable identifiers must remain consistent across dataset configuration, page configuration, and calculation requests.

In this example, rainfall is associated with the internal identifier precip.

A page selects its dataset separately. For example, the monthly climate analysis configuration includes:

```yaml
dataset:
    use: ALL
    variables:
        precip: [precip]
        tmax: [tmax]
        tmin: [tmin]
```

Adding a dataset to the shared configuration does not automatically add it to every product. The relevant product must also select and support it.


####       4.2.1.2. NetCDF paths, filenames, and coordinates

The configuration describes the source NetCDF files, including their locations, variable identifiers, coordinates, and metadata.

| Setting | Purpose |
|---|---|
| dir | Directory containing the source files |
| format | Filename pattern, including date placeholders |
| name | Descriptive dataset or variable name |
| varid | Variable identifier inside the source file |
| lon, lat, time | Configured coordinate names |
| prec | Configured numeric precision |
| units | Units associated with the variable |
| missval | Configured missing-value identifier |
| compute | Whether the variable is derived from other data |

Check these settings against the actual files. Coordinate and variable names are case-sensitive.

The current NetCDF conversion code derives dates from filenames using format. A correct filename pattern and compatible source-file structure are therefore essential.

The configuration also supports derived data. For example, monthly values can be calculated from daily or dekadal inputs.

Relevant settings include:

```yaml
compute: True
input: dekadal
function: sum
```

The appropriate function depends on the variable and product. Examples include *sum*, *mean*, and supported specialized functions such as *et0_hargreaves_modified*.

Specialized calculations can require additional settings. Follow an existing supported configuration for the same calculation rather than adding a function name alone.

For derived datasets written to their own Zarr stores, settings such as write_zarr and dir_zarr are also required by the preparation code.


####       4.2.1.3. Zarr output paths and processing settings

To prepare Zarr data from the configured NetCDF sources, specify the output directory and chunk dimensions.

For example:

```yaml
            zarr_dir: /data/maprooms/DATA/ENACTS/ZARR/ALL
            chunks:
                time: 1000
                lat: 50
                lon: 50
```

These settings belong under the relevant dataset timescale, alongside netcdf.

- zarr_dir identifies the output directory.
- chunks defines how the data is divided along its dimensions.

The values above are taken from the supplied configuration. Appropriate chunk sizes depend on the dataset and its access patterns.

Also check the configured output location for climatological products:

      climatology.zarr_dir

The account running preparation must be able to read the source files and write to the configured output directories.



###    4.2.2. Compute and convert datasets from NetCDF to Zarr

Maprooms uses functions from dst_api to convert source datasets, prepare derived data, and compute climatological products.

The following scripts are available in the top-level application directory:

      convert2zarr.py
      computed2zarr.py
      climato2zarr.py

Each script has a main() entry point that calls the corresponding preparation function.

| Script | Function | Implementation file | Responsibility |
|---|---|---|---|
| convert2zarr.py | create_zarr_datasets() | app/dst_api/scripts/zarrdata.py | Convert configured source NetCDF variables to Zarr |
| computed2zarr.py | create_computed_zarr_datasets() | app/dst_api/scripts/zarrdata.py | Write supported derived datasets configured for Zarr output |
| climato2zarr.py | compute_some_climatogies() | app/dst_api/scripts/zarrclim.py | Prepare mean/standard-deviation and percentile climatologies |

The conversion function processes source variables with compute: False where a Zarr output directory is configured.

The computed-data preparation function processes supported derived variables with compute: True and additional output settings such as *write_zarr: True*.

A Bash script named create_data.sh is supplied to run the preparation stages in order:
```bash
#!/bin/bash

ROOT_DIR="/home/aranaivoson/ENACTS/MAPROOMS/"

PYTHON_CMD="${ROOT_DIR}/venv/bin/python"
MAPROOM_DIR="${ROOT_DIR}/maprooms"

"$PYTHON_CMD" "${MAPROOM_DIR}/convert2zarr.py"
"$PYTHON_CMD" "${MAPROOM_DIR}/computed2zarr.py"
"$PYTHON_CMD" "${MAPROOM_DIR}/climato2zarr.py"

# Additional preparation commands may be supplied separately.

```

As with the installation script, update ROOT_DIR to match your computer. The script uses the Maproom virtual environment and active application configuration.

Run it from the directory containing the supplied script:

```bash
cd /home/aranaivoson/ENACTS/installation_doc
bash create_data.sh
```

Complete all required local configuration first. These preparation scripts import application modules that load shared settings and initialize supporting resources.

Inspect the output from each preparation stage. Reaching the end of the Bash script does not by itself confirm that all stages succeeded.

The current climatology function uses **1991–2020** as its reference period. *climato2zarr.py* calls it for *mean-stdev* and *percentile* products with *overwrite=False*.

These preprocessing parameters are separate from reference-period controls displayed in a product interface.

If prepared Zarr datasets have already been supplied, verify their paths, variables, and coverage before deciding whether further preparation is required.


###     4.2.3. Include new data in the Maproom

When updating an existing dataset or adding a new one, update the relevant active configuration.

For example, forecast datasets are configured in their own section: 

```yaml
...
forecast_data:
    seasonal:
        rainfall:
            dir: /data/maprooms/DATA/FORECASTS/SEASONAL_PRECIP
            lon: "X"
            lat: "Y"
        temperature:
            dir: /data/maprooms/DATA/FORECASTS/SEASONAL_TEMP
            lon: "X"
            lat: "Y"
    tendays:
        dir: /data/maprooms/DATA/FORECASTS/TEN_DAYS
        dimname:
            lon: lon
            lat: lat
            time: time
....

```
The required information includes the dataset location and its coordinate or variable settings.

Follow this sequence:

1. Update the relevant section of *datasets-config.yaml*.
2. Confirm that the source files match the expected naming and structure.
3. Run the preparation process required by that dataset type.
4. Update the product configuration or calculation code where necessary.
5. Verify variables, dates, maps, and charts in the application.

Forecasts and projections have their own processing requirements. Adding their paths does not automatically make them inputs to the general ENACTS NetCDF-to-Zarr conversion scripts.

For source datasets already converted to Zarr, the conversion code checks dates present in the existing store. Correcting a NetCDF file for an existing date does not necessarily refresh that date in Zarr. Follow the maintenance procedure when correcting historical data.


##  4.3. Preparation of local files

NMHS information is specific to each country or implementation. The relevant files are supplied or adapted alongside the other country-specific resources.

Complete these settings before running data preparation or starting the application.


**Meteorological service information**:

      app/yaml/dev-nmhs-info.yaml
      app/yaml/nmhs-info.yaml

Check the institution name, website, page title, logos, initial map center, zoom level, and timezone.

Place the required images in the locations expected by the configuration, commonly *app/static/images/*.

**Data directory and language settings:**

      config.py

Check **MAPROOMS_DATA_DIR** and the language lists.

**Shared application settings:**

      app/yaml/dev-global.yaml
      app/yaml/global.yaml

These files provide shared interface text and menu settings.

**Country-specific boundaries and basemap settings:**

      app/yaml/dev-subdivision-config.yaml
      app/yaml/subdivision-config.yaml

      app/yaml/dev-basemap.yaml
      app/yaml/basemap.yaml

The geographic data directory is configured through *geom_layers.shp_dir* in *datasets-config.yaml*.

Boundary filenames and attribute fields in *subdivision-config.yaml* must match the actual geographic files. Check the administrative or basin-selection defaults used by each domain.

**Page-specific settings:**

```html
app/<domain>/<component>/yaml/dev-<page>.yaml
app/<domain>/<component>/yaml/<page>.yaml
```

These files define product descriptions, variable choices, and interface controls.

Maintain the active files without the dev- prefix for the local implementation. Changes to the shared templates should follow the [project's maintenance workflow](9-maintain.md#91-git-process).



##  4.4. Test the Maproom locally

### 4.4.1. Start the application

After installation, configuration, and the required data preparation, activate the Python environment and start Maprooms:

```bash
cd /home/aranaivoson/ENACTS/MAPROOMS

# Activate the Python environment.
source venv/bin/activate

# Enter the application directory.
cd maprooms

# Start the development server.
python run.py
```

The supplied **run.py** starts Flask on port **8484** by default.

It enables debug mode and binds to **0.0.0.0**. This entry point is intended for development; production deployment requires the appropriate server configuration.

### 4.4.2. Open the application in the browser

Open either local address:

```url
http://localhost:8484
http://127.0.0.1:8484
```
### 4.4.3. Verify pages, maps, and charts

Use a product whose required datasets have been configured and prepared.

Check that:

1. The homepage displays the expected institutional information.
2. Navigation cards and product pages open.
3. Styles, icons, and browser libraries load.
4. A configured product displays a climate map.
5. Selecting a point or region displays chart data.
6. Variable and date controls match the available datasets.
7. Expanded charts open and display results.
8. Language selection displays the expected text.

For the monthly climate analysis example, the following files provide a useful path for inspection:

      app/__init__.py
      app/scripts/maproom_items.py
      app/climate/analysis/yaml/monthly.yaml
      app/climate/analysis/templates/monthly.html
      app/climate/analysis/templates/block-js.html
      app/climate/analysis/static/js/monthly.js
      app/climate/analysis/index.py
      app/climate/analysis/scripts/

A successful page load verifies the initial rendering. A working map and chart also verify communication between JavaScript, Flask endpoints, calculation functions, and datasets.

### 4.4.4. Inspect errors when verification fails

Use the terminal output and the browser's Console and Network panels to identify the failing stage.

| Symptom | First checks |
|---|---|
| Application fails during startup | Virtual environment, config.py, active YAML files, and dst_api checkout |
| Errors reference another computer's paths | MAPROOMS_DATA_DIR and paths in datasets-config.yaml |
| Interface styling or controls are missing | Browser asset requests and frontend vendor preparation |
| Map or chart requests fail | Request parameters, configured variables, prepared data, and date coverage |
| Geographic selection fails | Geographic directory, boundary filenames, and attribute fields |
| Configuration changes appear unchanged | Confirm the active file was edited and restart the development process where needed |

Some calculation endpoints return an error object with status: -1. Inspect the response body as well as the HTTP status when diagnosing a failed request.

