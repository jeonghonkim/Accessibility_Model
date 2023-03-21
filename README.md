# Accessibility_Model

`CBRE - Location Intelligence`
<br />Accessibility Model Ver 3.1
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
   - [X] Added the outputs to the current map and rendered the layers

* Ver.2 &nbsp;|&nbsp; December 2022 <br>
   - [X] Created new fields for accessibility score and traffic decays
   - [X] Captured traffics by routes with closest spatial-join

<br><br>


> **Parameters & Inputs** <br>

<img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_parameters.JPG" width="400" height="500"> &nbsp; &nbsp; <img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_examplecsv.JPG" width="300" height="500"> <br>

* *Workspace* &nbsp;|&nbsp; The current project's geodatabase is the default value
* *Title Name* &nbsp;|&nbsp; There should be no space in this parameter because it will be the name of new feature dataset in the workspace. 
* *Target CSV* &nbsp;|&nbsp; The csv should include three fields, which represent name, latitude value and logitude value for the target markets.
* *Latitude Field* &nbsp;|&nbsp; You can select the latitude field in the entered csv file. 
* *Longitutde Field* &nbsp;|&nbsp; You can select the longitude field in the entered csv file.
* *Buffer Distance*
* *Departure Time*
     - Second nested list item

 
[^1]: https://pro.arcgis.com/en/pro-app/latest/arcpy/network-analyst/closestfacility.htm
[^2]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/create-feature-dataset.htm
