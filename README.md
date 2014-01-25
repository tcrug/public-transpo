Introduction
===

A few years ago, I stumbled on [a question](http://skeptics.stackexchange.com/questions/4740/is-public-transport-less-fuel-efficient-than-cars) asking whether or not public transportation was less fuel-efficient than cars. It was quite intriguing, and coincided with a point in time when I'd been considering taking the bus to work. I dug around a bit and was able to find some data I thought could help answer the question.

[My answer at that time](http://skeptics.stackexchange.com/questions/4740/is-public-transport-less-fuel-efficient-than-cars/4750#4750) was done in Excel, simply by state averages. I thought a neat data challenge would be to look at the data in different ways and tease out any interesting findings:
- What are the characteristics of low-efficiency transit organizations?
- Is averaging by state the most accurate way to answer the question?
- My answer simply found the average efficiency of cars in the US. Is there a way to better represent passenger vehicle efficiency in a more complete sense?
- In what other ways could one represent the answer visually to tell a more complete story?
- Any peculiar findings/characteristics of the data?

Data Sources
===

*Public transportation data*

I've provided two data files in the repo, a .ods and .csv file.
- The .ods spreadsheet contains both raw and cleaned data from three tables contained in the 2009 data from the [National Transit Database](http://www.ntdprogram.gov/ntdprogram/data.htm) (NTD).
- The .csv is the result of my cleaning, merging, and re-naming of columns for an R-friendly version.

*Efficiency of automobiles*

Half of the question (though, the much harder half) is figuring out how efficient public transportation is. We also need some value to represent how efficient cars are. You can take a look at the Skeptics.Stackexchange answer linked above, where you'll find a table I pulled from the [Transportation Energy Data Book](http://cta.ornl.gov/data/spreadsheets.shtml), Table 2.12. The question also walks through how I backed out a figure of ~3,400 BTUs/passenger-mile for cars. The current version lists 3,364 BTUs/p-mile based on 2011 data.


Data Guide
===

To prepare the table from the NTD, I did the following:
- The tables, by default, contains one row per organization/mode type, followed by a row which calculates the total for this area. The rows calculating totals were removed.
- Tables `T17_Energy_Consumption.xls`, `T19_Op_Stats_Service.xls`, and `xC_Agency_Identification_By_State_ID.xls` were merged on unique combinations of the `ID`, `Org Type`, `Mode`, and `VOMS` columns.
- Any mis-matched entries or incomplete records were removed.
- The merged data was saved as a .csv file, and columns were re-named to be a bit more standard/friendly for R.

Here is a guide to the columns in the data set:

- `id`: agency ID
- `name`: agency name
- `city`: city name
- `state`: state name
- `uza`: From the [NTD glossary of terms](http://www.ntdprogram.gov/ntdprogram/Glossary.htm): "An area defined by the U. S. Census Bureau that includes one or more incorporated cities, villages, and towns (central place); and the adjacent densely settled surrounding territory (urban fringe) that together has a minimum of 50,000 persons."
- `population:` service area population
- `service_area_sq_mi`: the area served by a transit provider, square miles
- `org_type:` unsure
- `mode`: the primary mode of vehicle used for transportation
   - AG: automated guideway
   - AR: Alaska railroad
   - CC: cable car
   - CR: commuter rail, electric or diesel
   - DR: demand response, vans or buses that operate in response to on-demand requests from passengers
   - FB: ferry boat
   - HR: heavy volume rail, large capacity, rapid speed/acceleration
   - IP: inclined plain, a vehicle with right of way on steep grades, propelled by an attached cable
   - LR: lower capacity rails, operate on shared right of way roads with other vehicles, typically powerd by overhead power lines
   - MB: rubber tired passenter bus, uses fixed passenger routes for travel (same roads as everyone else)
   - MO: monorail
   - RB: rapid-transit bus, uses separate bus right-of-way routes during peak hours and features such as defined stations, traffic signal priority, or other features that emulate fixed railway systems
   - TB: trolley bus
   - VP: vanpool, where passengers meet at defined location and travel to the same destination
- `service`: type of service, direct operated (DO) or purchased transportation (PT)
- `voms:` vehicles operated in annual maxium service; the maximum number of vehicles put in service on the highest demand day of the year (excludes atypical days and/or special events)
- `diesel_gal`, `gas_gal`, ... `electric_battery_kwh`: fuel consumed by each transportation provider in the form or `type_quantity` where `gal` represents gallons and `kwh` represents kilowatt-hours.
- `annual_scheduled_vehicle_revenue_miles:` the total number of miles scheduled to be traveled by all vehicles, computed from the service route definitions. Excludes deviations, deadhead (travel to and from station and first/last stop, changing routes, etc.) and other anomalies.
- `annual_vehicle_miles:` the actual total miles traveled by all vehicles, including deviations, deadhead, etc.
- `annual_vehicle_hours:` total hours traveled by all vehicles, from the time they depart from the station until the time they return from scheduled service.
- `annual_vehicle_revenue_hours:` the number of hours the vehicles spend in actual revenue service (exludes maintenance, deadhead, etc.)
- `unlinked_passenger_trips`: the total number of passengers who board public transporation vehicles, regardless of how many separate vechicles are used from origin to destination (a bus trip where three separate lines are required would count as three).
- `btus_total`: a calculated column to convert fuel type/quantity to BTUs to unify the energy consumed by each agency. At the time, I was extracting these values from the 29th edition of the [Transportation Data Energy Book](http://cta.ornl.gov/data/index.shtml); they are now on the 32nd edition, and the analogous document I used can be found [here](http://cta.ornl.gov/data/tedb32/Edition32_Appendix_B.pdf). I had to manually get a value for each type, and multiply the corresponding column's volumes by that conversion factor to get BTUs for each row. You can take a look at the .ods spreadsheet for the calculation (`combined` spreadsheet, column AE).
- `btus_pmile:` a calculated column, BTUs divided by passenger-miles. From researching this question, this seems to be the preferred unit to evaluate efficiency. How many BTUs of energy, on average, does it require to take one passenger one mile.
- `primary_energy`: a calculated column. Some providers had fuel entries in more than one column. I wanted to take a look at using fuel type as a classifier, so I extracted the primary fuel by volume for each row.
- `average_occupancy`: a calculated column, passenger miles divided by vehicle miles. If we take the passenger mile total divided by the total miles traveled, we can extract the average number of passengers on each vehicle.