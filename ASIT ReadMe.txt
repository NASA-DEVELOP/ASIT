ReadMe for Automate Snow Index Tool : ASIT
--------------------------------------------------------------------------------
Authors: Charles Barrow, Saranee Dutta, Elaina Gonsoroski, Tyler Lynn
Language Used: JavaScript
Software Used: Google Earth Engine (GEE)
Contact Info: charles.barrow.iii@gmail.com
Websites Needed: https://earthengine.google.com
                 https://code.earthengine.google.com
--------------------------------------------------------------------------------
Introduction: This script was written for the Google Earth Engine in
order to automate the process of gathering imagery and analyzing regions
to locate snow and determine how much is present. It takes a small number
of inputs so that the process is as user friendly as possible while 
remaining flexible enough to be used in other regions.
--------------------------------------------------------------------------------
A regular gmail account is currently needed in order to access GEE. You must
visit https://earthengine.google.com and click the SIGN UP button in the upper
right hand corner. There are some rules for use, and they will have to wait
until they send you directions on logging in. After you have access you can
visit https://code.earthengine.google.com in order to run this script by copy
and pasting it into the code editor. 

------------Brief Instructions--------------------------------------------------
1-Using the Draw Shapes tool, draw a rough square around the desired study area.

2-Go to line 22 and paste the name of the satellite you wish you use. Options are
listed in line 21, just highlight, copy and paste.

3-Go to line 25 and paste the name of the sensor chosen from the options on
line 24. Sentinel begins with 'S' and Landsats begine with 'L'.

4-Go to line 32 and input a start date for the study period using yyyy-mm-dd.

5-Go to line 33 and input an end date for the study period using yyyy-mm-dd.
If a single day is desired, start date is the desired date, and end date is
the following day.

6-Go to line 36 and input coordinates desired for the center of the map. Tucson
is the default setting. Be aware that Longitude comes first and latitude second.
The coordinates use - to designate South and West.

7-Go to line 43 and type the name of the desired watershed exactly as it
appears in the Fusion Table. Capitalization and spaces are important. Must be
enclosed within single quotes 'Example Creek'.

8-Go to line 46 and set the minimum elevation you expect to find snow, in meters.

9-Go to lines 62-73 and ensure the resulting map parameters are acceptable

----Note-----
You must ensure that lines 22 and 25 are in agreement. If using a particular
satellite, the option for the same system must be in both variables.
DO NOT MODIFY LINES 11-16!

--------------------------------------------------------------------------------
------------Line by line instructions-------------------------------------------
--------------------------------------------------------------------------------
This guide will start at the top of the script and explain the process to 
use it. Numbers will be the same as the line of code they are referring to.
--------------------------------------------------------------------------------
0: Above line 1 there in a section showing imports. This is where the 
geometry input is stored. This must be created within GEE using the 
'Draw Shape' tool in the map portion of the screen. The shape you create
should be a little larger than what you intend to analyze, any shape will 
work, however simple squares and rectangles are probably best. Once you have
created this, it should automatically name itself 'geometry'. Leave this
name alone.
--------------------------------------------------------------------------------
4: Import the USGS National Elevation Dataset as a variable 'elevation'

5: Import a Fusion Table for the watershed shapes in the Madrean Sky
Islands region.

--------------------------------------------------------------------------------
11: Array showing the standard names for the sensor bands to be used in ASIT.
13: Array showing the bands for Sentinel 2a and 2b which correspond to line 11.
14: Array showing the bands for Landsat 8 which correspond to line 11.
15: Array showing the bands for Landsat 7 which correspond to line 11.
16: Array showing the bands for Landsat 5 which correspond to line 11.

--------------------------------------------------------------------------------
21: Comment showing which sensors can be used in this script.

22:Variable that must be changed to match the sensor being used. 4 Possible 
sensors will work with this and options can be copied from line 21.

24: Comment showing which sensor bands can be used in this script.

25: Variable that must be changed to match the sensor bands correctly. The code
uses the names shown in line 11, but sets the variables later, this line
allows the functions to remain unchanged while allowing the input to change.

--------------------------------------------------------------------------------
32: Variable which sets the start date for the time period to be analyzed.

