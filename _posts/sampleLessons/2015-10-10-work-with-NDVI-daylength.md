---
layout: post
title:  "Working with NEON (like) Remote Sensing Time Series Data"
date:   2014-07-21 20:49:52
output: html_document
permalink: /R/ndvi/
---


    #work with rasters
    library(raster)
    #best for importing shapefiles
    library(rgdal) 
    #plotting
    library(ggplot2)
    
    options(stringsAsFactors = FALSE)
    
    #set the working directory
    setwd("~/Documents/data/1_DataPortal_Workshop")


#First plot the basemap to see where the tower boundary is. 

Note, this will require opening up the geotiff imagery for the site.
Then plotting 

CRS: UTM ZONE 18N FOR HARVARD


    #import imagery
    baseImage <- stack("AOP/14060113_EH021656(20140601155722)-0263_ort.tif")
    
    #plot the image for the site
    plotRGB(baseImage,r=1,g=2,b=3, 
            main="Harvard Tower Site")
    
    #add the shapefile 
    #note: read ogr is preferred as it maintains prj info
    squarePlot <- readOGR("Landsat_TimeSeries/","HarClip_UTMZ18")

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "Landsat_TimeSeries/", layer: "HarClip_UTMZ18"
    ## with 1 features
    ## It has 1 fields

    #add the plot boundary to the image.
    plot(squarePlot, col="yellow", add=TRUE)

![ ]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/view-basemap-1.png) 


#Crop the Base Image to the AOI
This is the area used to clip the NDVI data from. We can expand it a bit.
Concerned with size but i think a bit larger could be ok.


    #crop the image to the plot boundary?
    new <- crop(baseImage,squarePlot)
    
    plotRGB(new, axes=F,main="RGB image cropped")

![ ]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/crop-image-1.png) 

    #not sure how to add a title to plotRGB??
    
    #export geotiff
    #write the geotiff - change overwrite=TRUE to overwrite=FALSE if you want to 
    #make sure you don't overwrite your files!
    writeRaster(new,"new","GTiff", overwrite=TRUE)

#Step 2

Import the NDVI time series. 

##NOTE: we could crop these to a larger boundary. right now they are about 1.5mb 
##a tile...


    #define the path to write tiffs
    tifPath <- "Landsat_TimeSeries/D01/LS5/P12R31/2011/"
    
    #open up the cropped files
    #create list of files to make raster stack
    allCropped <-  list.files(tifPath, full.names=TRUE)
    
    #create a raster stack from the list
    rastStack <- stack(allCropped)
    
    
    
    #layout(matrix(c(1,1,2,3), 2, 2, byrow = TRUE))
    #would like to figure out how to plot these with 2-3 in each row rather than 4
    plot(rastStack, zlim=c(1500,10000),nc=3)

![ ]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/process-NDVI-images-1.png) 

    #adjust the layout
    par(mfrow=c(7,2))
    #plot histograms for each image
    hist(rastStack,xlim=c(1500,10000))

    ## Error in plot.new(): figure margins too large

    #create data frame, calculate NDVI
    ndvi.df <- as.data.frame(matrix(-999, ncol = 2, nrow = length(allCropped)))
    colnames(ndvi.df) <- c("julianDays", "meanNDVI")
    i <- 0
    for (crop in allCropped){
      i=i+1
      #open raster
      imageCrop <- raster(crop)
      
      #calculate the mean of each
      ndvi.df$meanNDVI[i] <- cellStats(imageCrop,mean) 
      
      #grab julian days
      ndvi.df$julianDays[i] <- substr(crop,nchar(crop)-16,nchar(crop)-14)
    }

#PLOT NDVI for 2011

NOTE: will add tiles for 2009-2010 - have 30 years but that is a lot!
#should we expand the extent or not?
LOOK UP:
#how to connect the dots in the NDVI ggplot??

Now, let's plot NDVI for one year


    ##plot stuff
    #need to figure out the best plotting method to connect the dots! Or a better input format object
    
    ggplot(ndvi.df, aes(julianDays, meanNDVI)) +
      geom_point(size=4,colour = "blue") +
      ggtitle("NDVI for 2011\nLandsat Derived") +
      xlab("Julian Days") + ylab("Mean NDVI") +
      theme(text = element_text(size=20))

