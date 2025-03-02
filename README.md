# Accessibility_Model

<br />`Accessibility Model Ver.1`
<br />Author: Jeong Hoon Kim
<br />Completed as of March 21, 2023
<br><br>

> **Summary** <br>
 
This arcpy tool allows users to calculate drive times, miles, number of turn right and left, traffic volumes decays from starting points in major roads or streets to target destinations. It uses one csv file, which includes market name, latitude, and longitude fields, as an input. With the lat, long values, it locates market sites and creates buffers with user assgiend miles from the markets and starting points, which were the intersections between marekts and buffers. Also this tool creates routes and directions and allows users to customize departure time, wihch will affect the drive time. After running this tool, it adds four feature layers, which were markets, starts, buffers and routes, to the current map in a project, and the markets and routes layers have statistics tables.

<br><br>

> **Updates** <br>

* Ver.1 &nbsp;|&nbsp; May - August 2022 <br>
   - [X] Created a point feature layer with a csv file with lat, long values[^1]
   - [X] Created a buffer with a customized distance and linear unit[^2]
   - [X] Added workspace as a parameter for different environment settings[^3]
   - [X] Created routes with ClosestFacility in na module[^4]

* Ver.2 &nbsp;|&nbsp; December 2022 <br>
   - [X] Added the messages after each steps[^5]
   - [X] Captured traffics by routes with closest spatial-join[^6]
   - [X] Created new fields for accessibility score and traffic decays[^7]

* Ver.3 &nbsp;|&nbsp; March 2023 <br>
   - [x] Added title name parameters to save outputs in different feature datasets[^8]
   - [x] Created directions with Closest Facility nax module, instead of na module[^9]
   - [X] Created new definitions to count number of turn right and left
   - [X] Added the outputs to the current map[^10]
   - [X] Rendered imported feature layers[^11]

<br><br>


> **Parameters & Inputs** <br>

<img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_parameters.JPG" width="400" height="500"> &nbsp; &nbsp; 
<img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_examplecsv.JPG" width="300" height="500"> <br>

* *Workspace* &nbsp;|&nbsp; The current project's geodatabase is the default value
* *Title Name* &nbsp;|&nbsp; There should be no space in this parameter because it will be the name of new feature dataset in the workspace. 
* *Target CSV* &nbsp;|&nbsp; The csv should include three fields, which represent name, latitude value and logitude value for the target markets.
* *Latitude Field* &nbsp;|&nbsp; You can select the latitude field in the entered csv file. 
* *Longitutde Field* &nbsp;|&nbsp; You can select the longitude field in the entered csv file.
* *Buffer Distance* &nbsp;|&nbsp; You can enter the mile-distance from market locations as you wish.
* *Departure Time* &nbsp;|&nbsp; You can enter exact time will generate drive times. The currnet time will be applied if you select the clock symbol.


<br><br>


> **Outputs** <br>

<img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_testoutputs.JPG" width="400" height="400"> &nbsp; &nbsp;
<img src="https://github.com/jeonghonkim/Accessibility_Model/blob/main/access_testoutputsmap.JPG" width="400" height="400"> <br>

* *_AllStreets* &nbsp;|&nbsp; The street points are generated as starting points from intersections between the buffers and streets.
* *_Buffers* &nbsp;|&nbsp; The buffer features are generated by the market locations and the buffere distance you entered.
* *_Directions* &nbsp;|&nbsp; The directions are from the routes after solving Closest-Facility analysis with '_AllStreets' as incidents and '_Markets_Original' as facilities. The number of turn right and left of each routes can be calculated by the direction.
* *_Markets_Original* &nbsp;|&nbsp; The point feature class is generated by the entered csv file. It shows only the locations.
* *_Markets_Summary* &nbsp;|&nbsp; This feature class has a summary statistic table including number of starting points, drive time, drive miles, number of turn right and left, and traffics.
* *_Routes_Original* &nbsp;|&nbsp; The lines are the routes from 'AllStreets' to '_Markets_Original'.
* *_Routes_TrafficTurns* &nbsp;|&nbsp; This feeature class includes each route's drive time, drive mile, number of turn right and left, and traffics.


<br><br>


> **Detailed Steps** <br>