33: Variable which sets the end date for the time period to be analyzed.

36: Informs GEE where to center the map. Choose coordinates if you wish to move
from the default location. Longitude first, latitude second. Positive numbers
correspond to North and East coordinates, while negative numbers correspond to
South and West coordinates.

40: Variable takes the geometry created at the beginning and filters the Fusion
Tables. This is done so that large files of polygons can be cut down to only
the particular area of interest.

43: Variable which determines which watershed the calculations will run on.
Set this variable to match the name exactly as it appears in the Fusion Table.

46: Variable which sets the minimum elevation in meters.

49: Variable which filters the watersheds to only include the area input in
line 43. Optionally, ;// can be inserted after 'studyArea' and before
'.filter' in order to run the scrip on the entire region using only the 
geometry as a filter. 

----WARNING-----Removing the filter in line 49 to analyze the entire region
has a decent chance of crashing your browser. Oftentimes it will exceed the
memory limit. Recommend sticking to a single watershed at a time.

--------------------------------------------------------------------------------
54: Sets the variable 'blue' to match the first item in the array assigned in
line 24.

55: Sets the variable 'green' to match the first item in the array assigned in
line 24.

56: Sets the variable 'red' to match the first item in the array assigned in
line 24.

57: Sets the variable 'nir' to match the first item in the array assigned in
line 24.

58: Sets the variable 'swir1' to match the first item in the array assigned in
line 24.

59: Sets the variable 'swir2' to match the first item in the array assigned in
line 24.

62: Creates a set of parameters for the top map layer, defaults to snow index

69: Creates a set of parameters for the bottome map layer. The 'max' here 
will need to be changed to 1 for Landsat or 10000 for Sentinel.

82: sets the variable 'satellite' to match the sensor input from line 22 and
imports the image collection for that satellite from GEE servers.

85: Filters the image collection to only include scenes which touch the
study area that was set in line 40.

--------------------------------------------------------------------------------
92: Variable that uses the start and end dates from lines 32 and 33 to further
filter the image collection to only images that fit within the time frame.

96: Sets a more generic variable name that will be used to save calculations.

--------------------------------------------------------------------------------
101: Function which adds elevation from the imported DEM data as a band to each
image in the new withBands collection established in line 82.

--------------------------------------------------------------------------------
112: Cloud mapping formula when Landsat satellites are being used. 

124:Saito S3 formula when Landsat satellites are being used. 

--------------------------------------------------------------------------------
136: Cloud mapping formula when Sentinel satellites are being used.

147: Saito S3 formula when Sentinel satellites are being used. 

--------------------------------------------------------------------------------
162: Checks sensorBands variable for Sentinel input, if true, runs Sentinel
cloud detection formula, if false, runs Landsat version.

163: Checks sensorBands variable for Sentinel input, if true, runs Sentinel
Saito S3 formula, if false, runs Landsat version.

--------------------------------------------------------------------------------
169: First line of function which calculates the NDSI of the image. No changes
are needed here, regardless of sensor being used.

--------------------------------------------------------------------------------
181: First line of function to combine the NDSI and Saito S3 indexes. No changes
are needed here, regardless of sensor being used. CSI Index is result.

--------------------------------------------------------------------------------
194: Creates a mosaic of the last image to be loaded (most recent by default)
and calculates the square meters of snow pixels present in the watershed.

206: Prints the results of the area calculation.

--------------------------------------------------------------------------------
215: First line of the function that generates the time series chart. This does
not need to be changed unless particular format changes are desired. The title
is taken from the variable input in line 43. Times are taken from the metadata
of each image and are translated from UNIX time. Does not use the previously
created mosaic image.

----Note----
If you mosaic an image collection, this chart will not function because the
metadata is lost during mosaicking.

229: Prints the chart from line 215 into the Console window on the right hand
side of the screen. This can be maximized using the icon in the upper right
hand corner of the chart, and from there, exported as a .csv for use in
excel or another program.

--------------------------------------------------------------------------------
236: Adds a base map using the bottomLayer parameters set in lines 69-74.

237: Adds a map layer using the topLayer parameters from lines 62-67. 
This will display the calculated areas where snow is determined to be present.