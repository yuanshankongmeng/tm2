MapBox Streets Vector Tiles
===========================

Overview
--------

Each vector tiles for each zoom level contains only the information that would be appropriate for rendering at that scale. For example, the `place_label` layer contains only major cities at zoom level 4, but contains all sizes of cities, towns, and villages at zoom level 10. Line and polygon information is generalized at lower zoom levels, not containing more detail than is necessary.

### `class` vs `type`

Most layers have a `class` or a `type` field to distinguish different types of objects in the layer. Values of `type` are transferred directly from an OpenStreetMap tag value. For example a value of `river` comes from the tag `waterway=river`. On the other hand, `class` values represent a generalized or derived classification to simplify styling. A value of `park` could come from one of many tags, such as `leisure=park`, `tourism=zoo`, or `boundary=national_park`.

### `osm_id`

Most objects have a `osm_id` number. These are derived from object IDs in OpenStreetMap database and are useful for identifying specific features. Because OpenStreetMap has three overlapping sets of ID number spaces, the `osm_id`s in the vector tiles have been modified to make them unique:

<!-- V2 -->
- IDs of nodes are multiplied by -1 (1 becomes -1)
- IDs of ways stay the same (1 stays 1)
- IDs of relations are increased by 10^12 (1 becomes 1000000000001)

<!-- V3, coming soon
- IDs of nodes are multiplied by -1 (1 becomes -1)
- IDs of ways that are lines stay the same (1 stays 1)
- IDs of ways that are polygons are increased by 10^12 (1 becomes 1000000000001)
- IDs of relations that are lines are increased by 2\*10^12  (1 becomes 2000000000001)
- IDs of relations that are polygons are increased by 3\*10^12 (1 becomes 3000000000001)
-->


Layers
------

### Area Layers

At lower zoom levels, small areas are not included in the vector tiles. As you zoom in you will see more and more appear.

If you are familiar with OpenStreetMap data, you will see that the landuse/landcover areas have been separated into a small number of very general classes

Water areas appear on top of the background and landuse areas. Islands are represented as holes in the water. Landuse areas such as parks that extend into water bodies will be covered by the water.

A small number of area classes that should appear on top of water are instead in the `landuse_overlay` layer that appears on top of water. The `land` class of this layer contains some potentially structural elements including piers and breakwaters.


### Road Layers

The road layers also include route lines that are not roads, such as foot paths, cycle paths, and railroads. There are three layers to ensure proper ordering: tunnels, bridges, and regular roads. A simple road style that does not handle tunnels or bridges in any special way still needs to take them into account, or there will be gaps in your roads.

    #road,
    #tunnel,
    #bridge {
      // road styles
    }

The road layer contains multiple geometry types: points, lines, and polygons. This is done to ensure proper ordering of fills and casing, but complicates styles slightly. Use the `'mapnik::geometry_type'` property to limit your styles for each type.

    ['mapnik::geometry_type'=1] { /* points */ }
    ['mapnik::geometry_type'=2] { /* lines */ }
    ['mapnik::geometry_type'=3] { /* polygons */ }


### Label Layers

Label layers help you style the labels for objects that have names.

The order of the layers represents the order in which they are rendered. Objects in layers at the top of the list will be drawn on top of objects in layers at the bottom. (This is why all the top layers are for labels and the bottom ones are for landuse/landcover areas.)

#### Areas

Labels for polygons are stored in the vector tiles as points. These layers will have an `area` field to tell you how big the polygon they represent is.

#### Extra attributes for styling

City points use some extra fields to allow for better styling at low zoom levels. The `scalerank` value is a subjective representation of cultural/cartographic significance from 1 (high significance) to 9. Many cities lack a scalerank value entirely and should be considered lowest on the significance scale. There is also a `ldir` field that can help set the initial label direction to try when using `text-placement-type: simple`. See the TileMill [Advanced Label Placement][1] guide for details.

[1]: http://mapbox.com/tilemill/docs/guides/labels-advanced/

Country labels also have a `scalerank` field that can be used to fit more labels on the map at low zoom levels by giving smaller countries a smaller text-size.

#### Limited subset

To save on file size, all possible labels are not included in each vector tile, but a limited number of the most important ones bases on what could and should be reasonably labeled. For example, a very short road might not be in the `road_label` layer because there wouldn't be room to label it anyway.

