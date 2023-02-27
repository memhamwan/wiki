---
title: "Generating Coverage Map"
---

Based on prior works: https://fasma.org/modeling-repeaters-in-north-florida/ and https://github.com/kd7lxl/signalserver-scripts.

For this we'll be using a dockerized version of the signal-server. This is handy because the project doesn't distribute binaries and is a bit sporatically maintained, so this lets us skip any sort of build env setup steps. And docker is fun to play in, afterall. You can find the dockerfile in my fork of the [Signal-Server](https://github.com/turnrye/Signal-Server) project.

To explore the image, drop into a shell:

```
docker run -it --entrypoint /bin/bash signal-server
```

## Get your SRTM data setup

SRTM 3 second data is a good DEM dataset, but AW3D30 also provides great coverage and has better accuracy and precision (ref [10.3390/rs12213482](https://doi.org/10.3390/rs12213482)). So let's consider looking into this instead. There are more hoops to jump through though, and unless you're trying to model microwave systems and needing detailed maps, it's probably not worth the fuss. [Start here](https://wiki.openstreetmap.org/wiki/AW3D30) for instructions on how to download the data.

Once you have the data that you need, you'll need to convert the data to hgt format, and from there to sdf. Luckily Jaxa provided a script for the first part: https://www.eorc.jaxa.jp/ALOS/en/dataset/aw3d30/data/aw3d30_srtmhgt.zip

In the container, download the helper script. You'll also need to `apt update && apt install bc` because the script uses this calculator that is not preinstalled. 

Follow the readme which explains how to use the script with its `input` and `output` directories. Note that the script they include has bugs, so I fixed them. It appears that their datasets filenames changed but this script wasn't maintained. For reference, here are what my input filenames look like: `./input/ALPSMLC30_N031W092_DSM.tif` Here's what I ended up having to use for this to work:

```bash
#!/usr/bin/env bash
# Copyright 2019 Japan Aerospace Exploration Agency, 2023 Ryan Turner

INPUT_DIR=./input
OUTPUT_DIR=./output
vrtfile=./input.vrt

[ -d "$OUTPUT_DIR" ] || mkdir -p $OUTPUT_DIR || { echo "error: $OUTPUT_DIR " 1>&2; exit 1; }

gdalbuildvrt -overwrite -srcnodata -9999 -vrtnodata -9999 ${vrtfile} ${INPUT_DIR}/*_DSM.tif

res=$(echo 1/3600/2 |bc -l)
for aw3d30 in  "${INPUT_DIR}"/*_DSM.tif
do
   echo "Working on ${aw3d30}"
   [ -f "${aw3d30}" ] || continue
   srtm=$(echo "${aw3d30}" | awk -F / '{print substr($NF,11,1) substr($NF,13,6)".hgt"}')

   [ -f "${OUTPUT_DIR}/${srtm}" ] && { echo "skip ${srtm}" 1>&2; continue; }
   xmin=$(echo "${aw3d30}" | awk -F / 'substr($NF,15,1)=="E"{print substr($NF,21,3)*1} substr($NF,15,1)=="W"{print substr($NF,16,3)*(-1)}')
   ymin=$(echo "${aw3d30}" | awk -F / 'substr($NF,11,1)=="N"{print substr($NF,12,3)*1} substr($NF,11,1)=="S"{print substr($NF,12,3)*(-1)}')
   xmax=$(echo "${xmin}"+1 | bc)
   ymax=$(echo "${ymin}"+1 | bc)
   xmin=$(echo "${xmin}"-"${res}" | bc)
   ymin=$(echo "${ymin}"-"${res}" | bc)
   xmax=$(echo "${xmax}"+"${res}" | bc)
   ymax=$(echo "${ymax}"+"${res}" | bc)
   gdalwarp -te "${xmin}" "${ymin}" "${xmax}" "${ymax}" -ts 3601 3601 -r bilinear ${vrtfile} ${OUTPUT_DIR}/"${srtm}".tif
   gdal_translate -of SRTMHGT ${OUTPUT_DIR}/"${srtm}".tif ${OUTPUT_DIR}/"${srtm}"
   rm -f ${OUTPUT_DIR}/"${srtm}".tif

done

rm $vrtfile
```

Execute the commands within the osgeo/gdal image:

```
docker run -it --rm -v /Users/turnrye/signal-server/AW3D30:/AW3D30 -v /Users/turnrye/signal-server/AW3D30-sdf:/AW3D30-sdf --entrypoint /bin/bash osgeo/gdal
``` 

Once this is done, you should have a directory with a bunch of HGT files ready to go. So run the conversion from HGT to SDF now.

```
docker run -v /Users/turnrye/signal-server/AW3D30/aw3d30_srtmhgt/output:/srtm -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf --entrypoint /bin/bash signal-server -c 'cd /sdf && (for i in /srtm/*.hgt; do /signal-server/utils/sdf/usgs2sdf/srtm2sdf-hd $i; done)'
```

Remember that with your HD data, you'll need to modify the commands to use `signalserverHD` and `/Users/turnrye/signal-server/AW3D30-sdf`.

## Get your antenna data

Next, get the antenna data that you'll use for your map. If you're using a Ubiquiti antenna, you can find it [on this page](https://help.ui.com/hc/en-us/articles/204952114-airMAX-Antenna-Data). For this tutorial, I'll be using the AM-5G19-120. Download the `.ant` files.

Unzip the files to a convenient location, in my case `/Users/turnrye/signal-server/ant`. Then, just like before we need to convert the files. Let's use the `ant2azel` conversion script for this:

```
docker run -v /Users/turnrye/signal-server/ant:/ant --entrypoint python2 signal-server /signal-server/utils/antenna/ant2azel.py -i /ant/AMO-5G13-Hpol.ant
docker run -v /Users/turnrye/signal-server/ant:/ant --entrypoint python2 signal-server /signal-server/utils/antenna/ant2azel.py -i /ant/AMO-5G13-Vpol.ant
```

Our application expects the antenna for a given output file to be present in the same directory and named the same as the output file name (less the extension). So this means we now need to copy these files to our `/out` directory where we'll expect results to be.

Also, we actually are only going to be generating the coverage report for one polarity despite having dual polarity. This is because in all honesty, we don't expect this map to want to show coverages that could only be achieved with both polarities. This means that in some scenarios the map will be pessimistic, but that's for the better: it's almost always too generous anyway.

So, let's copy just the horizontal polarity data into our `out` directory with a simple `ant` name to shorten the next few commands.

```
cp /Users/turnrye/signal-server/ant/AM-5G19-120-Hpol.az /Users/turnrye/signal-server/out/ant.az
cp /Users/turnrye/signal-server/ant/AM-5G19-120-Hpol.el /Users/turnrye/signal-server/out/ant.el
```

## Create the coverage models

Now it's time to actually generate the coverage map. This assumes you want to run the coverage for one site with three sector antennas at (0,120,240) degrees true bearing. We'll call this "sco" and number the sectors 1, 2, and 3 accordingly.

First, copy the antenna files into place

```
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/sco1.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/sco1.el
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/sco2.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/sco2.el
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/sco3.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/sco3.el
```

Then, generate the coverages. Note these commands are where you specify key things like lat/lon, tx height, frequency, erp, rx height, receive threshold, and antenna bearing. To see what these options are, run `docker run -it --rm signal-server`. By the way, in my example here ERP is an insane 39kWatt, that is because of the following:

> 27 dBm + 19 dBi + 30 dBi = 39811 watts

Thanks Tom KD7LXL for piecing this together for me (and many others).

```
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.138667 -lon -90.020167 -txh 62.7888 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 0 -o /out/sco1
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.138667 -lon -90.020167 -txh 62.7888 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 120 -o /out/sco2
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.138667 -lon -90.020167 -txh 62.7888 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 240 -o /out/sco3
```

## Convert the output to a file you can use

The previous commands output an antiquated ppm file format which we can't really use. So, let's convert it to PNG.

```
convert /Users/turnrye/signal-server/out/sco1.ppm -transparent white /Users/turnrye/signal-server/out/sco1.png
convert /Users/turnrye/signal-server/out/sco2.ppm -transparent white /Users/turnrye/signal-server/out/sco2.png
convert /Users/turnrye/signal-server/out/sco3.ppm -transparent white /Users/turnrye/signal-server/out/sco3.png
```

The trouble is, PNGs dont have any geodata. We dont want to lose this, so let's add it to the file using the geotiff format. First, we have to refer back to our signalserver output from above. It tells us the bounds of the images. For instance, given this output from signalserver: `|36.004994|-88.768667|34.204996|-90.968667|`, we can juggle the numbers around and then use gdal_translate like so:
```
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -91.120167 36.038666 -88.920167 34.238668 /out/sco1.png /out/sco1.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -91.120167 36.038666 -88.920167 34.238668 /out/sco2.png /out/sco2.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -91.120167 36.038666 -88.920167 34.238668 /out/sco3.png /out/sco3.geotiff 
```

Out plops a geotiff that you can use in a GIS application. From here you could make tiles to publish you map to the web or do further GIS analysis.


## Full example

Let's try another location with a full example. In this run, I'll be using our real production data. A few adjustments have to be made but they're commented below.

| site | ground elevation per srtm | ground elevation per aw3d30 | real height | calculated height using aw3d30 |
| ---- | ------------------------- | --------------------------- | ----------- | -------------------------------|
| hil  | 91.94                     | 104                         | 100         | 100-(104-91.94) = 87.94        |
| mno  |  ? | 106 | ? | 131 - 106 = 25 |
| crw  | Real elevation: 168.554 | 163 | 169-163 = 6 |

Here's the entire block of commands:
```
# SCO
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -91.120167 36.038111 -88.920167 34.239223 /out/sco1.png /out/sco1-2.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -91.120167 36.038111 -88.920167 34.239223 /out/sco2.png /out/sco2-2.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -91.120167 36.038111 -88.920167 34.239223 /out/sco3.png /out/sco3-2.geotiff 


# HIL
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/hil1.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/hil1.el
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/hil2.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/hil2.el
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/hil3.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/hil3.el
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.105 -lon -89.868667 -txh 87.94 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 0 -o /out/hil1
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.105 -lon -89.868667 -txh 87.94 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 120 -o /out/hil2
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.105 -lon -89.868667 -txh 87.94 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 240 -o /out/hil3
convert /Users/turnrye/signal-server/out/hil1.ppm -transparent white /Users/turnrye/signal-server/out/hil1.png
convert /Users/turnrye/signal-server/out/hil2.ppm -transparent white /Users/turnrye/signal-server/out/hil2.png
convert /Users/turnrye/signal-server/out/hil3.ppm -transparent white /Users/turnrye/signal-server/out/hil3.png
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -90.968111 36.004444 -88.769223 34.205556 /out/hil1.png /out/hil1-2.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -90.968111 36.004444 -88.769223 34.205556 /out/hil2.png /out/hil2-2.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -90.968111 36.004444 -88.769223 34.205556 /out/hil3.png /out/hil3-2.geotiff

# MNO
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/mno1.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/mno1.el
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/mno2.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/mno2.el
cp /Users/turnrye/signal-server/out/ant.az /Users/turnrye/signal-server/out/mno3.az
cp /Users/turnrye/signal-server/out/ant.el /Users/turnrye/signal-server/out/mno3.el
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.234667 -lon -89.892 -txh 25 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 0 -o /out/mno1
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.234667 -lon -89.892 -txh 25 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 120 -o /out/mno2
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 35.234667 -lon -89.892 -txh 25 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 240 -o /out/mno3
convert /Users/turnrye/signal-server/out/mno1.ppm -transparent white /Users/turnrye/signal-server/out/mno1.png
convert /Users/turnrye/signal-server/out/mno2.ppm -transparent white /Users/turnrye/signal-server/out/mno2.png
convert /Users/turnrye/signal-server/out/mno3.ppm -transparent white /Users/turnrye/signal-server/out/mno3.png
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -90.993389 36.134111 -88.790611 34.335223 /out/mno1.png /out/mno1-2.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -90.993389 36.134111 -88.790611 34.335223 /out/mno2.png /out/mno2-2.geotiff 
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -90.993389 36.134111 -88.790611 34.335223 /out/mno3.png /out/mno3-2.geotiff

# CRW
cp /Users/turnrye/signal-server/ant/AMO-5G13-Vpol.az /Users/turnrye/signal-server/out/crw1.az
cp /Users/turnrye/signal-server/ant/AMO-5G13-Vpol.az /Users/turnrye/signal-server/out/crw1.el
docker run -it -v /Users/turnrye/signal-server/color:/color -v /Users/turnrye/signal-server/ant:/ant -v /Users/turnrye/signal-server/AW3D30-sdf:/sdf -v /Users/turnrye/signal-server/out:/out --entrypoint ./signalserverHD signal-server -sdf /sdf -m -dbm -pm 1 -dbg -lat 34.9745 -lon -89.866333 -txh 6 -f 5900 -erp 39811 -rxh 10 -rt -80 -R 100 -rot 0 -o /out/crw1
convert /Users/turnrye/signal-server/out/crw1.ppm -transparent white /Users/turnrye/signal-server/out/crw1.png
docker run -it -v /Users/turnrye/signal-server/out:/out --entrypoint gdal_translate osgeo/gdal -of GTiff -a_srs EPSG:4326 -a_ullr -90.964111 35.873944 -88.768555 34.075056 /out/crw1.png /out/crw1-2.geotiff

# LEB

# AZO
```


## Bonus points: generate xyztiles from QGis
```
{
  "area_units": "m2",
  "distance_units": "meters",
  "ellipsoid": "EPSG:7019",
  "inputs": {
    "BACKGROUND_COLOR": "rgba( 0, 0, 0, 0.00 )",
    "DPI": 98,
    "EXTENT": "-91.623616369,-88.389526398,34.549546689,35.700711754 [EPSG:4269]",
    "METATILESIZE": 4,
    "OUTPUT_DIRECTORY": "TEMPORARY_OUTPUT",
    "OUTPUT_HTML": "TEMPORARY_OUTPUT",
    "QUALITY": 75,
    "TILE_FORMAT": 0,
    "TILE_HEIGHT": 256,
    "TILE_WIDTH": 256,
    "TMS_CONVENTION": false,
    "ZOOM_MAX": 14,
    "ZOOM_MIN": 8
  }
}
```


_Below is an old blog post from the website. It serves as a reference that is 7 years out of date. Help wanted getting this back up-to-date._

Today I document how I generate my coverage maps, and I want to take my maps a step farther to calculate which amateurs fall in the coverage area. I’ll describe the complete process using a potential cell site “Crosstown”.

### Model the coverage

\
For RF modelling, I use [Radio Mobile](https://web.archive.org/web/20160313031456/http://www.cplus.org/rmw/english1.html). With measured performance data of the antennas and specified radio performance, I’m able to use accurate models values for our equipment. I use [SRTM](https://web.archive.org/web/20160313031456/http://www2.jpl.nasa.gov/srtm/) as the elevation data source.  I generate combined Cartesian radio coverage maps for all 3 sectors using these settings: web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/radio-mobile-coverage-files.zip.
_Quick stats: Crosstown’s antenna height is 57.302 m AGL, mobile node is 10 m AGL; systems freq range is 5845 – 5925 MHz; default surface refractivity, ground conductivity, and relative ground permittivity; the systems tx power is 1.3 watts, receiver threshold is 70.7946 uV; client nodes antenna gain is 29 dBi; sector’s antenna gain is 14 dBi._ 

Then I combined these images and converted them to grayscale to yield 1 raster for the site’s coverage area. The final output is pictured to the left.

### Get the FCC ULS data

I’ve also [exported the ULS database for active hams](https://data.fcc.gov/download/pub/uls/complete/l_amat.zip) with mailing addresses in Arkansas, Mississippi, and Tennessee. Using Bing! and a [short script](https://gist.github.com/turnrye/8996225) that I wrote in Python, I geolocated all of the addresses for the records, yielding a massive [csv file (actually it’s a TSV)](https://web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/tristate_amateurs_12_feb_14_with_geolocation.csv) with callsign, name, address, city, state, latitude, and longitude for every active licensed amateur operator in the tristate area.

### Get Quantum GIS setup with data

I created a project with [US county maps](https://apps.nationalmap.gov/downloader/) (specifically "USGS National Boundary Dataset (NBD) in \__\_ State or Territory (published 20220812)-GeoPackage" and USGS 1/3 Arc Second n35w090 20220729) added for reference, and then using the “Add Delimited Text Layer” plugin I imported the CSV files made previously (here’s the results: web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/tristate-amateurs-12feb-shapefile.zip). After adding transparency (85%) to the point symbols, it turned into a sort of “heat map”, as pictured to the right:

Next is to import the coverage map from Radio Mobile. Sadly, the auto-generated files do **not** import properly into QGIS, so instead what I did was convert the PNG file into a georeferenced PNG using gdal_translate and the following command (your lat/long of ll and ur should vary!):\
`gdal_translate -of PNG -a_srs EPSG:4326 \\\\\\\\\\\\\\\\\\\\\\\\ \\\\\\\\-a_ullr -90.59023 35.58994 -89.48978 34.69006 \\\\\\\\\\\\\\\\\\\\\\\\ Crosstown.png Crosstown-geo.png`

I imported the georeferenced PNG file into QGIS as a raster layer. In the Layer Properties I set render to “single band gray” with Gray band being “Band 2”, and I enabled “Invert color map”. I added the transparency pixels to make the white areas transparent. Finally, I added some global transparency to make it easier to see. With that, I got the output pictured to the right.

### Analyze the raster, join the vectors

Now I used the “point sampling tool” plugin to find all of the amateurs that fell within the coverage area. This created a point vector layer (download here: web.archive.org/web/20160313031456/http://memhamwan.org/wp-content/uploads/2014/02/crosstown-coverage-points-feb-12.zip) that contained the existing points with a new value: the color that exists at that point. With this data, we get a count of the number of active amateurs who have mailing addresses as geolocated by Bing! that fall within our calculated coverage map. For bonus points, I’d like to find out who is in coverage and who is not (right now its just a list of points and coverage map color; that doesn’t give me opportunity for spot checking). Since the point sampling tool created a separate layer, we have to use the vector tool “Join attributes by location”.

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