Analysis of GPS Variance in the Context of DMSM
================================

**Principal Author**  
Dr. John R. Withrow Jr.  
Cherokee Nation Technologies  
Contractor for USDA-FS FHTET  
NRRC Building A Ste 331  
2150 Centre Avenue  
Fort Collins, CO 80526  
johnwithrow@fs.fed.us  



### Executive Summary (Version Dated 20160622)
The most representative datasets from each observer have been determined based upon inspection of files from Nathan. They are organized in the following chart:

|           | Jeff           | Frank                           | Nathan                          |
|-----------|:--------------:|:-------------------------------:|:-------------------------------:|
|  Pre-DMSM | GPGGA_drew.csv | GPGGA_jlp.csv                   | Analysis.xlsx, Worksheet 1      |
| Post-DMSM |      -         | offlinedata.gdb/LOCATION_POINTS | offlinedata.gdb/LOCATION_POINTS |

For both Frank and Nathan signficant discrepancies between the Pre-DMSM and Post-DMSM information are visible. More investigation into this is warranted.

Investigation on differences with Nathan's unit due to the location of a nearby bluetooth GPS is on hold.  I am not sure how to discern the observations with the GPS device in the seat near the plane wing from the observations where the GPS device was near the front of the plane.  The shape file called BTWING.shp appears to contain not half the data, but all the data.

The best Pre-DMSM datasets from Jeff and Frank appear fragmented enough as to warrant a check to see if these are indeed the best sources of information.

For both Frank and Nathan, more observations appear in the Post-DMSM datasets than in the Pre-DMSM datasets, implying an interpolation process by the DMSM software described by Conly.

***

### Analytical Context

Some users of the DMSM interface have reported instances where the GPS location on the unit appears to lag behind the actual apparent location of the plane.  Four alternate hypotheses exist as explanations for this:  
1. This occurs when the GPS device is not receiving any location information,  
2. This occurs when the GPS device is receiving unusually erroneous information,   
3. This occurs as a result of a constant positional averaging process onboard the tablet but independent of the DMSM software, and  
4. This occurs as a result of a constant positional averaging process in the DMSM software.

The goal of the present analysis is to investigate the incoming raw GPS information onboard each of three tablets showing simultaneous recordings of the same 10 flight events on 09 Oct 2015.  More specifically, the variability in location and positional accuracy will be investigated in light of the above four hypotheses.

One of the three laptops was connected via bluetooth to an external GPS device of higher quality.  This external GPS device was located in the front of the airplane for five runs and for the other five runs was moved to a seat where the plane's wings would likely obscure signals from many GPS satellites.

The raw data takes the form of a series of multiple text files with data conforming to the National Marine Electronics Association (NMEA) format.  These are read by the DMSM software, which outputs positional information as a feature class in an ESRI geodatabase.  It is thought that in particular hypothesis 4 can be addressed via a direct comparison of positional information from these two sources.

Hypothesis 1 would be consistent with observing many temporal discontinuities in the GPS information.  Hypothesis 2 would be consistent with positional information that is either highly variable or reported as having high positional uncertainty.

Hypothesis 3 would be consistent with observing GPS coordinates that suggest that the plane is unrealistically speeding up and/or slowing down as it progresses through its course.

#### Software Environment - sessionInfo()

```r
sessionInfo()
```

```
## R version 3.3.0 (2016-05-03)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 7 x64 (build 7601) Service Pack 1
## 
## locale:
## [1] LC_COLLATE=English_United States.1252 
## [2] LC_CTYPE=English_United States.1252   
## [3] LC_MONETARY=English_United States.1252
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.1252    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] knitr_1.13
## 
## loaded via a namespace (and not attached):
##  [1] magrittr_1.5    formatR_1.4     htmltools_0.3.5 tools_3.3.0    
##  [5] Rcpp_0.12.5     stringi_1.1.1   rmarkdown_0.9.6 stringr_1.0.0  
##  [9] digest_0.6.9    evaluate_0.9
```
Random Number Seed: 19680828

```r
set.seed(19680828)
```
Working Directory

```r
setwd("D:\\Projects\\GIS_Projects\\20160617 DMSM GPS")
```

***

### Analytical Process
#### Data Location
All data for this investigation has been placed in the following directory:
\\sxfnlnas001\fhtet\Projects\DMSM\GPS

