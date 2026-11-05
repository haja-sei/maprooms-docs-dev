
# 8. Map and Chart Rendering

[Chapter 7](7-content.md) explained how to connect templates, controls, JavaScript requests, and Python calculations. This chapter focuses on how the returned results are displayed and how developers can adapt the map and chart behavior to a product.

Most rendering functionality is already available through shared application functions. Product development therefore involves selecting the appropriate behavior, providing compatible results, and configuring the interface for the features the page supports.

The main development decisions concern:

- Raster display, basemaps, boundaries, opacity, colors, and legends.
- Geographic selection and regional display.
- Pages with different combinations of maps, navigation, and charts.
- Expanded-chart appearance, sizing, settings, and image export.

Refer to [Chapter 7](7-content.md) for macro definitions, request construction, endpoint connections, and calculation functions.

## 8.1. Map rendering

The Leaflet map combines several elements:

- A basemap providing geographic context.
- A raster overlay displaying the calculated result.
- Boundary layers showing administrative divisions, basins, or other geographic units.
- Controls for navigation, selection, appearance, and downloads.

These elements have separate responsibilities. Changing the basemap or boundary style affects the presentation, while changing the dataset, variable, or calculation requires the appropriate product request.

The following files contain the main rendering functions and configuration.

| File | Path | Main responsibility |
|---|---|---|
| leaflet-map.js | app/static/js/ | Display raster images, manage basemaps and boundaries, control opacity, handle geographic selection, and clip the displayed raster |
| leaflet-map-settings.js | app/static/js/ | Connect product settings to map requests and update product-specific map information |
| colorkeys.js | app/static/js/ | Create and display HTML or image-based legends |
| colors.js | app/static/js/ | Register shared color scales in the browser and generate chart colors |
| leaflet-download.js | app/static/js/ | Export the displayed Leaflet map as a PNG image |
| rstudio-leaflet-2.0.3.js | app/static/js/ | Provide the pixel-preserving raster helper used by the default overlay mode |
| imagepng.py | app/scripts/ | Generate map images, geographic bounds, and legend information |
| colorbar.py | app/scripts/ | Prepare color scales, format legend values, and validate custom colors |
| *_colors.py* | app/scripts/ | Define shared custom palettes through COLORS_MAPROOM |
| subdivision.py | app/scripts/ | Read configured boundaries and prepare their GeoJSON representation |
| subdivision-config.yaml | app/yaml/ | Describe boundary layers and their use in each domain |
| basemap.yaml | app/yaml/ | Define the basemap choices displayed in the interface |
| colorbar.yaml | app/yaml/ | Define the available palette names and default extension colors |
| layers.html | app/templates/ | Provide the controls for boundaries, regional display, basemaps, opacity, and legends |
| map-pages.css, leaflet-site.css, and colorkeys.css | app/static/css/ | Define map dimensions, toolbar appearance, Leaflet styling, and legend presentation |

Changes in shared files affect every product that uses the corresponding functions. Keep product-specific differences in the page script or page styles when the shared behavior already meets the other products' requirements.

### 8.1.1. Raster display and map appearance

The main rendering functions are maintained in:

      app/static/js/leaflet-map.js

| Function | Rendering responsibility |
|---|---|
| displayRasterImage() | Coordinate the raster, legend, and associated display controls |
| leafletRasterImage() | Replace the raster overlay and apply its display options |
| createRasterImage() | Select the raster rendering mode |
| changeLeafletBasemapStyle() | Replace the basemap selected in the layer controls |
| setRasterImageOpacity() | Connect the opacity slider to the current raster |
| displayMapSubdivisions() | Connect boundary selection to the displayed layers |
| displayMapRegions() | Manage country and regional display |
| removeRasterImage() | Remove the current raster when replacing or clearing it |

#### 8.1.1.1. Preserve the raster rendering structure

The standard map request helper checks the outer response status and passes the successful payload to *displayRasterImage()*.

Inside that payload, the renderer expects the following structure:

