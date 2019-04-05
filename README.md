---
output:
  html_document: default
  pdf_document: default
---
# CUPN VEGETATION DASHBOARD
#### April 5, 2019
#### Ellen Cheng

&nbsp;

#### PURPOSE: Visualize CUPN forest monitoring data; Generate summary (figures and tables only) reports

&nbsp;

#### STEP 1. MAKE SURE REQUIRED PACKAGES ARE UPDATED
Make sure these R packages, if already installed, are updated to at least the versions specified in parentheses. If a package is not already installed, then the ‘Load libraries’ code chunk in the ‘CUPNVegFlex.RMD’ file will automatically install the current version:

* cowplot (v.0.9.2)
* flexdashboard (v.0.5.1)
* gridExtra (v.2.3)
* knitr (v.1.20)
* leaflet (v.2.0.1)
* magrittr (v.1.5)
* plotrix (v.3.7-4)
* RgoogleMaps (v.1.4.2)
* rmarkdown (v.1.10)
* rtf (v.0.4.13)
* shiny (v.1.1.0)
* shinyBS (v.0.61)
* shinyjs (v.1.0)
* shinyWidgets (v.0.4.3)
* tidyverse (v.1.2.1)

The following R packages should be updated via GitHub, which has a more recent version tan CRAN (as of August 2018). You will need to install and load the package ‘devtools’ to update the packages via Github.

* devtools::install_github("hadley/ggplot2") # v.2.2.1.9000
* devtools::install_github("ramnathv/htmlwidgets") # v.1.2.1
* devtools::install_github("ropensci/plotly") # v.4.7.1.9000

&nbsp;

#### STEP 2. MAKE SURE REQUIRED FILE FOLDERS ARE PRESENT
Make sure the following required file folders are present in the same directory as the Dashboard (‘CUPNVegFlex.RMD’):
* ‘Data_in’:  Put raw data files in this folder for Dashboard analysis.
* ‘logos’: This folder holds the logo image files used on the Dashboard.
* ‘Map_files’:  This folder holds files required for mapping (park boundary and google map RDS files, GIS shape files).
* ‘PDFs’: This folder holds .pdf files that explain the various pages of the Dashboard. They are accessed when a user clicks the ‘About this Page’ button on a Dashboard page.
* ‘Reports_out’: When reports are generated, the resulting files (.pdf or .doc) are stored in this folder.
* ‘RMD_files’: This folder holds the RMD files that are executed when raw data files are processed.
* ‘Temp_out’:  When raw data files are processed, the resulting output summary files (RMD) are stored in this folder.

&nbsp;

#### STEP 3. RUN THE DASHBOARD
Double-click the ‘CUPNVegFlex.RMD’ file to open it in RStudio. In RStudio, click the green triangle next to ‘Run Document’. The Dashboard is best viewed in a browser window, so in the Dashboard pop-up click the ‘Open in Browser’ option (top left corner). Follow the instructions on the main page of the Dashboard, for processing raw data to visualize with the Dashboard.

***
#### DOWNLOADING NEW GOOGLE MAPS FOR PARKS
This process is likely to change, but as of August 2018 the steps are:

1. Install and load the R package 'rlang'. To register your API key you will need version >= 0.2.1

2. Install and load the R package 'ggmap'. The most current version is available from GitHub, 'devtools::install_github("dkahle/ggmap", ref = "tidyup")'

3. Assign your API key to a variable, e.g., my_api <- "AIzaSyAgeq7...." (you can get an API key for free from the Google Cloud Platform, for Maps: https://cloud.google.com/maps-platform/?apis=maps)

4. In the RStudio console, type 'ggmap::register_google(key = my_api, account_type = "premium")'

5. The following code will read the CUPN shapefile, access the corresponding google map of the park, then combine the files in a single RDS that can be used by the Dashboard.

```{r}
CUPNmap <- readOGR("Map_files/CUPN_shapefiles/CUPN.shp") # UNIT_CODE has the 4-digit park code as a factor. No SubPark column.
CUPNmap <- spTransform(CUPNmap, CRS("+init=epsg:4326")) # make sure shapefiles are in same projection as google maps
CUPNmap@data$id <- rownames(CUPNmap@data)
CUPNmapbounds_df <- CUPNmap %>% broom::tidy() %>%
left_join(CUPNmap@data[,c("UNIT_CODE", "id")], by = "id")

ParkGoogleMaps_list <- list()
for (p in levels(CUPNmapbounds_df$UNIT_CODE)) { # for each Park
  subdat <- subset(CUPNmapbounds_df, UNIT_CODE == p)
  map_zoom <- min(MaxZoom(range(subdat$lat),range(subdat$long))) 

  ParkGoogleMaps_list[[p]] <- get_googlemap(
    center = c(lon = mean(range(subdat$long)), mean(range(subdat$lat))), zoom = map_zoom, key = my_api, maptype = "hybrid") # this is satellite + roads
  }

saveRDS(ParkGoogleMaps_list, file = "ParkGoogleMaps_list.RDS")
```