#### 20160617 Discussion with Sapio, Monahan, Edberg, Conly
Initial briefing of issue and hypothesis.  Discussion of data format and location.  Andy Trent at Missoula is listed as a good contact.  The data is stored according to tablet owner, not tablet user.  The three tablets are:  
1. "Jeff" - used by Nathan - connected by bluetooth to the external GPS  
2. "Drew" - used by Jeff  
3. "Jeanine" - used by Frank

The raw NMEA files show a variety of information sequentially output over time.  Each line begins with a five character delimeter preceeded by a dollar sign, which shows the type of information to follow to the EOL. So far it appears that four our purposes, the necessary data for the present analysis should be found exclusively in lines with the following delimiters:

$GPGGA - showing position and time  
$GPGSV - satellites used in the present position calculation

More specifically, according to aprs.gids.nl/nmea these lines are delimited as follows:

$GPGGA,hhmmss.ss,llll.ll,a,yyyyy.yy,a,x,xx,x.x,x.x,M,x.x,M,x.x,xxxx*hh  
1    = UTC of Position  
2    = Latitude  
3    = N or S  
4    = Longitude  
5    = E or W  
6    = GPS quality indicator (0=invalid; 1=GPS fix; 2=Diff. GPS fix)  
7    = Number of satellites in use [not those in view]  
8    = Horizontal dilution of position  
9    = Antenna altitude above/below mean sea level (geoid)  
10   = Meters  (Antenna height unit)  
11   = Geoidal separation (Diff. between WGS-84 earth ellipsoid and  
       mean sea level.  -=geoid is below WGS-84 ellipsoid)  
12   = Meters  (Units of geoidal separation)  
13   = Age in seconds since last update from diff. reference station  
14   = Diff. reference station ID#  
15   = Checksum  

$GPGSV,1,1,13,02,02,213,,03,-3,000,,11,00,121,,14,13,172,05*67  
1    = Total number of messages of this type in this cycle  
2    = Message number  
3    = Total number of SVs in view  
4    = SV PRN number  
5    = Elevation in degrees, 90 maximum  
6    = Azimuth, degrees from true north, 000 to 359  
7    = SNR, 00-99 dB (null when not tracking)  
8-11 = Information about second SV, same as field 4-7  
12-15= Information about third SV, same as field 4-7  
16-19= Information about fourth SV, same as field 4-7  

Investigations should begin with the information in the GPGGA lines.

#### 20160617 Data Investigation
**Drew's Tablet (Operated by Jeff)**  
A series of what appear to be 11 raw NMEA files are observed, along with:
Ten jpg files showing different passes
Jeanine.mxd (contents unknown as it was made using a later version of ArcGIS)

Useful_nmea_drew - contains only GPGGA, GPGSV, and GPRMC tags, perhaps a subsetted concatenation of all the raw files  
GPGGA_drew.csv - GPGGA lines only

I checked that the first GPGGA line of every raw NMEA file is also visible in GPGGA_drew.csv.  We'll go with primarily this file for analysis.

**Jeanine's Tablet (Operated by Frank)**  
Again, a series of what appear to be 11 raw NMEA files are observed, along with two other files:  
Useful_nmea_jlp - contains only GPGGA, GPGSV, and GPRMC tags, perhaps a subsetted concatenation of all the raw files  
GPGGA_jlp.csv - 315 GPGGA records from time stamp 164549 to 183753  
GPGGA_jlp.xlsx - Same contents as previous file  
GPGGA_jlp2.csv - Same as previous file but only showing LAT/LONG (corrected), Fix_Qual, Num_Sat, HDOP, and ASL  

We need the time stamp, so we'll go with the first file for analysis of incoming GPS data to DMSM.

There is also an offlinedata.gdb ESRI GDB that contains feature class LOCATION_POINTS, which appears to contain the final GPS data post DMSM.  No other point feature classes exist in the GDB that contain significant data.  We'll go with LOCATION_POINTS for the post-DMSM GPS data.