![ ]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/plot-mean-NDVI-1.png) 



    #create animation ot the NDVI outputs
    library(animation)
    
    #if(!file.exists("ndvi.gif")) { # Check if the file exists
      saveGIF(
        for (i in 1:length(allCropped)) {
                          plot(rastStack[[i]],
                          main=names(rastStack[[i]]),
                          legend.lab="NDVI",
                          col=rev(terrain.colors(30)),
                          zlim=c(1500,10000) )
          }, 
        movie.name = "ndvi.gif", 
        ani.width = 300, ani.height = 300, 
        interval=.5)

    ## Executing: 
    ## 'convert' -loop 0 -delay 50 Rplot1.png Rplot2.png Rplot3.png
    ##     Rplot4.png Rplot5.png Rplot6.png Rplot7.png Rplot8.png
    ##     Rplot9.png Rplot10.png Rplot11.png Rplot12.png Rplot13.png
    ##     'ndvi.gif'
    ## Output at: ndvi.gif

    ## [1] TRUE

    #}

![ ]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/create-animation-1.png) 

##The animated gif!

![NDVI time series animation]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/ndvi.gif)

Time series for NDVI for 2009 at Harvard Forest

#Cropping Rasters Using Shapefiles


    #load the imagery here. Then potentially crop it. Use as a base map
      
    #load xy point for tower location - overlay on to imagery base map
      
    #load the CHM here. then crop it and get average tree height
      
    #load DTM hill shade -- use that as a base map??


#Look at Day Length Data for Harvard

NOTE - i need to get the data from 2009-2011 to align with the Landsat Time Series


    #load the lubridate package to work with time
    library(lubridate)
    #readr is ideal to open fixed width files (faster than utils)
    library(readr)
    
    #read in fixed width file  
    dayLengthHar2011 <- read.fwf(
      file="precip_Daylength/Petersham_Mass_2011.txt",
      widths=c(8,9,9,9,9,9,9,9,9,9,9,9,9))
     
    names(dayLengthHar2011) <- c("Day","Jan","Feb","Mar","Apr",
                                 "May","June","July","Aug","Sept",
                                 "Oct","Nov","Dec") 
    #open file  
    #dayLengthHar2015 <- read.csv(file = "precip_Daylength/Petersham_Mass_2009.txt", stringsAsFactors = FALSE)
    
    #just pull out the columns with time information
    tempDF <- dayLengthHar2011[,2:13]
    tempDF[] <- lapply(tempDF, function(x){hm(x)$hour + hm(x)$minute/60})
    #populate original DF with the new time data in decimal hours 
    dayLengthHar2011[,2:13] <- tempDF
    
    #plot One MOnth of  data
    ggplot(dayLengthHar2011, aes(Day, Jan)) +
      geom_point()+
      ggtitle("Day Length\nJan 2009") +
      xlab("Day of Month") + ylab("Day Length (Hours)") +
      theme(text = element_text(size=20))

![ ]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/Clean-Up-Day-Length-1.png) 

#Convert to Julian Days and Plot

Next, plot full year's worth of daylength for 2011.
Note: this could be turned into a function to do multiple files.


    #convert to julian days
    
    #stack the data frame
    dayLengthHar2011.st <- stack(dayLengthHar2011[2:13])
    #remove NA values
    dayLengthHar2011.st <- dayLengthHar2011.st[complete.cases(dayLengthHar2011.st),]
    #add julian days (count)
    dayLengthHar2011.st$JulianDay<-seq.int(nrow(dayLengthHar2011.st))
    #fix names
    names(dayLengthHar2011.st) <- c("Hours","Month","JulianDay")
    
    #plot Years Worth of  data
    ggplot(dayLengthHar2011.st, aes(JulianDay,Hours)) +
      geom_point()+
      ggtitle("Day Length\nJan 2011") +
      xlab("Julian Days") + ylab("Day Length (Hours)") +
      theme(text = element_text(size=20))

![ ]({{ site.baseurl }}/images/rfigs/2015-10-10-work-with-NDVI-daylength/plot-daylength-1.png) 

