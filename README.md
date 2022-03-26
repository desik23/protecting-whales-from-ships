# Protecting Whales From Ships in Dominica

Desik Somasundaram and Hanna Weyland 
2021-11-16
protecting-whales-from-ships-13 created by GitHub Classroom

## Overview

In this project, we created a method to determine the boundary of a speed reduction zone and its effect on the local marine traffic in Dominica, a small island nation in the Caribbean. The speed reduction zone encompasses the preferred habitat of the local sperm whales, which was approximated using areas of frequent whale sightings. It was concluded that the speed reduction zone should be located off of the west coast of Dominica. The total amount of **increased travel time** for the 166,255 ships passing through this zone would be approximately **28.7 days** for a 10kts reduced speed zone in the identified habitat. 

## Data and Methods 

### Libraries 
Import appropriate libraries:
geopandas 
numpy
pandas
matplotlib.image
matplotlib.pyplot
os
glob

### Dominica and Whale Sighting Data

Data for Dominica's country outline was imported and reprojected into Dominica 1945 / British West Indies Grid. Whale sightings was also imported using GeoPandas but needed to be bootstrapped since there was no associated geometry set to this file. Once the data was converted into the correct geometries and projected into the local CRS, point observations were converted into habitat regions. 

### Creating a Grid 

A sampling grid was created to determine the number of sightings in each grid cell. A bounding box was made for the whale sighting and a 2km x 2km grid was created within the box. The number of cells in the grid was determined by dividing the (x and y) dimensions of the bounding box by the (horizontal and vertical) cell size, rounding up to the nearest integer to include the full region. Then we used [numpy.arange()] to produce an evenly spaced range of values in x and y.

Using the following function, we converted the corner points into a cell polygon. 

[def make_cell(x, y, cell_size):
    ring = [
        (x, y),
        (x + cell_size, y),
        (x + cell_size, y + cell_size),
        (x, y + cell_size)
    ] [cell = shapely.geometry.Polygon(ring)
    return cell]
    
Now we can iterate over each combination of x and y coordinates in two nested for-loops:

[cells = []
for x in xs:
    for y in ys:
        cell = make_cell(x, y, cell_size)
        cells.append(cell)]

Finally, we created a grid GeoDataFrame containing a cell for each row.

[grid = geopandas.GeoDataFrame({'geometry': cells}, crs=2002)]

### Extract Whale habitat

Next, we spatially joined the grid to our sighting data in order to count the number of sightings in each cell using an inner join. We used a *map-reduce* operation to count the sightings in each cell by grouping the rows of the joined dataset ([group_by()]) by the dataframe index, then reducing each group ([count()]). Next, we subsetted the grid dataframe to cells that contain more than 20 sightings. Then we can extract the unary union of this subset and used the convex hull of this union as our speed reduction zone.

### Vessel Data 

Vessel data also had to be bootstrapped similarly to whale sightings data. We then spatially subset the data to vessels that are within our habitat region. We calculated the vessel's speed by sorting our dataframe by MMSI (Maritime Mobile Service Identity: the vessel’s unique identifier and time using [sort_values()]. Then, we created a copy of our dataframe and shifted each observation down one row using [shift()]. Then, we joined the original dataframe with our shifted copy using [join()]. Lastly, we set the geometeries with the [set_geometeries()] function. 

### Final Calulations and results

We calculated the following statistics: 

The distance for each observation to the next. 
The time difference between each observation to the next.
The average speed between each observation to the next.
The time that the distance would have taken at 10 knots.
The difference between the time that it actually took and how much it would have taken at 10 knots.
Impact on shipping of a 10-knot reduced-speed zone in our identified whale habitat.

#### Results 

A 10-knot reduction in speed in our identified whale habitat area would add approximately 28 days to the 166,255 ships that travel through this zone. Although not all ships will not be affected by this reduction zone, the zone would prove beneficial to the whales in this area as well as the whale watching business. 