**Jeff's Tablet (Operated by Nathan)**
Raw data files do not exist in the same form as for the other tablets.  Directories exist showing a full dump of tablet contents and a series of screenshots.  Also a series of jpg files are observed similar to those of Drew's tablet.  In addition, we observe the following:  
Useful_nmea_mai.txt - GPGGA, GPGSV,GPRMC tags from time stamp 183859.89 to 184536.00 and then what appears to be a lot of missing data from time stamp 000000.00 to 000553.00.  
GPGGA_mai.csv - GPGGA lines only from time stamp 183900 to 184536 and same missing data from 000000 to 000553.  
Analysis.xlsx - Two worksheets  
- 2015-10-09_113518_BTWING$ - Records from time stamp 17:35:19 - 18:45:36  
- All$ - Records from time stamp 17:17:27 - 17:34:59 (ironically fewer records)  
BTWING.shp - Point shape file containing records from time stamp 17:35:19 - 18:45:36  

The most complete set of records appears to come from either the first worksheet of Analysis.xlsx or BTWING.shp.  We'll go with these for the pre-DMSM GPS data.

There is also an offlinedata.gdb ESRI GDB that contains feature class LOCATION_POINTS, which appears to contain the final GPS data post DMSM.  No other point feature classes exist in the GDB that contain significant data.  We'll go with LOCATION_POINTS for the post-DMSM GPS data.

One remaining concern is that I am not sure how to discern the observations with the GPS device in the seat near the plane wing.  The shape file called BTWING.shp appears to contain not half the data, but all the data.

#### 20160617 Email/Discussion with Conly
Conly has inspected the DMSM code for positional averaging algorithms.  None were found.  Hypothesis 4 appears unlikely.  However, an algorithm is utilized by the software to anticipate changes in direction, using an ongoing regression algorithm on directional angle.  Perhaps if it turns out that hypotheses 1 or 2 are true, then a similar algorithm can be utilized to project the plane's movement more smoothly.

His full email follows:  
Most of the code for dealing with GPS points in DMSM is in a function in a 280 line function that checks the preferences for: minimum distance, logging point spacing, observer position and smoothing window.

1.  If the distance from the last point is less than specified, it doesn't do anything with the point coming from GPS. It won't log the point, calculate rotation, or update the plane location.  
2.  If distance is greater than logging spacing, it will log it. You cannot have a logging distance less than your minimum distance.  
3.  Location smoothing 1) calculate angle from linear regression and then 2) calculate angle from endpoints . There's some  logic to resolve which one to use. I'll omit the code here, but I'm happy to share it with John or whomever - it shouldn't be causing the location lag unless it takes too long to execute.  
4.  Resolves the observer position with rotation - there's more complex logic here.  
5.  Finally it updates the plane location on the screen using location,  observer position and rotation.  

The bottom line here is that it will not update the plane location unless the distance exceeds the threshold established by minimum distance.  We could ask the people complaining about this where their minimum distance is set. The default is 5m - which should be small enough. 

Here are a couple of clarifications - 

Minimum distance is the increment in which everything else occurs. 
Location smoothing is only relevant when calculating the angle of the plane. There is no location averaging otherwise. 

#### Preprocessing Steps

The following files were created from manual procedures in ArcGIS:

***"D:\\Projects\\GIS_Projects\\20160617 DMSM GPS\\Frank_post.txt"***  
Exported the Attribute table from the feature class "\\\\sxfnlnas001\\fhtet\\Projects\\DMSM\\GPS\\1510_GPSTest\\Jeanine_Tab\\offlinedata.gdb\\LOCATION_POINTS

***"D:\\Projects\\GIS_Projects\\20160617 DMSM GPS\\Nathan_post.txt"***  
Exported the Attribute table from the feature class "\\\\sxfnlnas001\\fhtet\\Projects\\DMSM\\GPS\\1510_GPSTest\\Jeff_Tab\\offlinedata.gdb\\LOCATION_POINTS

#### Analysis 20160617

We name the datasets according to operator.

#### Jeff

Reading in the data:

```r
Jeff_preFN <- "X:\\Projects\\DMSM\\GPS\\1510_GPSTest\\Drew_Tab\\GPGGA_drew.csv"
Jeff_pre <- read.csv(Jeff_preFN,header=TRUE,row.names=NULL)
```

We have an initial inspection of the dataset:

```r
str(Jeff_pre)
```