<br>**1. Create analysis objects** <br><br>
&nbsp;&nbsp;&nbsp;*1) Feature dataset*<br>
&nbsp;&nbsp;&nbsp;Creating a feature dataset has multiple benefits. First, you can run the analysis multiple times with different feature names. Also, you can easily find the feature analysis outcomes in different feature dataset. The project title name you enter would be the feature datasetanme.<br><br>
&nbsp;&nbsp;&nbsp;*2) Markets and buffers and starting road points*<br>
&nbsp;&nbsp;&nbsp;Once you run the tool after entering the prepared csv file with name, latitude and longitude as well as the buffer-mile distance, it will create market locations as well as buffers feature layers. They will be saved in the generated feature dataset. Starting points are the intersection between the assigned street or roads layers and the generated buffers. You can change the 'major_roads' path in the code as you wish, such as all streets or major roads. All streets in US are used in this analysis.<br><br>
&nbsp;&nbsp;&nbsp;*3) Multipart to single part*<br>
&nbsp;&nbsp;&nbsp;The generated starting points can be a multipart feature point layer. In order to make all points as unique starts, use the multipart to single part geoprocessing tool.<br><br>

```python
    # 1. Create analysis objects
    # 1) Feature Dataset
    arcpy.management.CreateFeatureDataset(workspace, title_name, out_coordinate_system)
    accessbility_fd = os.path.join(workspace, title_name)

    # 2) Markets, buffers, and starting road points
    # Markets
    market_p = os.path.join(accessbility_fd, title_name+"_Markets_Original")
    arcpy.management.XYTableToPoint(trgt_csv, market_p, long_field, lat_field, out_coordinate_system)
    # Buffers
    buffer = os.path.join(accessbility_fd, title_name+"_Buffers")
    arcpy.Buffer_analysis(market_p, buffer, buff_dis) 
    # Starting road points
    major_roads = "Streets"
    start_multi = os.path.join(accessbility_fd, title_name+"_Start_Multi")        
    arcpy.Intersect_analysis([buffer, major_roads], start_multi, "", "", "point")

    # 3) Multipart to singlepart
    start_p = os.path.join(accessbility_fd, title_name+"_AllStreets")
    arcpy.MultipartToSinglepart_management(start_multi, start_p)
    arcpy.management.Delete(start_multi)
```
<br>

**2. Generate routes and directions** <br><br>
&nbsp;&nbsp;&nbsp;*1) Use Closest-Facility nax module in Network Analysis*<br>
&nbsp;&nbsp;&nbsp;Creating a closest-facility layer with nax module to get routes and directions from starting points to markets. This tool's network datasource is ArcGIS Online, which means that it will consume credits if you run this tool. You can use this tool without credit consumption if you have your own network dataset and change the datapath to the nds in your local drive. The analysis outcomes include drive time in minutes and drive distances in miles. Also the drive times is calculated by the entered time as a departure time. Because the nax module is used in this analysis, the desired output features, which were routes and directions, in the CloesestFacility objects are exported to the created feature dataset. Although nax moudle is much faster than na, please reference the following Esri's article showing differences between na and nax modules as well as the cases you need na module.[^12]<br><br>
&nbsp;&nbsp;&nbsp;*2) Filter the routes heading different markets*<br>
&nbsp;&nbsp;&nbsp;Since this tool's one of the most significant purpose is calculating accessibility of many differnt markets 'at once', it use Closest-Facility to meausre the values. However, if some of markets locate closely each other, interruptions might happnes. For example, some starting points generated at Market-1 can heads to different markets. Most of the cases happen when the points locate in highways or roads without u-turn. By filtering feature with different market-name and route-name, those outliers can be removed.<br><br>

