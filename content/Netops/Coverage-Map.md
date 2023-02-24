_Below is an old blog post from the website. It serves as a reference that is 7 years out of date. Help wanted getting this back up-to-date._

Today I document how I generate my coverage maps, and I want to take my maps a step farther to calculate which amateurs fall in the coverage area. I’ll describe the complete process using a potential cell site “Crosstown”.

### Model the coverage

\
For RF modelling, I use [Radio Mobile](https://web.archive.org/web/20160313031456/http://www.cplus.org/rmw/english1.html). With measured performance data of the antennas and specified radio performance, I’m able to use accurate models values for our equipment. I use [SRTM](https://web.archive.org/web/20160313031456/http://www2.jpl.nasa.gov/srtm/) as the elevation data source.  I generate combined Cartesian radio coverage maps for all 3 sectors using these settings: [radio-mobile-coverage-files](https://web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/radio-mobile-coverage-files.zip).\
_Quick stats: Crosstown’s antenna height is 57.302 m AGL, mobile node is 10 m AGL; systems freq range is 5845 – 5925 MHz; default surface refractivity, ground conductivity, and relative ground permittivity; the systems tx power is 1.3 watts, receiver threshold is 70.7946 uV; client nodes antenna gain is 29 dBi; sector’s antenna gain is 14 dBi._ 

Then I combined these images and converted them to grayscale to yield 1 raster for the site’s coverage area. The final output is pictured to the left.

### Get the FCC ULS data

I’ve also [exported the ULS database for active hams](https://data.fcc.gov/download/pub/uls/complete/l_amat.zip) with mailing addresses in Arkansas, Mississippi, and Tennessee. Using Bing! and a [short script](https://gist.github.com/turnrye/8996225) that I wrote in Python, I geolocated all of the addresses for the records, yielding a massive [csv file (actually it’s a TSV)](https://web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/tristate_amateurs_12_feb_14_with_geolocation.csv) with callsign, name, address, city, state, latitude, and longitude for every active licensed amateur operator in the tristate area.

### Get Quantum GIS setup with data

I created a project with [US county maps](https://apps.nationalmap.gov/downloader/) (specifically "USGS National Boundary Dataset (NBD) in \__\_ State or Territory (published 20220812)-GeoPackage" and USGS 1/3 Arc Second n35w090 20220729) added for reference, and then using the “Add Delimited Text Layer” plugin I imported the CSV files made previously (here’s the results: [tristate-amateurs-12feb-shapefile](https://web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/tristate-amateurs-12feb-shapefile.zip)). After adding transparency (85%) to the point symbols, it turned into a sort of “heat map”, as pictured to the right:

Next is to import the coverage map from Radio Mobile. Sadly, the auto-generated files do **not** import properly into QGIS, so instead what I did was convert the PNG file into a georeferenced PNG using gdal_translate and the following command (your lat/long of ll and ur should vary!):\
`gdal_translate -of PNG -a_srs EPSG:4326 \\\\\\\\\\\\\\\\\\\\\\\\ \\\\\\\\-a_ullr -90.59023 35.58994 -89.48978 34.69006 \\\\\\\\\\\\\\\\\\\\\\\\ Crosstown.png Crosstown-geo.png`

I imported the georeferenced PNG file into QGIS as a raster layer. In the Layer Properties I set render to “single band gray” with Gray band being “Band 2”, and I enabled “Invert color map”. I added the transparency pixels to make the white areas transparent. Finally, I added some global transparency to make it easier to see. With that, I got the output pictured to the right.

### Analyze the raster, join the vectors

Now I used the “point sampling tool” plugin to find all of the amateurs that fell within the coverage area. This created a point vector layer (download here: [crosstown-coverage-points-feb-12](https://web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/crosstown-coverage-points-feb-12.zip)) that contained the existing points with a new value: the color that exists at that point. With this data, we get a count of the number of active amateurs who have mailing addresses as geolocated by Bing! that fall within our calculated coverage map. For bonus points, I’d like to find out who is in coverage and who is not (right now its just a list of points and coverage map color; that doesn’t give me opportunity for spot checking). Since the point sampling tool created a separate layer, we have to use the vector tool “Join attributes by location”.

### Out comes data

With the [result](https://web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/amateurs-with-coverage-12-feb.csv), we’re able to finally see all of the data together. In my case, since the coverage map’s color was inverted, a value of “255” meant that the point is in the coverage area. With that, we can conclude that **1,344 active licensed amateurs have their mailing addresses within the coverage area of our first potential site**.

With this new data, we could potentially make a web interface for a user  to query a callsign and return what sites coverage areas they are in. There is more work to be done! I’ve attached all of the source files I could think of for this work. Be sure to let me know if I missed one or if you have a question about this by leaving a comment below!

## New notes (WIP 2022-09)

Files to grab for qgis:
- https://prd-tnm.s3.amazonaws.com/StagedProducts/GovtUnit/GPKG/GOVTUNIT_Arkansas_State_GPKG.zip
- https://prd-tnm.s3.amazonaws.com/StagedProducts/GovtUnit/GPKG/GOVTUNIT_Mississippi_State_GPKG.zip
- https://prd-tnm.s3.amazonaws.com/StagedProducts/GovtUnit/GPKG/GOVTUNIT_Tennessee_State_GPKG.zip
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n34w090/USGS_13_n34w090_20220601.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n34w091/USGS_13_n34w091_20180705.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n35w090/USGS_13_n35w090_20220729.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n35w091/USGS_13_n35w091_20190718.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n35w092/USGS_13_n35w092_20180814.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n36w089/USGS_13_n36w089_20220729.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n36w090/USGS_13_n36w090_20220729.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n36w091/USGS_13_n36w091_20220727.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n36w092/USGS_13_n36w092_20180508.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n37w089/USGS_13_n37w089_20220727.tif
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/TIFF/historical/n37w090/USGS_13_n37w090_20220727.tif
- https://geonames.usgs.gov/docs/stategaz/MS_Features.zip
- https://geonames.usgs.gov/docs/stategaz/TN_Features.zip
- https://geonames.usgs.gov/docs/stategaz/AR_Features.zip
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Struct/GPKG/STRUCT_Arkansas_State_GPKG.zip
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Struct/GPKG/STRUCT_Mississippi_State_GPKG.zip
- https://prd-tnm.s3.amazonaws.com/StagedProducts/Struct/GPKG/STRUCT_Tennessee_State_GPKG.zip
- https://data.fcc.gov/download/pub/uls/complete/r_tower.zip