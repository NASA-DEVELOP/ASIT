var geometry = /* color: #d63000 */ee.Geometry.Polygon(
        [[[-110.82733154296875, 32.646313145909126],
          [-111.03057861328125, 32.33820027152775],
          [-110.58837890625, 31.93351676190369],
          [-110.24505615234375, 32.15236189465576],
          [-110.7861328125, 32.685619853722]]]);


/**********************************************************************************************
********************************Import Data****************************************************
**********************************************************************************************/
var elevation = ee.Image('USGS/NED');
var skyIslands = ee.FeatureCollection('ft:1QF55Lg3fS3RInJ047KyN_PXqbbty2cHDJRAI3Icy');

/**********************************************************************************************
********************************Global Variables***********************************************
**********************************************************************************************/

var STD_Names = ['blue', 'green', 'red', 'nir', 'swir1', 'swir2'];  /* Do NOT Edit */

var S2_Bands =  ['B2',   'B3',    'B4',   'B8A',  'B11',   'B12'];  /* Do NOT Edit */
var LC8_Bands = ['B2',   'B3',    'B4',   'B5',   'B6',    'B7'];   /* Do NOT Edit */
var LE7_Bands = ['B1',   'B2',    'B3',   'B4',   'B5',    'B7'];   /* Do NOT Edit */
var LT5_Bands = ['B1',   'B2',    'B3',   'B4',   'B5',    'B7'];   /* Do NOT Edit */


/*  Change the following two variables to match the satellite that you are using. */

/* 'COPERNICUS/S2' :: 'LANDSAT/LC8_L1T_TOA_FMASK' :: 'LANDSAT/LE7_L1T_TOA_FMASK' :: 'LANDSAT/LT5_L1T_TOA_FMASK' */
var sensor = 'COPERNICUS/S2';

/*  S2_Bands  ::  LC8_Bands  ::  LE7_Bands  ::  LT5_Bands  */
var sensorBands = S2_Bands;


/*************************************Temporal Boundaries***********************************************/
/*Choose the beginning and end dates to be analyzed. No longer than 1 season. 
If analyzing a single day, choose the day of the image as the startDate, and the following day as endDate. */

var startDate = '2016-01-11';         /* 'yyyy-mm-dd' */
var endDate = '2016-01-12';

/* Set coordinates for the center of the  map */
Map.setCenter(-110.55, 32.30, 10);    /* (Longitude, Latitude, Zoom) + = N/E - = S/W  */

/***************************************Study Area Geomety***********************************************/
/* Select a study area by drawing a shape around your desired study area.  */
var studyArea = skyIslands.filterBounds(geometry);

/* Type name of desired watershed exactly as it appears in the Fusion Table */
var areaOfInterest = 'Upper Rincon Creek';

/* Set the minimum elevation to run the script on, in meters.  */
var elevBounds = 1500;

/* Filter feature class down to one watershed. */
var watershed = studyArea.filter(ee.Filter.eq('name', areaOfInterest)); 

/***************************************Assign Input Bands STD Names***********************************************/

/* DO NOT EDIT */
var blue = sensorBands[0];
var green = sensorBands[1];
var red = sensorBands[2];
var nir = sensorBands[3];
var swir1 = sensorBands[4];
var swir2 = sensorBands[5];

/***************************************Display Map Parameters***********************************************/
var topLayer = {
 /* Assign 'CSI' to whichever color band you prefer it to be displayed as  */ 
  bands: [red, green, 'CSI'],
  /*  Opacity of 0.0 is fully transparent and 1.0 is fully opaque.  */
  opacity: 0.5
};

var bottomLayer = {
  bands: [red, green, blue],
  min: 0,
/*  Set max to 1 for Landsat, and 10000 for Sentinel */
  max: 10000
};


/**************************************************************************************************************/
/********STOP********ONLY CHANGE BELOW THIS LINE IF YOU ARE CERTAIN OF WHAT YOU WANT TO DO*********STOP********/
/**************************************************************************************************************/

/* This imports the imagery collection based on the variable 'sensor' input from earlier  */
var satellite = ee.ImageCollection(sensor);

/* Boundary used to eliminate images from the collection not included in the variable 'studyArea' */
var spatialFiltered = satellite.filterBounds(studyArea);

/***********************************************************************************************
********************************Organize Data***************************************************
***********************************************************************************************/

/* Reduce image collection to include only images within the bounds of the study period */
var targetDate = spatialFiltered.filterDate(startDate, endDate);


/* Change variable name for uniformity with following code */
var withBands = targetDate;

/**********************************************************************************************
**********************************Adding Elevation Data****************************************
**********************************************************************************************/
var addElev = function(image) {
  return image.addBands(elevation.select('elevation'));
};
var withBands = withBands.map(addElev);