```python
    # 2. Generate routes and directions
    # 1) Use Closest-Facility nax module
    # Set NETWORKDATASET object variables
    input_facilities = market_p
    input_incidents = start_p
    output_routes = os.path.join(accessbility_fd, title_name+"_Routes_Original")
    output_directions = os.path.join(accessbility_fd, title_name+"_Directions")
        
    # Instantiate a ClosestFacility solver object using esri source - consume credits!
    closest_facility = arcpy.nax.ClosestFacility("https://www.arcgis.com/")        
    nd_travel_modes = arcpy.nax.GetTravelModes("https://www.arcgis.com/")
    travel_mode = nd_travel_modes["Driving Time"]

    # Set ClosestFacility properties
    closest_facility.travelMode = travel_mode
    closest_facility.timeUnits = arcpy.nax.TimeUnits.Minutes
    closest_facility.defaultTargetFacilityCount = 1
    closest_facility.routeShapeType = arcpy.nax.RouteShapeType.TrueShapeWithMeasures
    closest_facility.returnDirections = True
    closest_facility.timeOfDay = datetime.datetime.strptime(depart_time, '%m/%d/%Y %I:%M:%S %p')
    closest_facility.timeOfDayUsage = arcpy.nax.TimeOfDayUsage.DepartureTime
    closest_facility.timeZone = arcpy.nax.TimeZoneUsage.UTC
    closest_facility.travelDirection = arcpy.nax.TravelDirection.ToFacility
    closest_facility.distanceUnits = arcpy.nax.DistanceUnits.Miles
    closest_facility.directionsDistanceUnits = arcpy.nax.DistanceUnits.Miles
        
    # Load Closest Facility inputs
    closest_facility.load(arcpy.nax.ClosestFacilityInputDataType.Facilities, input_facilities)
    closest_facility.load(arcpy.nax.ClosestFacilityInputDataType.Incidents, input_incidents)
        
    # Solve the analysis
    result = closest_facility.solve()
        
    # Export the routes and directions to feature classes
    if result.solveSucceeded:
        result.export(arcpy.nax.ClosestFacilityOutputDataType.Routes, output_routes)
        result.export(arcpy.nax.ClosestFacilityOutputDataType.Directions, output_directions)
    else:
        print("Solve failed")
        print(result.solverMessages(arcpy.nax.MessageSeverity.All))
        
    # 2) Filter the routes heading different markets
    # Add fields and split 'Name' field with Markets(Facility) and Roads(Incident)
    arcpy.AddField_management(output_routes, "Name_MarketID", "TEXT")
    arcpy.AddField_management(output_routes, "Name_RoadID", "TEXT")
    with arcpy.da.UpdateCursor(output_routes, ["Name", "Name_RoadID", "Name_MarketID"]) as cursor:
        for row in cursor:
            row[1] = row[0].split(" - ")[0]
            row[2] = row[0].split(" - ")[1]
            cursor.updateRow(row)
        
    # Select routes having same 'Name_RoadID' and 'Name_MarketID' to filter incidents falling into other markets
    routes_selected = arcpy.management.SelectLayerByAttribute(output_routes, 'NEW_SELECTION', 'Name_RoadID = Name_MarketID')
    routes_filtered = os.path.join(accessbility_fd, title_name+"_Routes_Filtered")
    arcpy.management.CopyFeatures(routes_selected, routes_filtered)
```
<br>

**3. Capture traffics** <br><br>
&nbsp;&nbsp;&nbsp;*1) Spatial join with traffic points*<br>
&nbsp;&nbsp;&nbsp; After filtering the routes, spatial-join is used to mesaure the closest traffics of each routes. Many different ways have been developed to capture traffic counts. For example, street polygons can be used to capture the traffics within 1 mile (You can see in the detailed scripts in the above list). Also, you can develop and customize this part as you wish.<br><br>
&nbsp;&nbsp;&nbsp;*2) Calculate traffic fields*<br>
&nbsp;&nbsp;&nbsp; The capcuated traffics can be normalized with the following two calculations, which are 'Traffic Decays' and 'Traffic Scores'. <br>
* **Traffic Decays**&nbsp;&nbsp;|&nbsp;&nbsp;Closest Traffics / Drive Times<br> This value is the number of traffics divided by drive times. Higher values represents that there are more traffics in the route with the same amount of drive time. Higher traffic decays can be interpreted as less accessibility in this model.
* **Traffic Scores**&nbsp;&nbsp;|&nbsp;&nbsp;(1 / Drive Miles) + (Closest Traffics * α)<br> Traffic scores can be achieved with drive miles and closest traffics. The values can be used in the cases that higher traffics are positive signals. In this analysis, 0.2 is used for the α value, but you can increase the value, such as 0.5  or 0.7, if the impact of the traffics is more forceful.<br><br>