```
## 'data.frame':	196 obs. of  15 variables:
##  $ row.names: chr  "$GPGGA" "$GPGGA" "$GPGGA" "$GPGGA" ...
##  $ Sentence : num  163931 163932 163933 163934 163935 ...
##  $ Time_Z   : num  4053 4053 4053 4053 4053 ...
##  $ Lat      : Factor w/ 2 levels "","N": 2 2 2 2 2 2 2 2 2 2 ...
##  $ N_S      : num  10528 10528 10528 10529 10529 ...
##  $ Long     : Factor w/ 2 levels "","W": 2 2 2 2 2 2 2 2 2 2 ...
##  $ E_W      : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Fix_Qual : int  13 13 14 13 15 14 14 14 14 14 ...
##  $ Num_Sat  : num  3.2 3.2 2.4 2.4 2.4 2.4 2.4 1.6 1.6 1.6 ...
##  $ HDOP     : num  2906 2909 2911 2912 2911 ...
##  $ ASL      : Factor w/ 1 level "M": 1 1 1 1 1 1 1 1 1 1 ...
##  $ H_geoid  : num  -20.5 -20.5 -20.5 -20.5 -20.5 -20.5 -20.5 -20.5 -20.5 -20.5 ...
##  $ Time_last: Factor w/ 1 level "M": 1 1 1 1 1 1 1 1 1 1 ...
##  $ DGPS_id  : logi  NA NA NA NA NA NA ...
##  $ Checksum : Factor w/ 22 levels "*50","*51","*52",..: 22 17 22 21 14 16 7 19 8 7 ...
```

```r
head(Jeff_pre)
```

```
##   row.names Sentence   Time_Z Lat      N_S Long E_W Fix_Qual Num_Sat
## 1    $GPGGA   163931 4053.044   N 10528.38    W   1       13     3.2
## 2    $GPGGA   163932 4053.044   N 10528.43    W   1       13     3.2
## 3    $GPGGA   163933 4053.044   N 10528.49    W   1       14     2.4
## 4    $GPGGA   163934 4053.045   N 10528.54    W   1       13     2.4
## 5    $GPGGA   163935 4053.046   N 10528.59    W   1       15     2.4
## 6    $GPGGA   163936 4053.047   N 10528.64    W   1       14     2.4
##     HDOP ASL H_geoid Time_last DGPS_id Checksum
## 1 2905.6   M   -20.5         M      NA      *6F
## 2 2909.1   M   -20.5         M      NA      *6A
## 3 2911.3   M   -20.5         M      NA      *6F
## 4 2911.5   M   -20.5         M      NA      *6E
## 5 2911.2   M   -20.5         M      NA      *67
## 6 2910.5   M   -20.5         M      NA      *69
```

We only need the fields "TimeZ","Lat","Long","NumSat","HDOP" and need to change the Lat and Long fields to decimal degrees.  We also remove lines containing missing data:

```r
Jeff_pre <- Jeff_pre[,c(2,3,5,9,10)]
colnames(Jeff_pre) <- c("TimeZ","Lat","Long","NumSat","HDOP")
Jeff_pre$Lat <- floor(Jeff_pre$Lat/100) + (Jeff_pre$Lat %% 100)/60
Jeff_pre$Long <- -1*(floor(Jeff_pre$Long/100) + (Jeff_pre$Long %% 100)/60)
Jeff_pre <- Jeff_pre[complete.cases(Jeff_pre),]
```

Plotting this dataset in latitude/longitude coordinates:

```r
plot(Jeff_pre$Long,Jeff_pre$Lat,xlab="Longitude",ylab="Latitude",main="Dataset - Jeff")
```

![plot of chunk Jeff04](figure/Jeff04-1.png)

***

#### Frank

Reading in the Pre-DMSM data:

```r
Frank_preFN <- "X:\\Projects\\DMSM\\GPS\\1510_GPSTest\\Jeanine_Tab\\GPGGA_jlp.csv"
Frank_pre <- read.csv(Frank_preFN,header=TRUE)
```

We have an initial inspection of the dataset:

```r
str(Frank_pre)
```

