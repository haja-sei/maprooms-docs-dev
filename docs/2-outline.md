# 2. Outline

## 2.1. Summary of this documentation

This documentation explains how to understand, install, configure, develop, and maintain Maprooms. It follows the workflow from preparing a development environment to creating products, configuring their presentation, and maintaining a local implementation.


### 2.1.1. Overview

[Chapter 1](1-overview.md) introduces Maprooms and the technical approach used by the Python implementation.

- [1.1. Introduction](1-overview.md#11-introduction): Purpose of Maprooms, the ENACTS context, national and regional implementations, and the transition from INGRID to Python.
- [1.2. Technical aspects of the new Maproom](1-overview.md#12-technical-aspects-of-the-new-maproom): Flask, country-specific datasets and configuration, supporting tools, Python calculations, Jinja templates, and JavaScript.
- [1.3. Typical interaction with a Maproom](1-overview.md#13-typical-interaction-with-a-maproom): Navigation, page rendering, calculation requests, and the display of maps and charts.

### 2.1.2. Requirements

[Chapter 3](3-requirement.md) describes the environment and access required before installation.

- [3.1. Operating system](3-requirement.md#31-operating-system): Supported development environments and considerations for Linux, macOS, and Windows through WSL.
- [3.2. Development environment](3-requirement.md#32-development-environment): Python, required libraries, frontend dependencies, GitHub access, SSH authentication, repository cloning, and the dst_api submodule.
- [3.3. Storage](3-requirement.md#33-storage): Application files, source and prepared datasets, runtime data, caches, and databases.

### 2.1.3. Installation

[Chapter 4](4-installation.md) explains how to prepare and verify a local Maproom installation.

- [4.1. Installation of the Maproom](4-installation.md#41-installation-of-the-maproom): Application directories, the installation Bash script, and local configuration files.
- [4.2. Data preparation](4-installation.md#42-data-preparation): Dataset identifiers, variable mappings, NetCDF inputs, Zarr outputs, preparation scripts, and the inclusion of new datasets.
- [4.3. Preparation of local files](4-installation.md#43-preparation-of-local-files): Institutional information, language settings, product configuration, geographic boundaries, and basemaps.
- [4.4. Test the Maproom locally](4-installation.md#44-test-the-maproom-locally): Starting the application, opening it in a browser, checking pages and products, and investigating errors.

### 2.1.4. Maproom Structure

[Chapter 5](5-maproom-structure.md) explains the application architecture and the relationships between its components.

- [5.1. Architecture overview](5-maproom-structure.md#51-architecture-overview): Repository structure, Flask packages, domains and components, blueprints, configuration hierarchy, and the role of dev- files.
- [5.2. Interconnection between application components](5-maproom-structure.md#52-interconnection-between-application-components): Connections between templates, page scripts, shared JavaScript, routes, Python calculations, and datasets.

### 2.1.5. Create and Register a Module

[Chapter 6](6-module.md) describes how to create the initial structure of a product page and register it in the application.

- [6.1. Create module page navigation](6-module.md#61-create-module-page-navigation): Navigation-card entries, product-page entries, icons, and icon-loader functions.
- [6.2. Create the module structure](6-module.md#62-create-the-module-structure): Domain and component selection, HTML and JavaScript placeholders, and page YAML configuration.
- [6.3. Connect the module to a blueprint](6-module.md#63-connect-the-module-to-a-blueprint): Existing or new blueprints, registration, and communication through POST requests.
- [6.4. Test the initial rendering](6-module.md#64-test-the-initial-rendering): Navigation checks, template loading, and verification of the initial page structure.

### 2.1.6. Module Content

[Chapter 7](7-content.md) develops the connections required to turn the initial page into a working product.

- [7.1. HTML templates](7-content.md#71-html-templates): Shared layout inheritance, page blocks, Jinja macros, configuration values, map controls, chart previews, and expanded-chart dialogs.
- [7.2. Static JavaScript](7-content.md#72-static-javascript): Page initialization, user events, map updates, geographic selection, chart requests, and shared browser-side functions.
- [7.3. Python scripts](7-content.md#73-python-scripts): Package organization, icon generation, spatial and time-series calculations, shared utilities, caching, statistical functions, and dataset extraction.

### 2.1.7. Map and Chart Rendering

[Chapter 8](8-control-render.md) focuses on presentation and product-specific interface behavior.

- [8.1. Map rendering](8-control-render.md#81-map-rendering): Raster display, basemaps, opacity, colors, legends, image export, boundaries, regional clipping, geographic selection, and pages with different navigation or chart requirements.
- [8.2. Expanded-chart rendering](8-control-render.md#82-expanded-chart-rendering): Visible-container timing, presentation settings, calculation requests and local redraws, Plotly formatting, responsive sizing, chart information, and image export.

### 2.1.8. Maintain the Maproom

[Chapter 9](9-maintain.md) describes the routine processes used to maintain and share a Maproom implementation.

- [9.1. Git process](9-maintain.md#91-git-process): Working branches, inspecting changes, selective staging, commits, fetching and merging updates, resolving conflicts, testing, pushing branches, and creating pull requests.
- [9.2. dev- files and local datasets-config.yaml](9-maintain.md#92-dev-files-and-local-datasets-configyaml): The update_maprooms.sh workflow, installation paths, application and submodule updates, configuration-copy rules, preserved local files, and Python dependency installation.
