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
* March, 2023 <br>
   - [x] #739
- [ ] https://github.com/octo-org/octo-repo/issues/740
- [ ] Add delight to the experience when all tasks are complete :tada:

<br><br>


> **Parameters & Inputs** <br>

<img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_parameters.JPG" width="400" height="500"> &nbsp; &nbsp; <img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_examplecsv.JPG" width="300" height="500"> <br>

* Workspace
1. First list item
   - First nested list item
     - Second nested list item
* Title Name
* Target Location CSV file
* Latitude Field
* Longitutde Field
* Buffer Distance
* Departure Time

 