```
## 'data.frame':	314 obs. of  15 variables:
##  $ Sentence: Factor w/ 1 level "$GPGGA": 1 1 1 1 1 1 1 1 1 1 ...
##  $ Time_Z  : num  164549 164550 164551 164552 164553 ...
##  $ Lat     : num  4057 4057 4057 4057 4057 ...
##  $ N_S     : Factor w/ 1 level "N": 1 1 1 1 1 1 1 1 1 1 ...
##  $ Long    : num  10519 10519 10519 10519 10519 ...
##  $ E_W     : Factor w/ 1 level "W": 1 1 1 1 1 1 1 1 1 1 ...
##  $ Fix_Qual: int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Num_Sat : int  14 13 12 13 14 14 14 14 14 12 ...
##  $ HDOP    : num  2.4 1.6 3.2 2.4 2.4 2.4 3.2 2.4 4.8 4.8 ...
##  $ ASL     : num  2900 2900 2901 2901 2902 ...
##  $ X       : Factor w/ 1 level "M": 1 1 1 1 1 1 1 1 1 1 ...
##  $ H_geoid : num  -20.4 -20.4 -20.4 -20.4 -20.4 -20.4 -20.4 -20.4 -20.4 -20.4 ...
##  $ X.1     : Factor w/ 1 level "M": 1 1 1 1 1 1 1 1 1 1 ...
##  $ X.2     : logi  NA NA NA NA NA NA ...
##  $ Checksum: Factor w/ 16 levels "*60","*61","*62",..: 10 16 7 8 16 1 4 15 16 4 ...
```

```r
head(Frank_pre)
```

```
##   Sentence Time_Z      Lat N_S     Long E_W Fix_Qual Num_Sat HDOP    ASL X
## 1   $GPGGA 164549 4057.443   N 10518.89   W        1      14  2.4 2900.2 M
## 2   $GPGGA 164550 4057.441   N 10518.94   W        1      13  1.6 2900.5 M
## 3   $GPGGA 164551 4057.435   N 10518.99   W        1      12  3.2 2900.7 M
## 4   $GPGGA 164552 4057.432   N 10519.05   W        1      13  2.4 2901.2 M
## 5   $GPGGA 164553 4057.427   N 10519.10   W        1      14  2.4 2902.5 M
## 6   $GPGGA 164554 4057.427   N 10519.15   W        1      14  2.4 2902.5 M
##   H_geoid X.1 X.2 Checksum
## 1   -20.4   M  NA      *69
## 2   -20.4   M  NA      *6F
## 3   -20.4   M  NA      *66
## 4   -20.4   M  NA      *67
## 5   -20.4   M  NA      *6F
## 6   -20.4   M  NA      *60
```

We only need the "TimeZ","Lat","Long","NumSat","HDOP" fields, and need to change Lat and Long to decimal degrees:

```r
Frank_pre <- Frank_pre[,c(2,3,5,8,9)]
colnames(Frank_pre) <- c("TimeZ","Lat","Long","NumSat","HDOP")
Frank_pre$Lat <- floor(Frank_pre$Lat/100) + (Frank_pre$Lat %% 100)/60
Frank_pre$Long <- -1*(floor(Frank_pre$Long/100) + (Frank_pre$Long %% 100)/60)
```

Plotting this dataset in latitude/longitue coordinates:

```r
plot(Frank_pre$Long,Frank_pre$Lat,xlab="Longitude",ylab="Latitude",main="Dataset - Frank (pre)")
```

![plot of chunk Frank04](figure/Frank04-1.png)

Reading in the Post-DMSM data:

```r
Frank_postFN <- "D:\\Projects\\GIS_Projects\\20160617 DMSM GPS\\Frank_post.txt"
Frank_post <- read.csv(Frank_postFN,header=TRUE)
```

We have an initial inspection of the dataset:

```r
str(Frank_post)
```

```
## 'data.frame':	1930 obs. of  12 variables:
##  $ FID       : Factor w/ 1930 levels "0","1","1,000",..: 1 2 1043 1154 1265 1376 1487 1598 1709 1820 ...
##  $ DATE      : Factor w/ 1 level "10/9/2015 0:00:00": 1 1 1 1 1 1 1 1 1 1 ...
##  $ ELEVATION : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ STATUS    : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ REGION_ID : int  2 2 2 2 2 2 2 2 2 2 ...
##  $ LOCATION_P: Factor w/ 1930 levels "{000f650a-64eb-4b25-b0da-5e6b6fb11d8d}",..: 1553 1064 961 1395 1678 1599 117 1140 1137 1420 ...
##  $ MISSION_ID: logi  NA NA NA NA NA NA ...
##  $ GlobalID  : Factor w/ 1930 levels "{004C57F7-52A5-4224-8DE2-4023FEF143FB}",..: 1647 1830 1312 1155 1034 658 604 1166 1765 1054 ...
##  $ OBJECTID  : Factor w/ 1930 levels "1","1,002","1,003",..: 1 888 1360 1443 1535 1617 1693 1764 1862 785 ...
##  $ Pass      : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ POINT_X   : num  -105 -105 -105 -105 -105 ...
##  $ POINT_Y   : num  40.4 40.4 40.4 40.4 40.4 ...
```