| Payload field | Purpose |
|---|---|
| data.png | Encoded map image |
| data.bounds | Geographic corners used to position the image |
| ckeys.title | Legend title |
| ckeys.labels | Labels for the HTML legend and color-break controls |
| ckeys.colors | Colors used by the HTML legend |
| ckeys.raster | Select whether the legend is displayed as an image |
| ckeys.png | Legend image when the image-based legend mode is used |

This distinction matters when implementing another spatial calculation: the renderer receives the successful payload, rather than the complete response containing the outer status field.

Keep the image and its geographic bounds together. The shared image-generation functions prepare compatible bounds and account for the map projection used by the raster display.

A custom image generator must preserve the same alignment between the image pixels and the geographic extent. Otherwise, the raster can appear displaced relative to the boundaries, particularly when displaying or clipping a large geographic area.

The raster helper accepts these main options:

| Option | Default | Purpose |
|---|---|---|
| id | png-overlay-id | Identify the overlay that will be replaced |
| opacity | 1 | Set the initial raster opacity |
| type | pixels | Select the raster rendering mode |
| fitbounds | false | Determine whether the raster helper fits the map to the image extent |

The pixels mode uses *addRasterImage()* to preserve the appearance of grid cells when zooming. The image mode uses Leaflet's image overlay.

Keep a consistent overlay identifier across redraws so that the shared helper can replace the previous raster.

The fitbounds option controls the raster helper's behavior. Boundary and regional-display functions can also change the map extent.

The application retains the active map through MAP_BE. Reuse the map returned by *createLeafletTileLayer()* when updating the product, and replace its raster through the shared rendering functions.

#### 8.1.1.2. Configure basemaps, opacity, and display styles

Basemap labels and choices are defined in:

      app/yaml/basemap.yaml

The corresponding basemap implementations are selected by:

      selectLeafletBasemapStyle()
      in:
      *app/static/js/leaflet-map.js*

Adding a basemap name to the YAML file makes it available to the interface, but the selected identifier must also be supported by the JavaScript function.

When adding a provider, connect its identifier to the required tile-layer implementation and provider settings. For example, the existing Mapbox helper reads its access token from the institutional options passed to the map.

The opacity control is connected through *setRasterImageOpacity()*. It changes the current map.rasterImage directly and does not require another climate calculation.

The shared styles are divided between:

| Style file | Main use |
|---|---|
| map-pages.css | Map height, page toolbar, expanded-chart containers, and application controls |
| leaflet-site.css | Leaflet control fonts, popup appearance, and related map styling |
| colorkeys.css | Legend structure and presentation |

The default *.div-map-container* height is defined in map-pages.css. A product can override this height in its own template when it needs a different page arrangement.

After changing the dimensions of a map that is already initialized, call:

```js
map.invalidateSize();
```

The shared map functions also manage interaction with offcanvas panels and selection controls. They temporarily disable map navigation while those controls are being used.

When adding another control above the map, ensure that clicks intended for the control do not also trigger geographic selection.

#### 8.1.1.3. Keep raster colors and legends consistent

The map image and its legend must describe the same color scale.

The main functions are:

| Function | File | Purpose |
|---|---|---|
| colorbarSetDefault() | leaflet-map.js | Set the initial palette and color-break controls |
| colorbarGetData() | leaflet-map.js | Read the color settings included in a map request |
| colorbarSettings() | leaflet-map.js | Manage the color-control fields and palette previews |
| displayColorBar() | colorkeys.js | Display an HTML legend |
| displayColorBarRaster() | colorkeys.js | Display a legend supplied as an image |
| createCategoricalColorKeyH() | colorkeys.js | Create the horizontal legend for categorical results |
| format_ColorScale() | colorbar.py | Prepare legend labels and colors |
| check_invalid_colors() | colorbar.py | Validate custom colors used by the calculation |

The ckeys.raster field controls the legend presentation. It does not change the rendering mode of the main map raster.

When ckeys.raster is true, the browser displays ckeys.png. Otherwise, it creates the HTML legend. The horizontal HTML legend also supports categorical information when the returned color-key structure includes categories.