```python
    # 3. Capture cloesest traffics and calculate traffic decays (traffics devided by drive times) by routes
    # 1) Spatial join with traffic points 
    sl_traffics = "Traffics"
    routes_traffics = os.path.join(accessbility_fd, title_name+"_Routes_Traffics")
    arcpy.analysis.SpatialJoin(routes_filtered, sl_traffics, routes_traffics, "JOIN_ONE_TO_ONE", "KEEP_ALL", "", "CLOSEST_GEODESIC", "", "")
    arcpy.management.Delete(routes_filtered)
    
    # 2) Calculate traffics fields
    # Create 'Traffic Decays' and 'Traffic Scores'
    arcpy.AddField_management(routes_traffics, "Traffic_Decays", "DOUBLE")
    arcpy.AddField_management(routes_traffics, "Traffic_Scores", "DOUBLE")
    arcpy.management.CalculateField(routes_traffics, "Traffic_Decays", "!TRAFFIC1! / !Total_Minutes!", "PYTHON3")
    arcpy.management.CalculateField(routes_traffics, "Traffic_Scores", "(1 / !Total_Miles!) + (!TRAFFIC1! * 0.2)", "PYTHON3")
```
<br>

**4. Calculate number of turns by routes**<br><br>
&nbsp;&nbsp;&nbsp;*1) Convert feature class table to datframe in pandas*<br>
&nbsp;&nbsp;&nbsp; There are several steps you need to convert feature class table to dataframe in pandas. Arcpy supports TableToNumPyArray in da module[^13] and it works for only gdb tables, not for feature class tables. Therefore, the first step is to transform feature class table to gdb table, and TableToNumPyArray can be used to import it into dataframe. The follwing codes show two feature class, which were routes and directions, are changed to dataframes. Also, please reference the following link sharing the functions to convert feature tables to dataframes[^14].<br><br>
&nbsp;&nbsp;&nbsp;*2) Create functions to count turns by each route*<br>
&nbsp;&nbsp;&nbsp; The number of turn right and left can be calculated from the directino dataframe. Since directinot dataframe does not have a route id column, it needs to be created. Although it can be achieved by many different ways, we are going to use Type column in this analysis. The direction table's order is organized by the route id and 18 indicates the start of each route. By using the features, we can get incremental values in the route id column. <br>
&nbsp;&nbsp;&nbsp; Once we differentiate route ids, functions to count turn right and left need to be created. There are three types of sentences indicating turns in the direction dataframe, which were 'turn left/right', 'bear left/right', and 'sharp left/right'. The first two types need to be differentiated by upper and lower characeters since they can be located in the start of the sentence. Also, there are sentences indicating turn two times. For the left turn, there are two types, which are 'U-turn' and 'Turn left, then turn left'. There is only one type for the right, which is 'Turn right, then turn right'. You can achieve 2 turns without interruptions by covering if you use the 2 turns fuctions after you apply the 1 turn function.<br><br>
&nbsp;&nbsp;&nbsp;*3) Convert df to table & join it to feature class*<br>
&nbsp;&nbsp;&nbsp; A summary dataframe of nubmers of turn right and left can be created from the above fuctions. Once it is created, the dataframe can be converted into a table in geodatabase in Arcgis Pro with the reverse way, 'NumPyArrayToTable'. It can be joined with the routes feature class with the route id. <br><br>

