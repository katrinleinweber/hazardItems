hazardItems
==============
[![Build Status](https://travis-ci.org/USGS-R/hazardItems.svg)](https://travis-ci.org/USGS-R/hazardItems)  
[![Coverage Status](https://coveralls.io/repos/USGS-R/hazardItems/badge.svg)](https://coveralls.io/r/USGS-R/hazardItems)

services for coastal change hazards items

Getting started in R is easy! Check out how to set up your local environment with <a href="http://owi.usgs.gov/R/resources.html">this helpful set of resources from the USGS Office of Water Information R Community</a>. At a minimum, you'll need to have <a href="http://cran.rstudio.com/">R installed on your machine</a>.



```{r setup, include=FALSE}
library(rmarkdown)
options(continue=" ")
options(width=60)
library(knitr)
library(hazardItems)

```


#Introduction


This package **hazardItems** provides services in support of coastal change hazards items for the Coastal Change Hazards Portal at <a href="http://marine.usgs.gov/coastalchangehazardsportal" target="_blank">marine.usgs.gov/coastalchangehazardsportal</a>.  
    
`historical.service`, `storm.service` and `vulnerability.service` parse metadata records and create summaries specific to their respective themes. The subtypes are defined by the attribute names.

`thumb.service` expects an input json.url and returns a thumbnail png for the item.

`refreshItemThumbnail` creates a new thumbnail PNG for the requested item ID and puts it to the user specified endpoint.

`addLayer` service creates a new item from a zipped shapefile from an authenticated user.

`template` service creates or replaces new child items based on an input id. 

`setBaseURL` is a supporting function for directing requests and posts to the proper development, testing or production tier 

`authenticateUser` authenticates user against the current specified endpoint, and `checkAuth` verifies if the user is still authenticated

`checkItemExists` is a supporting function that accepts an item ID and returns TRUE or FALSE depending on if the item exists on the currently specified tier/endpoint.

`saveTrack` allows a user to create or update the NHC NOAA hurricane track on the currently set endpoint. The details for controlling these layers are set in the hazardItems.yaml file. 

#Installation
To install **hazardItems** 

Install package with dependencies:
```{r, eval=FALSE}
install.packages("hazardItems",
repos = c("http://owi.usgs.gov/R"),
dependencies = TRUE, type = 'both')
```

#Updating 
To update **hazardItems** to the latest version, when you already have installed hazardItems before:

```{r, eval=FALSE}
update.packages(repos = c("http://owi.usgs.gov/R", "http://cran.rstudio.com/"))
```

#Item Metadata Packages

##Historical Metadata
`historical.service` parses the metadata record and creates a summary specific to the HISTORICAL theme, with subtype being defined by the attribute name. For example, to create a historical item from a local shapefile metadata file called NewJerseyS_shorelines.shp.xml and create a subtype named 'date':

```{r, results='hide'}
serviceEndpoint  <-  system.file('extdata',
"NewJerseyS_shorelines.shp.xml", package = 'hazardItems')
attribute	<-	'date'
summary	<-	historical.service(serviceEndpoint,attribute)
print(summary)
sink('output.json')
cat(summary)
sink()
```

##Storm Service Metadata
`storm.service` parses the metadata record and creates a summary specific to the STORM theme, with a subtype being defined by the attribute name. For example, to create a storm item from a remote url with an attribute named 'PCOL3':

```{r, results='hide'}
serviceEndpoint  <-  'http://olga.er.usgs.gov/data/NACCH/GOM_erosion_hazards_metadata.xml'
attribute	<-	'PCOL3'
summary	<-	storm.service(serviceEndpoint,attribute)
print(summary)
```

##Vulnerability Service Metadata
`vulnerability.service` parses the metadata record and creates a summary specific to the VULNERABILITY theme, with subtype being defined by the attribute name. The serviceEndpoint can be any valid xml file, either locally hosted or at a remote URL. For example, to create a vulnerability item from a local shapefile metadata file called `CVI_Pacific.shp.xml` and create an attribute named `SEA_LEVEL`:

```{r, results='hide'}
serviceEndpoint  <-  system.file('extdata',
"CVI_Pacific.shp.xml", package = 'hazardItems')
attribute  <-	'SEA_LEVEL'
summary	<-	vulnerability.service(serviceEndpoint,attribute)
print(summary)
```


##Real Time Storm Item Creation
A real-time storms event item is established in a special way. When it needs to be updated, the parent aggregation is dropped and re-created each time a model update is run and NACCH scientists want to update the data by using the `createStorm()` function. The parent item represents the storm. For example, "Hurricane Sandy" could be the title of a parent aggregation item for a real-time storm. The NOAA NHC Hurricane cone, track and points are all added automatically when the `createStorm()` function is run. 

<br />
<br />
An example workflow to create a new real time storm would involve the following:

```{r,eval=FALSE}
library(hazardItems)
setBaseURL("dev") #setBaseURL("prod") in the event of actual storm. setBaseURL("qa") in the event of testing on the qa server.
createStorm(templateId = NULL, "D:\\mkhdata\\model_output\\Sandy_CIDA060520150800.zip") #don't forget to double escape slashes if you are running this command in a Windows environment.
```

Where the model output shapefile is zipped into a zip file and pointed through in the arguments above.

When this function completes, the user must record the ID returned in the console (e.g. CAQw7M1) and use that argument in the function `templateId = "CAQw7M1")` for future updates to the same active storm.

##Updating a Real Time Storm Item
If a user wants to update the model run output on an existing real time storm item, they must have the templateId handy, and follow the example workflow to update the real time storm:

```{r,eval=FALSE}
library(hazardItems)
setBaseURL("dev") #setBaseURL("prod") in the event of actual storm.
createStorm(templateId = "CAQw7M1", "D:\\mkhdata\\model_output\\Sandy_CIDA060520151400.zip")
```

When this function is run to replace the existing out of date item, the previously posted parent and children are orphaned. Record the new templateId returned when the function is successful in order to replace with future updates.

#Adding Layers to the Portal 

The `addLayer` function accepts a zipped shapefile and returns an itemID. Users can use this function to post new shapefiles to the Portal. The zipped shapefile must be put into your environment's `extdata` directory and the package `hazardItems` must be rebuilt to be available to R. If the user is not currently authenticated, the `checkAuth` function will be called and the user will be asked for a valid username and password

To Use:
```{r,eval=FALSE}
addLayer("D:\\mkhdata\\new_data\\Sandy_CIDA.zip"))
```


#Rendering
##Thumbnail Generation 

The `thumb.service` accepts a json URL and creates a handsome thumbnail preview of the layer. For the CCH Portal, the thumbnails are used on the <a href="https://marine.usgs.gov/coastalchangehazardsportal/publish/item/">mediation page</a> as an item preview, in your Bucket and in the search results panel.

To use:
```{r,eval=FALSE}
serviceEndpoint <- 'http://marine.usgs.gov/coastalchangehazardsportal/data/item/CAkR645'
thumb.service(serviceEndpoint)
```


#Refresh/delete of various caches

There are a number of functions that are available to clear out cached contents when items change. 

##Delete Entire Application Tile Cache

The `deleteTileCache` function takes the cache URL set by `pkg.env$tile_cache` and drops the entire tile cache for the application. The user must be authenticated to perform this task. The second argument is unnecessary, user need only set the base URL for the proper endpoint (dev, qa or prod) and run `deleteTileCache()`. 

If you do a lot of changes (e.g. adding items, moving children around, changing geographic extens, editing geographic features, adding children to an aggregate (if it's a displayed child)), you should use this function to make sure that users are experiencing the latest changes. This cache is repopulated by application use.

To use:
```{r,eval=FALSE}
setBaseURL('dev') #setBaseURL("prod") in the event of production use.
tileCacheUrl <- pkg.env$tile_cache 
deleteTileCache() # if the user is not currently authenticated, the checkAuth function will be called and the user will be asked for a valid username and password
```

##Delete Download Cache 

The `deleteDownload' function accepts the itemID for an item and drops the cached download for the item. The user must be authenticated to perform this task. 

This is automatically run when save is done on the mediation page, and also cleans up ancestor items as well.

To use:
```{r,eval=FALSE}
setBaseURL('dev') #setBaseURL("prod") in the event of production use.
deleteDownload("CCNKrhr") #itemID you wish to delete the download cache for. if the user is not currently authenticated, the checkAuth function will be called and the user will be asked for a valid username and password
```

##Reseed the Download Cache for an Item

You can seseed the download for an item by calling:
```{r,eval=FALSE}
setBaseURL('dev') #setBaseURL("prod") in the event of production use.
downloadItem("CCNKrhr") #itemID you wish to refresh the download for. 
```

The item download is also refreshed upon request, by visiting the back of the card.


##Thumbnail Refresh

The `refreshItemThumbnail` accepts an item ID and will verify that the item exists, before putting a newly created thumbnail in the database.

To use:
```{r,eval=FALSE}
setBaseURL('dev') #setBaseURL("prod") in the event of production use.
refreshItemThumbnail("CCGftiy") #or whichever itemID you wish to regenerate and put
```

##Refreshing/deleting items and aggregations - Example Workflow

The workflow for changing an item or aggregation and needing to refresh the various caches/downloads/bits includes:

* Update the item via the <a href="https://marine.usgs.gov/coastalchangehazardsportal/publish/item/" target="_blank">mediation page</a>
* Delete the tile cache for the entire application with `deleteTileCache`
* Delete the download cache for the item with `deleteDownload` (rinse and repeat for all parent items)
* Regen the download cache for the item with `downloadItem` (rinse and repeat for all parent items)
* Regen the thumbnail for the item with `refreshItemThumbnail` (rinse and repeat for all parent items)

Alternatively, by visiting the <a href="https://marine.usgs.gov/coastalchangehazardsportal/publish/item/" target="_blank">mediation page</a> a user can replace items and and make changes to aggregations, which will trigger updates to the download files and thumbnails for the item and all ancestors/aggregations to which it belongs. 

Currently, when items are changed via the mediation page, all item and ancestor caches are cleared (thumbnail, download) automatically with the exception of the tile cache. 

The item downloads are not recreated automatically, but can be recreated by visiting the back of the card for the item or by calling the `downloadItem` function for the item. The initial response given by the server when an item is first requested (either by loading the back of card or clicking the download item button when the item is not yet available) will be a 202 (request received). Once the item is ready to download, the response will be 200.  

Bounding box updates are all currently built into the system and will update based on changes the user makes either via the mediation page or otherwise.


#Health/Status Checks and Item info

There are a number of services that can be checked that will provide insight on the Portal's items. These services can be visited on all endpoints/tiers.

<a href="https://marine.usgs.gov/coastalchangehazardsportal/data/health">Health Check service</a> provides a summary of application states that are used for application monitoring 

<a href="https://marine.usgs.gov/coastalchangehazardsportal/data/thumbnail/">Thumbnail service</a> provides a summary of thumbnail generation details, indicating whether or not the thumbnails need to be refreshed. A sweeper program is being created to use this information to automatically clean up on a regular basis.

<a href="https://marine.usgs.gov/coastalchangehazardsportal/data/download">Download item service</a> provides a status on item's and their download, the timestamp it was created and its location.

<a href="https://marine.usgs.gov/coastalchangehazardsportal/data/item/CARv9Z5">Item service</a> provides a document summarizing all of the item's details including bbox, children, publications and so forth. Replace the `itemID` at the end of the request based on desired item.

<a href="https://marine.usgs.gov/coastalchangehazardsportal/data/thubmnail/?dirty=true">Review which thumbnails are in need of refreshing. Working on an automatic function to reset these.</a>


Disclaimer
----------
This software is in the public domain because it contains materials that originally came from the U.S. Geological Survey, an agency of the United States Department of Interior. For more information, see the official USGS copyright policy at [http://www.usgs.gov/visual-id/credit_usgs.html#copyright](http://www.usgs.gov/visual-id/credit_usgs.html#copyright)

This information is preliminary or provisional and is subject to revision. It is being provided to meet the need for timely best science. The information has not received final approval by the U.S. Geological Survey (USGS) and is provided on the condition that neither the USGS nor the U.S. Government shall be held liable for any damages resulting from the authorized or unauthorized use of the information. Although this software program has been used by the USGS, no warranty, expressed or implied, is made by the USGS or the U.S. Government as to the accuracy and functioning of the program and related program material nor shall the fact of distribution constitute any such warranty, and no responsibility is assumed by the USGS in connection therewith.

This software is provided "AS IS."


 [
    ![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png)
  ](http://creativecommons.org/publicdomain/zero/1.0/)