```r
head(Frank_post)
```

```
##   FID              DATE ELEVATION STATUS REGION_ID
## 1   0 10/9/2015 0:00:00         0      0         2
## 2   1 10/9/2015 0:00:00         0      0         2
## 3   2 10/9/2015 0:00:00         0      0         2
## 4   3 10/9/2015 0:00:00         0      0         2
## 5   4 10/9/2015 0:00:00         0      0         2
## 6   5 10/9/2015 0:00:00         0      0         2
##                               LOCATION_P MISSION_ID
## 1 {cf03aadb-f3e2-4239-8c08-69afb06a16eb}         NA
## 2 {8dedf83a-12b7-4ced-8345-617ba86b0955}         NA
## 3 {810a1d37-a613-4975-a217-36abe4b09d2e}         NA
## 4 {b6a70192-7f37-4b49-810b-cc9e297b1cfa}         NA
## 5 {e01380aa-db2b-48e4-9450-6cf1e1829089}         NA
## 6 {d6293fce-faf8-4018-9dc4-676b3a1a9674}         NA
##                                 GlobalID OBJECTID Pass   POINT_X  POINT_Y
## 1 {DCB56D16-F861-4EDE-9040-E016682F3DBE}        1    0 -105.0080 40.44935
## 2 {F2D7EB16-73EE-43C5-96C3-BBA69456F668}        2    0 -105.0086 40.44662
## 3 {B07035FB-AE19-4560-A547-13B977FF8284}        3    0 -105.0082 40.44965
## 4 {9B2686DC-0DAC-41AF-8F01-83668E17295F}        4    0 -105.0082 40.44604
## 5 {89DE5BED-70DE-4366-B012-1E69CCAFF260}        5    0 -105.0073 40.44389
## 6 {5456114A-2D5C-4F30-85F6-213F138908DD}        6    0 -105.0061 40.44130
```

Lat and Long fields appear to be already converted, and only the Lat and Long fields are useable:

```r
Frank_post <- Frank_post[,11:12]
colnames(Frank_post) <- c("Long","Lat")
```

Plotting this dataset in the latitude/longitude plane:

```r
plot(Frank_post$Long,Frank_post$Lat,pch=19,col="blue",xlab="Longitude",ylab="Latitude",main="Dataset - Frank (post)")
```

![plot of chunk Frank08](figure/Frank08-1.png)

Frank's Post-DMSM dataset contains 1930 observations, significantly more than the 314 observations in his Pre-DMSM dataset.  Interpolation by the the DMSM software is imiplicated.

***

#### Nathan

Reading in the Pre-DMSM data:

```r
library(xlsx)
```

```
## Loading required package: rJava
```

```
## Loading required package: xlsxjars
```

```r
Nathan_preFN <- "X:\\Projects\\DMSM\\GPS\\1510_GPSTest\\Jeff_Tab\\Analysis.xlsx"
Nathan_pre <- read.xlsx(Nathan_preFN,1,header=TRUE)
```

We have an initial inspection of the dataset:

```r
str(Nathan_pre)
```