Palette choices and palette definitions have different roles:

- app/yaml/colorbar.yaml lists the palette names offered by the controls.
- app/scripts/_colors.py defines shared custom palettes.
- app/static/js/colors.js registers these shared palettes in the browser through /maproom_colors.

When adding a custom palette, define it centrally and expose its name in the palette list so that Python rendering and browser previews use the same palette.

Color and break changes must be included in the next product map request to regenerate the raster and its matching legend. In the monthly Climate analysis page, the redraw button provides that request.

Basemap and opacity changes can update the existing display directly.

The legend-title input updates the current HTML legend title. An image-based legend contains its title in the generated image, so changing that title requires regenerating the legend image.

### 8.1.2. Download the map

The Leaflet image export is implemented by:

      downloadLeafletMap()
      in:
      app/static/js/leaflet-download.js

*createLeafletTileLayer()* connects this function to the shared save button:

      map-control-save

The export captures the map as displayed in the browser. It:

1. Temporarily hides navigation and selected controls.
2. Preserves the legend styles used in the map.
3. Captures the map container with dom-to-image-more.
4. Downloads the result as leaflet_map.png.
5. Restores the controls and styles.

The shared layout loads the required export library.

When changing the map layout, verify the exported image as well as the browser display. New controls, differently positioned legends, or external tile sources can affect the captured result.

Update the export selectors in leaflet-download.js when new controls need to be excluded from the image.

The Matplotlib button is a separate action. The shared *create_modal_matplotlib()* macro creates its dialog, and existing page scripts can open that dialog. Those steps alone do not generate an additional map figure or download.

A product that supports a separate generated figure must connect its rendering and export behavior. Hide the action on pages that do not provide it.

### 8.1.3. Display boundaries and select geographic areas

Boundary display, regional clipping, and spatial averaging serve different purposes.

| Operation | Main control or setting | Effect |
|---|---|---|
| Display boundary outlines | map-display-subdivision | Choose the geographic layers drawn over the map |
| Display the country or a region | select-country-region and select-region-name | Change the visible geographic area and raster mask |
| Choose an extraction method | support-spatial-average | Select a grid point or a polygon layer for calculations |
| Choose a polygon | list-spatial-average or a map click | Identify the polygon passed to the chart calculation |

Displaying a boundary does not automatically select it for a calculation.

#### 8.1.3.1. Boundary configuration

