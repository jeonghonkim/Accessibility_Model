# Accessibility_Model

`CBRE - Location Intelligence`
<br />Accessibility Model Ver.3
<br />Completed as of March 21, 2023
<br><br>

> **Summary** <br>
 
This arcpy tool allows users to calculate drive times, miles, number of turn right and left, traffic volumes
decays from starting points in major roads or streets to target destinations. It uses one csv file, which includes 
market name, latitude, and longitude fields, as an input. With the lat, long values, it locates market sites and 
creates buffers with user assgiend miles from the markets and starting points, which were the intersections between 
marekts and buffers. Also this tool creates routes and directions and allows users to customize departure time, 
wihch will affect the drive time. After running this tool, it adds four feature layers, which were markets, starts,
buffers and routes, to the current map in a project, and the markets and routes layers have statistics tables.

<br><br>

> **Updates** <br>

* Ver.3 &nbsp;|&nbsp; March 2023 <br>
   - [x] Created routes and directions with Closest Facility nax module, instead of na module[^1]
   - [x] Added title name parameters to save outputs in different feature datasetrs[^2]
   - [X] Created new definitions to count number of turn right and left
   - [X] Added the outputs to the current map and rendered the layers[^3]

* Ver.2 &nbsp;|&nbsp; December 2022 <br>
   - [X] Created new fields for accessibility score and traffic decays
   - [X] Captured traffics by routes with closest spatial-join

* Ver.1 &nbsp;|&nbsp; May - August 2022 <br>
   - [X] Created a point feature layers with a csv file with lat, long values[^4]
   - [X] Captured traffics by routes with closest spatial-join


<br><br>


> **Parameters & Inputs** <br>

<img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_parameters.JPG" width="400" height="500"> &nbsp; &nbsp; <img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_examplecsv.JPG" width="300" height="500"> <br>

* *Workspace* &nbsp;|&nbsp; The current project's geodatabase is the default value
* *Title Name* &nbsp;|&nbsp; There should be no space in this parameter because it will be the name of new feature dataset in the workspace. 
* *Target CSV* &nbsp;|&nbsp; The csv should include three fields, which represent name, latitude value and logitude value for the target markets.
* *Latitude Field* &nbsp;|&nbsp; You can select the latitude field in the entered csv file. 
* *Longitutde Field* &nbsp;|&nbsp; You can select the longitude field in the entered csv file.
* *Buffer Distance* &nbsp;|&nbsp; You can enter the mile-distance from market locations as you wish.
* *Departure Time* &nbsp;|&nbsp; You can enter exact time will generate drive times. The currnet time will be applied if you select the clock symbol.

<br><br>

> **Detailed Steps** <br>

**1. Create analysis objects** <br>
&nbsp;&nbsp;&nbsp;*1) Feature dataset*<br>
&nbsp;&nbsp;&nbsp;Creating a feature dataset has multiple benefits. First, you can run the analysis multiple times with different feature names. Also, you can easily find the feature analysis outcomes in different feature dataset. The project title name you enter would be the feature datasetanme.<br>
&nbsp;&nbsp;&nbsp;*2) Market locations and buffers*<br>
&nbsp;&nbsp;&nbsp;Once you run the tool after entering the prepared csv file with name, latitude and longitude as well as the buffer-mile distance, it will create market locations as well as buffers feature layers. They will be saved in the generated feature dataset.<br>
&nbsp;&nbsp;&nbsp;*3) Starting road points*<br>
&nbsp;&nbsp;&nbsp;Starting points are the intersection between the assigned street or roads layers and the generated buffers. You can change the 'major_roads' path in the code as you wish, such as all streets or major roads. Major roads are starting points in this analysis.<br>
&nbsp;&nbsp;&nbsp;*4) Multipart to single part*<br>
&nbsp;&nbsp;&nbsp;The generated starting points can be a multipart feature point layer. In order to make all points as unique starts, use the multipart to single part geoprocessing tool.<br>

<br>

```diff
# Set parameters
work_dbs = arcpy.GetParameterAsText(0) # workspace, current workspace
title_name = arcpy.GetParameterAsText(1) # project title / feature dataset name, String
trgt_csv = arcpy.GetParameterAsText(2) # target site csv, table or table view
lat_field = arcpy.GetParameterAsText(3) # latitude field in target sv, field
long_field = arcpy.GetParameterAsText(4) # long field in target csv, field
buff_dis = arcpy.GetParameterAsText(5) # "0.25 Miles" Default, linear unit
depart_time = arcpy.GetParameterAsText(6) # date & time, date

# Define project, map, and workspace
arcpy.env.workspace = work_dbs
workspace = work_dbs
aprx = arcpy.mp.ArcGISProject("CURRENT") # Current Project
aprxMap = aprx.listMaps("Map")[0] # Dafault Map
out_coordinate_system = arcpy.SpatialReference("WGS 1984")

# 1. Create analysis objects
# 1) Feature Dataset
arcpy.management.CreateFeatureDataset(workspace, title_name, arcpy.SpatialReference("WGS 1984"))
accessbility_fd = os.path.join(workspace, title_name)

# 2) Market locations and buffers
market_p = os.path.join(accessbility_fd, title_name+"_Markets_Original")
arcpy.management.XYTableToPoint(trgt_csv, market_p, long_field, lat_field, arcpy.SpatialReference("WGS 1984"))
buffer = os.path.join(accessbility_fd, title_name+"_Buffers")
arcpy.Buffer_analysis(market_p, buffer, buff_dis) 

# 3) Starting road points
major_roads = r"MapMajorRoads"
start_multi = os.path.join(accessbility_fd, title_name+"_Start_Multi")
arcpy.Intersect_analysis([buffer, major_roads], start_multi, "", "", "point")

# 4) Multipart to singlepart
start_p = os.path.join(accessbility_fd, title_name+"_AllStreets")
arcpy.MultipartToSinglepart_management(start_multi, start_p)
arcpy.management.Delete(start_multi)

```
<br>

in the feature datatset with project title name. 



```diff

# Converted to signle-part point by using the Multipart to Singlepart tool
start_p = os.path.join(accessbility_fd, title_name+"_AllStreets")
arcpy.MultipartToSinglepart_management(start_multi, start_p)
arcpy.management.Delete(start_multi)
```
3. 
4. dfadf
5. 
6. 

[^1]: https://pro.arcgis.com/en/pro-app/latest/arcpy/network-analyst/closestfacility.htm
[^2]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/create-feature-dataset.htm
[^3]: https://pro.arcgis.com/en/pro-app/latest/arcpy/mapping/simplerenderer-class.htm
[^4]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/xy-table-to-point.htm