```python
    # 4. Calculate number of turn by routes
    # 1) Convert feature class table to datframe in pandas
    # Convert feature tables to gdb tables in workspace
    arcpy.TableToTable_conversion(routes_traffics, workspace, title_name+"_RoutesTraf_Table")
    arcpy.TableToTable_conversion(output_directions, workspace, title_name+"_Directions_Table")
    
    # Convert route and direction tables to dataframes in pandas
    routesTraf_table = os.path.join(workspace, title_name+"_RoutesTraf_Table")
    routesTraf_field = [f.name for f in arcpy.ListFields(routesTraf_table)]
    routesTraf_array = arcpy.da.TableToNumPyArray(routesTraf_table, routesTraf_field)
    routesTraf_df = pd.DataFrame(routesTraf_array, columns = routesTraf_field)
    
    directions_table = os.path.join(workspace, title_name+"_Directions_Table")
    directions_field = [f.name for f in arcpy.ListFields(directions_table)]
    directions_array = arcpy.da.TableToNumPyArray(directions_table, directions_field)
    directions_df = pd.DataFrame(directions_array, columns = directions_field)
    
    # 2) Create functions to count turns by each route
    # Create 'Route ID' column
    directions_df['Type'] = directions_df['Type'].astype(str)
    def route_id(row):
        if '18' in row['Type']: # type '18' indicates start of each route
            val = 1
        else:
            val = 0
        return val
    directions_df['Route_ID'] = directions_df.apply(route_id, axis=1)
    directions_df['Route_ID'] = directions_df['Route_ID'].cumsum()
    
    # Filter directions with 'Route ID'
    route_filtered_id = routesTraf_df['IncidentOID'].values.tolist()
    directions_filtered_df = directions_df[directions_df["Route_ID"].isin(route_filtered_id)]
    
    # Create 'Turn Right' and 'Turn Left' columns in direction dataframe
    # Create definition to create the columns
    def turn_left1(row):
        if 'turn left' in row['Text']:
            val = 1
        elif 'Turn left' in row['Text']:
            val = 1
        elif 'sharp left' in row['Text']:
            val = 1
        elif 'Bear left' in row['Text']:
            val = 1
        elif 'bear left' in row['Text']:
            val = 1
        else:
            val = 0
        return val
    
    def turn_left2(row):
        if 'U-turn' in row['Text']:
            val = 2
        elif 'Turn left, then turn left' in row['Text']:
            val = 2
        else:
            val = row['Turn_Left']
        return val
    
    def turn_right1(row):
        if 'turn right' in row['Text']:
            val = 1
        elif 'Turn right' in row['Text']:
            val = 1
        elif 'sharp right' in row['Text']:
            val = 1
        elif 'Bear right' in row['Text']:
            val = 1
        elif 'bear right' in row['Text']:
            val = 1
        else:
            val = 0
        return val
    
    def turn_right2(row):
        if 'Turn right, then turn right' in row['Text']:
            val = 2
        else:
            val = row['Turn_Right']
        return val
    
    directions_df['Turn_Left'] = directions_df.apply(turn_left1, axis=1)
    directions_df['Turn_Left'] = directions_df.apply(turn_left2, axis=1)
    directions_df['Turn_Right'] = directions_df.apply(turn_right1, axis=1)
    directions_df['Turn_Right'] = directions_df.apply(turn_right2, axis=1)
    
    # Create a summary dataframe of turns by 'Route ID'
    summary_fields = ['Turn_Left', 'Turn_Right']
    routesTurn_df = directions_df.groupby(['Route_ID'], as_index=False)[summary_fields].sum()
    
    # Convert dataframe to gdb table
    routesTurn_records = routesTurn_df.to_records(index=False)
    routesTurn_array = np.array(routesTurn_records, dtype = routesTurn_records.dtype.descr)
    routesTurn_table = os.path.join(workspace, title_name+"_RoutesTurn_Table")
    arcpy.da.NumPyArrayToTable(routesTurn_array, routesTurn_table)
    
    # Join gdb table to feature class
    route_joined_fc = arcpy.management.AddJoin(routes_traffics, "IncidentOID", routesTurn_table, "Route_ID", "KEEP_COMMON")
    routes_traffics_turns = os.path.join(accessbility_fd, title_name+"_Routes_TrafficsTurns")
    arcpy.management.CopyFeatures(route_joined_fc, routes_traffics_turns)
```
<br>

**5. Generate Summary Statistics**<br><br>
&nbsp;&nbsp;&nbsp;*1) Create summary statistics by each markets*<br>
&nbsp;&nbsp;&nbsp; The summary statistics by market locations can be generated by using 'Summary Statistics' in arcpy. Aftrer get the field names from routes, you can get mean, maximum, minimum, range, and standard deviation of each values. The created statistic table can be joined to the original market feature class, so that the location and summaries can be displayed together on the map.
<br><br>