Boundary preparation and local geographic paths are covered in the [installation chapter](4-installation.md#43-preparation-of-local-files). For rendering, the important configuration is:

      app/yaml/subdivision-config.yaml

| Setting | Effect on the interface or renderer |
|---|---|
| subdivision.<key>.field | Identify the source attribute used for polygon names and selection |
| subdivision.<key>.region | Identify the source attribute used to associate polygons with a region |
| subdivision.<key>.order | Determine the layer's position in the styling hierarchy |
| subdivision.<key>.group | Place the layer in the corresponding geographic group in the selection controls |
| subdivision.<key>.location | Enable administrative location information for selected grid points |
| subdivision_sel | Select the default displayed boundary layer for each domain |
| subdivision_clip | Select the boundary layer used for regional display and clipping |
| subdivision_map | Associate each domain with the group used by its layer controls |

The *get_subdivisions_data()* function in *app/scripts/subdivision.py* reads the configured files, converts their geometry to EPSG:4326, and normalizes the relevant properties to field and, where configured, region.

The browser receives this data through */map_subdivisions_data* and makes it available through SUBDIV_BE.

The shared boundary functions depend on those normalized properties. Custom boundary-loading code must preserve the property names expected by the selection and regional-filtering functions.

Initial boundary styles are generated by *addSubdivisionStyles()*. The order values are used to construct the styling hierarchy, so keep them unique and sequential when extending the configured layer list.

The style controls created by *createSubdivisionStyles()* allow users to change boundary color, line width, and opacity. These changes update the browser's layer styles.

#### 8.1.3.2. Visual clipping from calculation geometry

*displayMapRegions()* changes the visible map when the user selects a region.

It uses the configured clipping layer and calls *maskImageMapRegions()* to mask the existing raster image. The original image is retained in browser storage so that the full-country view can be restored.

Stored map results are checked by *isStoredLeafletMapValid()* before reuse. The current implementation checks their structure, domain, and page path.

Visual masking changes the displayed image. It does not perform a new spatial-average calculation.

Chart calculations receive their geographic selection through *queryParamsSpatialAverage()*, as described in [Chapter 7](7-content.md#7222-plotly-previewjs).

The geographic-selection helper also manages the map's click behavior. In the current implementation, changing the selection mode clears the map click handlers before attaching the appropriate point or polygon behavior.

Coordinate additional product-specific map interactions with this helper so that changing the selection mode preserves the intended behavior.

Selection initialization can also apply the current polygon or create an initial grid-point marker and invoke the chart callback. Enable it only when the page provides the corresponding callback and chart containers.

### 8.1.4. Adapt the map to the product page

A product can use the shared map without providing every navigation or chart feature.

The existing source contains several useful examples:

| Product | Source example | Relevant behavior |
|---|---|---|
| Model agreement | app/climate/projection/templates/model-agreement.html and static/js/model-agreement.js | Displays a map without chart-preview cards or expanded-chart initialization |
| Seasonal change | app/climate/projection/static/js/seasonal-change.js | Hides time navigation while retaining geographic selection and chart updates |
| Ten-day forecast | app/climate/forecast/static/js/ten-days.js | Hides the complete navigation container and initializes its map controls asynchronously |
| Agricultural decision support | app/agriculture/analysis/static/js/decision-support.js | Displays the map without connecting chart-preview selection callbacks |

For a page containing a map without charts:

- Keep the map container, map controls, and product map request.
- Include the preview cards and expanded-chart dialogs only when the product uses them.
- Connect geographic selection to chart callbacks only when the corresponding charts exist.
- Adjust the map height to use the available page space.

For example, model-agreement.html increases the map height to 88vh.

Time navigation and geographic selection are contained within the same navigation structure, but can be hidden separately:

| Required behavior | Page-level action |
|---|---|
| Hide time navigation and keep geographic selection | $('#div-map-time-navigation').removeClass('d-flex').addClass('d-none'); |
| Hide the complete time and geographic-selection navigation | $('#container-map-navigation').hide(); |
| Hide an unsupported Matplotlib action | $('#map-control-matplotlib').closest('.btns-map-control').hide(); |

The first action is already introduced in [Chapter 7](7-content.md#7213-trigger-map-and-chart-updates). The distinction here is that hiding the parent navigation container also hides the spatial-selection controls.

Keep the shared markup required by the rendering utilities when adapting the page. Hiding an unused control can preserve those structural dependencies.

Where a control initializer is asynchronous, wait for it before making the first map request. The ten-day forecast page demonstrates this pattern with *setOffCanvas10DaysFcstMapControl()*.

## 8.2. Expanded-chart rendering

[Chapter 7](7-content.md) explained how preview buttons, dialog controls, requests, and Python calculations are connected. This section focuses on the rendering behavior that developers need when changing an existing chart or adding another chart type.

The main files are:

| File | Path | Main responsibility |
|---|---|---|
| expand-charts.js | app/static/js/ | Configure the dialog's behavior and connect its controls to the appropriate chart functions |
| plotly-expand.js | app/static/js/ | Format product results and construct Plotly traces and layouts |
| plotly.js | app/static/js/ | Provide shared layouts, themes, sizing, range selectors, language settings, cleanup, and image export |
| expand-charts-settings.js | app/static/js/ | Build the chart-settings interface, validate presentation options, and restore settings after redraws |
| chart-infos.js | app/static/js/ | Prepare chart information and descriptive download filenames |
| colors.js | app/static/js/ | Generate colors for charts containing multiple series |
| map-pages.css | app/static/css/ | Define expanded-dialog and Plotly-container dimensions |

### 8.2.1. Render charts when their containers are visible

The shared *expandModalCharts()* helper is maintained in:

      app/static/js/maprooms.js

It prepares the drawing container, connects the chart-settings interface, and schedules the chart callback after the dialog becomes visible.

This timing is important because a hidden dialog does not provide the same usable dimensions as a visible chart container.

The helper also accepts optional settings for:

| Argument | Purpose |
|---|---|
| plotElementSuffix | Identify the actual drawing element when it is a child of the standard chart container |
| colorsModule | Provide custom editing behavior for the chart's traces |
| axisKindResolver | Describe axes that need specific numeric or year-based handling |
| caseKeyResolver | Keep presentation settings separate for different chart configurations |
| disallowedTickUnits | Exclude date intervals that are unsuitable for the product |

Reuse this helper when adding another expanded chart so that drawing, settings, and dialog timing follow the shared behavior.

The standard drawing container uses .modal-expand-charts-plotly. Its dimensions are defined in map-pages.css.

When changing the dialog layout, preserve a usable width and height for the drawing container. The shared *getChartWidth()* and *getChartHeight()* functions use its dimensions, then its parent's dimensions, before applying a browser-size fallback.

### 8.2.2. Configure chart presentation and settings

The shared objects in *plotly.js* provide the starting point for chart presentation:

| Object or function | Purpose |
|---|---|
| expand_layout | Common margins, backgrounds, and axis presentation for expanded charts |
| plotly_config | Shared Plotly configuration, including responsive behavior |
| plotly_rangeslider | Default range-slider appearance |
| setPlotlyColors() | Axis and legend colors for the current theme |
| setPlotlyThemeColors() | Update chart presentation when the page theme changes |
| formatPlotlyHoverDate() | Prepare hover-date text for the supported temporal resolutions |

The default plotly_config hides the native Plotly modebar because the application supplies its own toolbar buttons.

Use product-specific layouts for differences that belong to one chart. Change the shared objects when the behavior should apply across the products that use them.

The chart-settings dialog is managed by *enablePlotlyChartSettings()* in:

      app/static/js/expand-charts-settings.js

It supports:

- Chart titles and axis labels.
- Title font, size, and color.
- Axis tick intervals.
- Trace colors and supported line or marker sizes.
- Secondary-axis settings when the chart provides a second vertical axis.
- Resetting the current chart configuration to its original presentation.

The settings interface validates text and tick intervals. Tick handling depends on the axis type and the space available for labels.

The dialog's close action dismisses it. Its close-and-apply action commits the edited settings.

For products with special axes, reuse the resolver helpers in expand-charts.js:

| Helper | Use |
|---|---|
| alwaysFloatAxis() | Treat the selected axis as a continuous numeric axis |
| parameterDrivenAxis() | Determine integer handling and units from the selected parameter |
| yearIndexAxis() | Treat numeric values as years |
| combineAxisResolvers() | Combine the rules required by a chart |
| makeCaseKeyResolver() | Identify the current combination of selected chart options |

For example, the raw-data chart separates settings according to the selected variable, series arrangement, and plot type.

Presentation settings are stored in JavaScript objects by chart identifier and case key during the current page lifetime. They can therefore survive a chart redraw or dialog reopening.

These settings are separate from the returned datasets stored through maproomDB. They are not a permanent user-preference store across page reloads.

The application wraps *Plotly.newPlot()* to capture defaults and reapply the appropriate saved presentation settings. Preserve the shared script-loading order when creating a page that uses this behavior.

A chart with specialized trace styling can provide a custom color module with *populate()*, *apply()*, and *restore()* methods.

### 8.2.3. Distinguish new requests from presentation redraws

A control change does not always require another Python calculation.

The monthly raw-data implementation in *setAnalysisExpandModalRaw()* provides a useful reference:

| Change | Existing behavior |
|---|---|
| Selected variable | Request the corresponding data |
| Replot action | Request data using the current controls |
| Seasonal calculation settings | Request the corresponding seasonal data |
| Single-series or yearly-series arrangement | Redraw from the stored ts_rawdata result |
| Supported bar or line presentation | Redraw from the stored ts_rawdata result |
| Start month used to arrange yearly series | Reformat and redraw the stored result |
| Title, trace style, or tick interval | Apply presentation settings to the chart |

Follow this distinction when connecting another control.

A control that changes the required dataset or calculation must use the product's request function. A control that reformats an already available result can call the corresponding display function.

Reuse the event namespaces and *.off(...).on(...)* pattern used by the dialog initializers when binding handlers that may be registered again after reopening a dialog.

The request construction and response handling remain as described in [section 7.2.2.3](7-content.md#7223-plotly-expandjs).

### 8.2.4. Format and display the returned values

The product-specific display functions in *plotly-expand.js* decide:

- Which Plotly trace types to create.
- How series are grouped or compared.
- Which values and dates appear on the axes.
- How units, labels, hover text, and legends are presented.
- Whether range controls are appropriate.

For example, *expand_analysis_format_rawdata()* can rearrange a returned time series into yearly comparisons before *expand_analysis_display_rawdata()* creates the traces.

This formatting is performed on a copy of the returned result, allowing the original stored data to be reused for another presentation.

After a display function has prepared its data traces and productLayout, it can finish with the shared rendering pattern:

```js
const layout = deepMerge(setPlotlyColors(), productLayout);

layout.width = getChartWidth(container);
layout.height = getChartHeight(container);

purgePlotlyChart(container);

Plotly.newPlot(
    container,
    data,
    deepMerge(expand_layout, layout),
    plotly_config
).then(() => {
    setPlotlyThemeColors(container);
    resizePlotlyChart(container);
});
```

Here, container is the identifier of the actual Plotly drawing element.

The *resizePlotlyChart()* helper updates the chart after browser resizing and removes its window-resize handler when the dialog closes.

The *purgePlotlyChartExpandModal()* helper connects dialog closure to Plotly cleanup. Reuse the shared cleanup functions when extending an existing modal.

Range controls should match the horizontal axis:

| Axis or display | Appropriate behavior |
|---|---|
| Chronological dates | Use addRangeselector() with date-based ranges |
| Numeric years | Use its year mode |
| Probability distributions, categories, or another incompatible axis | Hide the period-selection controls with showRangeselector() |

The existing range helper uses the shared toolbar structure to locate its buttons. Preserve that structure when adapting the modal markup described in [Chapter 7](7-content.md#7123-expanded-chart-modal-and-control-macros).

Date labels and hover text should use the shared formatting functions, particularly for dekadal and seasonal data. Their date representations should retain the meaning of the product's temporal resolution.

### 8.2.5. Chart information and image export

The expanded-chart toolbar provides separate information, settings, replot, and image-download actions.

Chart information is maintained in:

      app/static/js/chart-infos.js

The standard chart request helper calls *saveChartInfo()* with the query and successful result. This information is used by the information dialog and by the download filename.

When bypassing that helper for a custom chart, preserve the corresponding information update so that the displayed chart, information dialog, and filename refer to the same result.

The image-export function is:

      downloadPlotlyImageJPG(container, infoContainer = container)
      in:
      app/static/js/plotly.js

It:

1. Prepares a white-background version suitable for export.
2. Hides the range slider and range selector in the exported image.
3. Applies suitable axis, title, and legend colors.
4. Generates the image and adds the configured institutional logo when available.
5. Downloads a JPEG image.
6. Restores the chart layout used in the interface.

The logo is read from *MTO_INIT.chartLogo*, with *MTO_INIT.iconImage* as the fallback.

The filename is prepared by *getChartDownloadName()* using available chart information, including the variable and period.

When a chart is drawn inside a child container, pass that drawing element as container. If its information was saved against a parent container, provide that identifier as infoContainer.

For a new chart type, verify the complete rendering behavior: opening the dialog, changing its presentation, resizing it, switching themes, reopening it, and exporting its image. These checks exercise the rendering-specific connections described in this chapter.