```
## 'data.frame':	1024 obs. of  20 variables:
##  $ Lat        : num  40.9 40.9 40.9 40.9 40.9 ...
##  $ Lng        : num  -105 -105 -105 -105 -105 ...
##  $ Alt        : num  2889 2889 2889 2888 2888 ...
##  $ Acc        : num  2.05 2.05 2.28 2.31 2.31 2.32 2.32 2.32 2.32 2.07 ...
##  $ Time       : Factor w/ 1022 levels "2015-10-09T17:17:27.000Z",..: 1 2 3 4 5 6 7 8 9 10 ...
##  $ Prv        : Factor w/ 1022 levels "17:17:27.000Z",..: 1 2 3 4 5 6 7 8 9 10 ...
##  $ OrgLat     : num  40.9 40.9 40.9 40.9 40.9 ...
##  $ OrgLng     : num  -105 -105 -105 -105 -105 ...
##  $ OrgAlt     : num  2889 2889 2889 2888 2888 ...
##  $ OrgAcc     : num  2.45 2.47 2.92 2.92 2.92 2.92 2.92 2.92 2.92 2.47 ...
##  $ Speed      : num  82.1 82.2 82.3 82.3 82.4 ...
##  $ Bearing    : num  86.4 86.4 86.3 86.3 86.3 ...
##  $ AdvPrv     : Factor w/ 1 level "gps [GPS]": 1 1 1 1 1 1 1 1 1 1 ...
##  $ Dly        : POSIXct, format: "1899-12-30 00:00:00" "1899-12-30 00:00:01" ...
##  $ Dst        : num  0 82.3 72.9 80.8 82.1 82.3 82.4 82.3 82.4 92.1 ...
##  $ AltOfst    : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ Pressure   : logi  NA NA NA NA NA NA ...
##  $ PressureRef: logi  NA NA NA NA NA NA ...
##  $ RefAge     : logi  NA NA NA NA NA NA ...
##  $ FromBT     : logi  FALSE FALSE FALSE FALSE FALSE FALSE ...
```

```r
head(Nathan_pre)
```

```
##        Lat       Lng    Alt  Acc                     Time           Prv
## 1 40.88961 -105.4316 2889.0 2.05 2015-10-09T17:17:27.000Z 17:17:27.000Z
## 2 40.88966 -105.4307 2888.9 2.05 2015-10-09T17:17:28.000Z 17:17:28.000Z
## 3 40.88970 -105.4298 2888.7 2.28 2015-10-09T17:17:29.000Z 17:17:29.000Z
## 4 40.88975 -105.4288 2888.5 2.31 2015-10-09T17:17:30.000Z 17:17:30.000Z
## 5 40.88980 -105.4279 2888.3 2.31 2015-10-09T17:17:31.000Z 17:17:31.000Z
## 6 40.88985 -105.4269 2888.2 2.32 2015-10-09T17:17:32.000Z 17:17:32.000Z
##     OrgLat    OrgLng OrgAlt OrgAcc Speed Bearing    AdvPrv
## 1 40.88963 -105.4312 2889.0   2.45 82.08   86.43 gps [GPS]
## 2 40.88968 -105.4302 2888.9   2.47 82.18   86.36 gps [GPS]
## 3 40.88973 -105.4292 2888.7   2.92 82.28   86.33 gps [GPS]
## 4 40.88978 -105.4283 2888.5   2.92 82.31   86.30 gps [GPS]
## 5 40.88983 -105.4273 2888.3   2.92 82.39   86.26 gps [GPS]
## 6 40.88988 -105.4263 2888.2   2.92 82.41   86.25 gps [GPS]
##                   Dly  Dst AltOfst Pressure PressureRef RefAge FromBT
## 1 1899-12-30 00:00:00  0.0       0       NA          NA     NA  FALSE
## 2 1899-12-30 00:00:01 82.3       0       NA          NA     NA  FALSE
## 3 1899-12-30 00:00:01 72.9       0       NA          NA     NA  FALSE
## 4 1899-12-30 00:00:01 80.8       0       NA          NA     NA  FALSE
## 5 1899-12-30 00:00:01 82.1       0       NA          NA     NA  FALSE
## 6 1899-12-30 00:00:01 82.3       0       NA          NA     NA  FALSE
```

Lat and Long fields appear to be already converted.  We only need the "Lat","Long","Alt","Acc","Time","OrgAcc","Speed","Bearing" fields:

```r
Nathan_pre <- Nathan_pre[,c(1,2,3,4,5,10,11,12)]
colnames(Nathan_pre) <- c("Lat","Long","Alt","Acc","Time","OrgAcc","Speed","Bearing")
```

Plotting this dataset in the latitude/longitude plane:

```r
plot(Nathan_pre$Long,Nathan_pre$Lat,pch=19,col="blue",xlab="Longitude",ylab="Latitude",main="Dataset - Nathan (pre)")
```

![plot of chunk Nathan04](figure/Nathan04-1.png)

Reading in the Post-DMSM data:

```r
Nathan_postFN <- "D:\\Projects\\GIS_Projects\\20160617 DMSM GPS\\Nathan_post.txt"
Nathan_post <- read.csv(Nathan_postFN,header=TRUE)
```