```python
    # 5. Generate summary statistics
    # 1) Get the field names and create summary statistics table by markets
    # Get field names
    field_mile = [f.name for f in arcpy.ListFields(routes_traffics_turns, "*Total_Miles*")][0]
    field_minute = [f.name for f in arcpy.ListFields(routes_traffics_turns, "*Total_Minutes*")][0]
    field_right = [f.name for f in arcpy.ListFields(routes_traffics_turns, "*Turn_Right*")][0]
    field_left = [f.name for f in arcpy.ListFields(routes_traffics_turns, "*Turn_Left*")][0]
    field_trafficD = [f.name for f in arcpy.ListFields(routes_traffics_turns, "*Traffic_Decays*")][0]
    field_trafficS = [f.name for f in arcpy.ListFields(routes_traffics_turns, "*Traffic_Scores*")][0]
    field_case = [f.name for f in arcpy.ListFields(routes_traffics_turns, "*FacilityOID*")][0]
    
    # Create summary statistics table
    access_summary_table = os.path.join(workspace, title_name+"_AccessSummary_Table")
    statistic_fields = [[field_mile, "MEAN"], [field_mile, "MAX"], [field_mile, "MIN"], [field_mile, "RANGE"], [field_mile, "STD"],
                        [field_minute, "MEAN"], [field_minute, "MAX"], [field_minute, "MIN"], [field_minute, "RANGE"], [field_minute, "STD"],
                        [field_right, "MEAN"], [field_right, "MAX"], [field_right, "MIN"], [field_right, "RANGE"], [field_right, "STD"],
                        [field_left, "MEAN"], [field_left, "MAX"], [field_left, "MIN"], [field_left, "RANGE"], [field_left, "STD"],
                        [field_trafficD, "MEAN"], [field_trafficD, "MAX"], [field_trafficD, "MIN"], [field_trafficD, "RANGE"], [field_trafficD, "STD"],
                        [field_trafficS, "MEAN"], [field_trafficS, "MAX"], [field_trafficS, "MIN"], [field_trafficS, "RANGE"], [field_trafficS, "STD"]]
    case_field = field_case
    arcpy.analysis.Statistics(routes_traffics_turns, access_summary_table, statistic_fields, case_field)
    
    # 2) Join the summary table to original market feature class
    market_summary = os.path.join(accessbility_fd, title_name+"_Markets_Summary")
    market_joined_fc = arcpy.management.AddJoin(market_p, "OBJECTID", access_summary_table, field_case, "KEEP_ALL")
    arcpy.management.CopyFeatures(market_joined_fc, market_summary)
```
<br><br>


> **Additional Notes** <br>

Please reference the attached accessibility arcpy jupyter notebooks.

<br>

[^1]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/xy-table-to-point.htm
[^2]: https://pro.arcgis.com/en/pro-app/2.8/tool-reference/analysis/buffer.htm
[^3]: https://pro.arcgis.com/en/pro-app/2.8/arcpy/functions/getparameterastext.htm
[^4]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/network-analyst/make-closest-facility-analysis-layer.htm
[^5]: https://pro.arcgis.com/en/pro-app/2.8/arcpy/functions/addmessage.htm
[^6]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/analysis/spatial-join.htm
[^7]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/add-fields.htm
[^8]: https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/create-feature-dataset.htm
[^9]: https://pro.arcgis.com/en/pro-app/latest/arcpy/network-analyst/closestfacility.htm
[^10]: https://pro.arcgis.com/en/pro-app/latest/arcpy/mapping/map-class.htm
[^11]: https://pro.arcgis.com/en/pro-app/latest/arcpy/mapping/simplerenderer-class.htm
[^12]: https://pro.arcgis.com/en/pro-app/latest/arcpy/network-analyst/choosing-between-the-two-modules-arcpy-nax-versus-arcpy-na-.htm
[^13]: https://pro.arcgis.com/en/pro-app/latest/arcpy/data-access/what-is-the-data-access-module-.htm](https://pro.arcgis.com/en/pro-app/latest/arcpy/data-access/tabletonumpyarray.htm
[^14]: https://gist.github.com/d-wasserman/e9c98be1d0caebc2935afecf0ba239a0
