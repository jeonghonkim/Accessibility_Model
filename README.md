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

1. Create analysis objects

Once you run the tool after entering the prepared csv file with name, latitude and longitude as well as the buffer-mile distance, it will create three feature layers, which were Markets, Buffers, and Starts, in the feature datatset with project title name. Starting points are the intersection between the assigned street or roads layers and the buffers. You can change the 'major_roads' path in the code as you wish, such as all streets or only for major roads.

<br>

```diff
# Create a Feature Dataset
arcpy.management.CreateFeatureDataset(out_dataset_path = workspace, 
                            out_name = title_name,
                            spatial_reference = arcpy.SpatialReference("WGS 1984"))
accessbility_fd = os.path.join(workspace, title_name)

# Create unprojected feature layer for office sites
market_p = os.path.join(accessbility_fd, title_name+"_Markets_Original")
arcpy.management.XYTableToPoint(
    in_table = trgt_csv, 
    out_feature_class = market_p, 
    x_field = long_field, 
    y_field = lat_field, 
    coordinate_system = arcpy.SpatialReference("WGS 1984"))

# Create Buffer
buffer = os.path.join(accessbility_fd, title_name+"_Buffers")
arcpy.Buffer_analysis(market_p, buffer, buff_dis) 

# Create intersections between the buffer and major roads feature layer
#major_roads = r"C:\ArcGIS\Business Analyst\US_2022\Data\Streets Data\NorthAmerica.gdb\Streets"
major_roads = r"C:\ArcGIS\Business Analyst\US_2022\Data\Streets Data\NorthAmerica.gdb\MapMajorRoads\MapMajorRoads"
start_multi = os.path.join(accessbility_fd, title_name+"_Start_Multi")
arcpy.Intersect_analysis([buffer, major_roads], start_multi, "", "", "point")

# Converted to signlepar tpoint by using the Multipart to Singlepart tool
start_p = os.path.join(accessbility_fd, title_name+"_AllStreets")
arcpy.MultipartToSinglepart_management(start_multi, start_p)
arcpy.management.Delete(start_multi)
```
<br>

2. Create analysis objects

```diff
# Create a Feature Dataset
arcpy.management.CreateFeatureDataset(out_dataset_path = workspace, 
                            out_name = title_name,
                            spatial_reference = arcpy.SpatialReference("WGS 1984"))
accessbility_fd = os.path.join(workspace, title_name)
```
3. 
4. dfadf
5. 
6. 

[^1]: https://pro.arcgis.com/en/pro-app/latest/arcpy/network-analyst/closestfacility.htm
[^2]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/create-feature-dataset.htm
[^3]: https://pro.arcgis.com/en/pro-app/latest/arcpy/mapping/simplerenderer-class.htm
[^4]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/xy-table-to-point.htm