We have an initial inspection of the dataset:

```r
str(Nathan_post)
```

```
## 'data.frame':	1809 obs. of  13 variables:
##  $ FID       : Factor w/ 1809 levels "0","1","1,000",..: 1 2 922 1033 1144 1255 1366 1477 1588 1699 ...
##  $ DATE      : Factor w/ 1 level "10/9/2015 0:00:00": 1 1 1 1 1 1 1 1 1 1 ...
##  $ ELEVATION : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ STATUS    : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ REGION_ID : int  2 2 2 2 2 2 2 2 2 2 ...
##  $ LOCATION_P: Factor w/ 1809 levels "{0027c88c-d1c9-4181-9dd4-ca595c404fc2}",..: 911 1791 164 1385 1347 659 903 684 1391 753 ...
##  $ MISSION_ID: logi  NA NA NA NA NA NA ...
##  $ GlobalID  : Factor w/ 1809 levels "{009D737B-1C05-42F9-A7E2-AE8239B1AB1F}",..: 937 1350 330 1586 1039 983 258 666 905 803 ...
##  $ OBJECTID  : Factor w/ 1809 levels "1","1,340","1,341",..: 1 574 1267 1378 1489 1600 1711 1788 1799 464 ...
##  $ Pass      : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ HDOP      : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ POINT_X   : num  -105 -105 -105 -105 -105 ...
##  $ POINT_Y   : num  40.6 40.6 40.6 40.6 40.6 ...
```

```r
head(Nathan_post)
```

```
##   FID              DATE ELEVATION STATUS REGION_ID
## 1   0 10/9/2015 0:00:00         0      0         2
## 2   1 10/9/2015 0:00:00         0      0         2
## 3   2 10/9/2015 0:00:00         0      0         2
## 4   3 10/9/2015 0:00:00         0      0         2
## 5   4 10/9/2015 0:00:00         0      0         2
## 6   5 10/9/2015 0:00:00         0      0         2
##                               LOCATION_P MISSION_ID
## 1 {81e7bd2e-dfb1-4bb4-ac5f-aeea0720c2bb}         NA
## 2 {fc7f0465-861e-4964-9c81-0c8c314a6fa8}         NA
## 3 {1882fb2b-42bf-432f-95e1-dea47a88a050}         NA
## 4 {c4a1ca67-2b38-4ac0-a55d-6697d57bfad5}         NA
## 5 {c01c9e23-9dc7-42e1-a84f-1e393e9c7254}         NA
## 6 {5add74ca-0bc1-4a13-a120-fccf3e2cf583}         NA
##                                 GlobalID OBJECTID Pass HDOP   POINT_X
## 1 {807728AA-6100-4A5E-BC7B-D3942BF68B24}        1    0    0 -105.1532
## 2 {B76C4CF5-939C-44A9-B951-243D78702496}        2    0    0 -105.1560
## 3 {2B756310-F87F-449E-B867-2766CC31A1A3}        3    0    0 -105.1589
## 4 {DAC24760-0B1E-4F77-953A-58C4EAFA8EE9}        4    0    0 -105.1618
## 5 {8DD26C98-EE61-419C-BBBB-4994BD28C85F}        5    0    0 -105.1647
## 6 {86344A1A-F93D-4923-B5CD-D09C9C47A0F4}        6    0    0 -105.1675
##    POINT_Y
## 1 40.59375
## 2 40.59564
## 3 40.59754
## 4 40.59944
## 5 40.60133
## 6 40.60322
```

Lat and Long fields appear to be already converted, and only the Lat and Long fields are useable:

```r
Nathan_post <- Nathan_post[,12:13]
colnames(Nathan_post) <- c("Long","Lat")
```

Plotting this dataset in the latitude/longitude plane:

```r
plot(Nathan_post$Long,Nathan_post$Lat,pch=19,col="blue",xlab="Longitude",ylab="Latitude",main="Dataset - Nathan (post)")
```

![plot of chunk Nathan08](figure/Nathan08-1.png)

Nathan's Post-DMSM dataset shows 1809 observations, again more than the 1024 observations in the Pre-DMSM dataset, again implying interpolation by DMSM.

### Discussion



***

### Conclusion

***

### References

***

#### Analysis End