/**********************************************************************************************
                                    Cloud Mapping Landsat Variant
***********************************************************************************************/

/* Cloud Mapping: Blue for identifying clouds, SWIR1 for differentiating clouds from snow */
var addClouds = function(image) {
  var cloudy = (image.select(swir1).gte(0.12)).and(image.select(blue).gte(0.2))
  .rename('Cloud_Index');
  return image.addBands(cloudy);
};


/**********************************************************************************************
                                Saito S3 Index for all Landsat sensors
***********************************************************************************************/

/* Saito 1999 S3 Index */
var addS3index = function(image) {
  var snow3 = (((image.select(nir)).multiply((image.select(red)).subtract(image.select(swir1))))
  .divide((((image.select(nir))).add(image.select(red))).multiply((image.select(nir)).add(image.select(swir1)))))
  .rename('S3index');
  return image.addBands(snow3);
};

/**********************************************************************************************
                                    Cloud Mapping Sentinel Variant
***********************************************************************************************/

/* Cloud Mapping: Blue for identifying clouds, SWIR1 for differentiating clouds from snow */
var addSClouds = function(image) {
  var cloudy = ((image.select(swir1).divide(10000)).gte(0.12)).and((image.select(blue).divide(10000)).gte(0.2))
  .rename('Cloud_Index');
  return image.addBands(cloudy);
};


/**********************************************************************************************
                                Saito S3 Index for Sentinel sensors
***********************************************************************************************/
/* Saito 1999 S3 Index Sentinel Variant */
var addSentS3 = function(image) {
  var snow3 = ((((image.select(nir)).divide(10000))
  .multiply(((image.select(red)).divide(10000)).subtract((image.select(swir1)).divide(10000))))
  .divide((((image.select(nir)).divide(10000)).add((image.select(red)).divide(10000)))
  .multiply(((image.select(nir)).divide(10000)).add((image.select(swir1)).divide(10000)))))
  .rename('S3index');
  return image.addBands(snow3);
};

/**********************************************************************************************
                                  Calling the appropriate Formulae
***********************************************************************************************/
/* These two lines check for Sentinel input and run those formulae if true, if false it defaults
  to the Landsat formulae.*/
var withBands = (sensorBands == S2_Bands) ? withBands.map(addSClouds) : withBands.map(addClouds) ;
var withBands = (sensorBands == S2_Bands) ? withBands.map(addSentS3) : withBands.map(addS3index) ;

/**********************************************************************************************
                                    NDSI
***********************************************************************************************/

var addNDSI = function(image) {
  var ndsi = image.normalizedDifference([green, swir1]).rename('NDSI');
  return image.addBands(ndsi);
};
var withBands = withBands.map(addNDSI); 

/**********************************************************************************************
                                Combined Snow Index
***********************************************************************************************/

/* If both assign snow to a pixel (and Cloud_Index does not indicate clouds) then assign snow */

var addCSI = function(image) {
  var combSnowIndex = (image.select('NDSI').gte(0.05).and(image.select('S3index').gte(0.01))
  .and(image.select('Cloud_Index').eq(0))
  .and(image.select('elevation').gte(elevBounds))
  ).rename('CSI');
  return image.addBands(combSnowIndex);
};
var withBands = withBands.map(addCSI);

/*************************************************************************************************
*******************************Computing Area of Snow Pixels**************************************
*************************************************************************************************/
/*Combine the withBands variable into a single mosaic so it can be clipped*/
var mosaic = withBands.mosaic();

var pixelPicker = mosaic.select(['CSI']);
var areaImage = pixelPicker.multiply(ee.Image.pixelArea());

var snowSurfaceArea = areaImage.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: watershed,
  scale: 30,
  maxPixels: 1e9
  });

print('Snow Area: ', snowSurfaceArea.get('CSI'), 'square meters');

/*************************************************************************************************
*******************************Creating Time Series Chart*****************************************
*************************************************************************************************/

/*  Takes 7 inputs: 1. Image Collection. 2. Physical Boundary regions. 3. Command to calculate mean.
4. Band on which calculations are run. 5. Scale. 6. UNIX Time for when image was taken. 7. Label */

var snowTimeSeries = ui.Chart.image.seriesByRegion(
  withBands, watershed, ee.Reducer.mean(), 'CSI', 30, 'system:time_start', 'label')
  .setChartType('ScatterChart')
  .setOptions({
    title: areaOfInterest,
    vAxis: {title: 'Combined Snow Index'},
    lineWidth: 1,
    pointSize: 4,
    series: {
      0: {color: 'red'},
    }
  });

//Display
print(snowTimeSeries);

/*************************************************************************************************
*******************************Adding layers to the map*******************************************
*************************************************************************************************/

Map.addLayer(withBands, bottomLayer);
Map.addLayer(withBands, topLayer);