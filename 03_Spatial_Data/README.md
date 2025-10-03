<img src="https://art.apx.com/images/ART.png" width="500"/>

[![LinkedIn Badge](https://img.shields.io/badge/Project-Profile-blue)](https://art.apx.com/mymodule/reg/TabDocuments.asp?r=111&ad=Prpt&act=update&type=PRO&aProj=pub&tablename=doc&id1=109) [![Pubs Badge](https://img.shields.io/badge/Project-Pubs-critical)](https://orcid.org/my-orcid?orcid=0000-0002-1792-0351) [![Twitter Badge](https://img.shields.io/badge/Project-Tweets-critical?color=blue)](https://x.com/) [![Program Badge](https://img.shields.io/badge/Project-Steward-critical)](https://www.ambiente.gob.ec/) [![Annexes Badge](https://img.shields.io/badge/Submission-Annexes-critical?color=blue)](https://nextcloud.ambiente.gob.ec)

------------------------------------------------------------------------

# Task 3 Nesting Framework Analysis

#### *Development and Validation of Spatial Covariates for Inputs to Deforestation Risk Modelling*

-   [Introduction](#introduction)
-   [Import data](#import-data)
    -   [AOI Jurisdiction](#aoi-jurisdiction)
    -   [Built Environment](#built-environment)
    -   [Transportation](#transportation)
    -   [Hydrography](#hydrography)
    -   [Topography](#topography)
-   [Risk-allocated deforestation](#risk-allocated-deforestation)
-   [Stratification of deforestation pixels](#visualize-deforestation-allocation)

### Introduction 

The following spatial covariates were imported as potential drivers of deforestation risk. Covariates were merged between demographic and geographic datasets surrounding the project area and national level datasets beyond the project area in order to enable jurisdictional analysis.

------------------------------------------------------------------------

### Import data 

#### AOI Jurisdiction {#aoi-jurisdiction}

``` r
library(geodata)
library(tmaptools)
library(tmap)
library(sf)

# AOI Jurisdictional Bdry 
aoi_country = geodata::gadm(country="ECU", level=0, path="./03_Spatial_Data/AOI") |> sf::st_as_sf()
aoi_states  = geodata::gadm(country="ECU", level=1, path="./03_Spatial_Data/AOI") |> sf::st_as_sf()
st_write(aoi_country, "./03_Spatial_Data/AOI/aoi_country.shp", delete_dsn = T)
st_write(aoi_states, "./03_Spatial_Data/AOI/aoi_states.shp", delete_dsn = T)
```

```         
Deleting source `./03_Spatial_Data/AOI/aoi_country.shp' using driver `ESRI Shapefile'
Writing layer `aoi_country' to data source 
  `./03_Spatial_Data/AOI/aoi_country.shp' using driver `ESRI Shapefile'
Writing 1 features with 2 fields and geometry type Multi Polygon.
```

```         
Deleting source `./03_Spatial_Data/AOI/aoi_states.shp' using driver `ESRI Shapefile'
Writing layer `aoi_states' to data source 
  `./03_Spatial_Data/AOI/aoi_states.shp' using driver `ESRI Shapefile'
Writing 24 features with 11 fields and geometry type Unknown (any).
```

``` r
crs_master = sf::st_crs(aoi_country)

# Visualize
tmap::tmap_mode("plot")
tmap::tm_shape(aoi_country) + tmap::tm_borders(col="purple", lwd=2) +
  tmap::tm_shape(aoi_states) + tmap::tm_borders(col="purple", lwd=.5) +
  tmap::tm_scalebar(position=c("RIGHT", "BOTTOM"), text.size = .5) +
  tmap::tm_compass(color.dark="gray60",text.color="gray60",position=c("RIGHT", "top")) +
  tmap::tm_graticules(lines=T,labels.rot=c(0,90),lwd=0.2) +
  tmap::tm_layout(legend.position=c("left", "top"), legend.bg.color = "white") +
  tmap::tm_title("Risk Covariates Map: AOI Jurisdictional Boundaries", size=.8) + 
  tmap::tm_basemap("OpenStreetMap")
```

![](MAPS/Map_RiskCovariateScopingA_202508.png)

------------------------------------------------------------------------

#### Built Environment 

``` r
library(osmdata)
library(httr2)
library(ggmap)

# Check OSM data & derive bbox window
osmdata::available_features()  
```

```         
  [1] "4wd_only"                    "abandoned"                  
  [3] "abutters"                    "access"                     
  [5] "addr"                        "addr:*"                     
  [7] "addr:city"                   "addr:conscriptionnumber"    
  [9] "addr:country"                "addr:county"                
 [11] "addr:district"               "addr:flats"                 
 [13] "addr:full"                   "addr:hamlet"                
 [15] "addr:housename"              "addr:housenumber"           
 [17] "addr:inclusion"              "addr:interpolation"         
 [19] "addr:place"                  "addr:postbox"               
 [21] "addr:postcode"               "addr:province"              
 [23] "addr:state"                  "addr:street"                
 [25] "addr:subdistrict"            "addr:suburb"                
 [27] "addr:unit"                   "admin_level"                
 [29] "aeroway"                     "agricultural"               
 [31] "alcohol"                     "alt_name"                   
 [33] "amenity"                     "area"                       
 [35] "atv"                         "backward"                   
 [37] "barrier"                     "basin"                      
 [39] "bdouble"                     "bicycle"                    
 [41] "bicycle_road"                "biergarten"                 
 [43] "boat"                        "border_type"                
 [45] "boundary"                    "brand"                      
 [47] "bridge"                      "building"                   
 [49] "building:architecture"       "building:fireproof"         
 [51] "building:flats"              "building:levels"            
 [53] "building:material"           "building:min_level"         
 [55] "building:part"               "building:soft_storey"       
 [57] "bus"                         "bus_bay"                    
 [59] "bus:lanes"                   "busway"                     
 [61] "capacity"                    "carriage"                   
 [63] "castle_type"                 "change"                     
 [65] "charge"                      "clothes"                    
 [67] "construction"                "construction_date"          
 [69] "construction#Railways"       "covered"                    
 [71] "craft"                       "crossing"                   
 [73] "crossing:island"             "cuisine"                    
 [75] "cutting"                     "cycle_rickshaw"             
 [77] "cycleway"                    "cycleway:left"              
 [79] "cycleway:left:oneway"        "cycleway:right"             
 [81] "cycleway:right:oneway"       "denomination"               
 [83] "destination"                 "diet:*"                     
 [85] "direction"                   "dispensing"                 
 [87] "disused"                     "dog"                        
 [89] "drinking_water"              "drinking_water:legal"       
 [91] "drive_in"                    "drive_through"              
 [93] "ele"                         "electric_bicycle"           
 [95] "electrified"                 "embankment"                 
 [97] "embedded_rails"              "emergency"                  
 [99] "end_date"                    "entrance"                   
[101] "est_width"                   "fee"                        
[103] "female"                      "fire_object:type"           
[105] "fire_operator"               "fire_rank"                  
[107] "food"                        "foot"                       
[109] "footway"                     "ford"                       
[111] "forestry"                    "forward"                    
[113] "frequency"                   "frontage_road"              
[115] "fuel"                        "full_name"                  
[117] "gauge"                       "gender_segregated"          
[119] "golf_cart"                   "goods"                      
[121] "hand_cart"                   "hazard"                     
[123] "hazmat"                      "healthcare"                 
[125] "healthcare:counselling"      "healthcare:speciality"      
[127] "height"                      "hgv"                        
[129] "highway"                     "historic"                   
[131] "horse"                       "hot_water"                  
[133] "hov"                         "ice_road"                   
[135] "incline"                     "industrial"                 
[137] "inline_skates"               "inscription"                
[139] "int_name"                    "internet_access"            
[141] "junction"                    "kerb"                       
[143] "landuse"                     "lane_markings"              
[145] "lanes"                       "lanes:bus"                  
[147] "lanes:psv"                   "layer"                      
[149] "leaf_cycle"                  "leaf_type"                  
[151] "leisure"                     "lhv"                        
[153] "lit"                         "loc_name"                   
[155] "location"                    "male"                       
[157] "man_made"                    "max_age"                    
[159] "max_level"                   "maxaxleload"                
[161] "maxheight"                   "maxlength"                  
[163] "maxspeed"                    "maxstay"                    
[165] "maxweight"                   "maxwidth"                   
[167] "military"                    "min_age"                    
[169] "min_level"                   "minspeed"                   
[171] "mofa"                        "moped"                      
[173] "motor_vehicle"               "motorboat"                  
[175] "motorcar"                    "motorcycle"                 
[177] "motorroad"                   "mountain_pass"              
[179] "mtb:description"             "mtb:scale"                  
[181] "name"                        "name_1"                     
[183] "name_2"                      "name:left"                  
[185] "name:right"                  "narrow"                     
[187] "nat_name"                    "natural"                    
[189] "nickname"                    "noexit"                     
[191] "non_existent_levels"         "nudism"                     
[193] "office"                      "official_name"              
[195] "old_name"                    "oneway"                     
[197] "oneway:bicycle"              "oneway:bus"                 
[199] "openfire"                    "opening_hours"              
[201] "opening_hours:drive_through" "operator"                   
[203] "orientation"                 "oven"                       
[205] "overtaking"                  "parking"                    
[207] "parking:condition"           "parking:lane"               
[209] "passenger_lines"             "passing_places"             
[211] "place"                       "power"                      
[213] "power_supply"                "priority"                   
[215] "priority_road"               "produce"                    
[217] "proposed"                    "proposed:name"              
[219] "protected_area"              "psv"                        
[221] "psv:lanes"                   "public_transport"           
[223] "railway"                     "railway:preserved"          
[225] "railway:track_ref"           "recycling_type"             
[227] "ref"                         "ref_name"                   
[229] "reg_name"                    "religion"                   
[231] "religious_level"             "rental"                     
[233] "residential"                 "roadtrain"                  
[235] "route"                       "sac_scale"                  
[237] "sauna"                       "service"                    
[239] "service_times"               "shelter_type"               
[241] "shop"                        "short_name"                 
[243] "shoulder"                    "shower"                     
[245] "side_road"                   "sidewalk"                   
[247] "site"                        "ski"                        
[249] "smoking"                     "smoothness"                 
[251] "social_facility"             "sorting_name"               
[253] "speed_pedelec"               "sport"                      
[255] "start_date"                  "step_count"                 
[257] "substation"                  "surface"                    
[259] "tactile_paving"              "tank"                       
[261] "taxi"                        "tidal"                      
[263] "toilets"                     "toilets:wheelchair"         
[265] "toll"                        "topless"                    
[267] "tourism"                     "tourist_bus"                
[269] "tracks"                      "tracktype"                  
[271] "traffic_calming"             "traffic_sign"               
[273] "trail_visibility"            "trailblazed"                
[275] "trailblazed:visibility"      "trailer"                    
[277] "tunnel"                      "turn"                       
[279] "type"                        "unisex"                     
[281] "usage"                       "vehicle"                    
[283] "vending"                     "voltage"                    
[285] "water"                       "wheelchair"                 
[287] "wholesale"                   "width"                      
[289] "winter_road"                 "wood"                       
```

``` r
osmdata::available_tags("place")
```

```         
# A tibble: 30 × 2
   Key   Value      
   <chr> <chr>      
 1 place allotments 
 2 place archipelago
 3 place borough    
 4 place city       
 5 place city_block 
 6 place continent  
 7 place country    
 8 place county     
 9 place district   
10 place farm       
# ℹ 20 more rows
```

``` r
aoi_bbox = osmdata::getbb("Ecuador")

# Hospitals
buildings_hospital = aoi_bbox |> osmdata::opq() |>
  osmdata::add_osm_feature(key = "amenity", value = "hospital") |>
  osmdata::osmdata_sf() # |> st_transform(crs_master)

# Housing
buildings_residential = aoi_bbox |> osmdata::opq() |>
  osmdata::add_osm_feature(key = "building", value = "residential") |>
  osmdata::osmdata_sf() # |> st_transform(crs_master)

# Religion
buildings_workship = aoi_bbox |> osmdata::opq() |>
  osmdata::add_osm_feature(key = "building", value = "religious") |>
  osmdata::osmdata_sf() 

# Towns 
townnames = aoi_bbox |> osmdata::opq() |>
  osmdata::add_osm_feature(key = "place", value = "town") |>
  osmdata::osmdata_sf()

# Villages
villages = aoi_bbox |> osmdata::opq() |>
  osmdata::add_osm_feature(key = "place", value = "village") |>
  osmdata::osmdata_sf()

# Merging collections
places_merged = townnames$osm_points |> bind_rows(villages$osm_points) |>
  group_by(across(-geometry)) |> sf::st_cast("POINT") |>
  summarise(geometry = st_union(geometry), .groups = "drop")
#sf::st_write(places_merged, "./03_Spatial_Data/POP/places_merged.shp",
             #layer_options = "OVERWRITE=true",
             #delete_dsn=T,
             #delete_layer=T)

buildings_merged = buildings_hospital$osm_points |>
  bind_rows(buildings_residential$osm_points, buildings_workship$osm_points) |>
  group_by(across(-geometry)) |> sf::st_cast("POINT") |>
  summarise(geometry = st_union(geometry), .groups = "drop")
#sf::st_write(buildings_merged, "./03_Spatial_Data/POP/buildings_merged.shp",
             #layer_options = "OVERWRITE=true",
             #delete_dsn=T,
             #delete_layer=T)

# Visualize
tmap::tm_shape(aoi_country) + tmap::tm_borders(col="purple", lwd=1) +
  tmap::tm_shape(aoi_states) + tmap::tm_borders(col="purple", lwd=.5) +
#  tmap::tm_shape(buildings_merged) + tmap::tm_symbols(size=0.15,lwd=0.5,fill="white",col="purple") +
#  tmap::tm_add_legend(type="symbols", col="white", fill="purple", size=0.8, labels="Buildings") +
  tmap::tm_shape(places_merged) + tmap::tm_symbols(size=0.2,lwd=0.5,fill="white",col="blue") +
  tmap::tm_add_legend(type="symbols", col="white", fill="blue", size=0.8, labels="Towns & Villages") +
  tmap::tm_scalebar(position=c("RIGHT", "BOTTOM"), text.size = .5) +
  tmap::tm_compass(color.dark="gray60",text.color="gray60",position=c("RIGHT", "top")) +
  tmap::tm_graticules(lines=T,labels.rot=c(0,90),lwd=0.2) +
  tmap::tm_layout(legend.position=c("left", "top"), legend.bg.color = "white") +
  tmap::tm_title("Risk Covariates Map: Built Environment", size=.8) + 
  tmap::tm_basemap("OpenStreetMap") 
  #tmap::tm_basemap("OpenTopoMap") 
  #tmap::tm_basemap("Esri.WorldImagery") 
```

![](MAPS/Map_RiskCovariateScopingB_202508.png)<!-- -->

``` r
  #tmap::tm_basemap("OpenTopoMap") 
  #tmap::tm_basemap("Esri.WorldImagery") 
```

------------------------------------------------------------------------

#### Transportation 

``` r
# Motorway/Highways
transport_motorway <- aoi_bbox %>% opq() %>%
  add_osm_feature(key = "highway", value = "motorway") %>%
  osmdata_sf()

# Roads
transport_roads <- aoi_bbox %>% opq() %>%
  add_osm_feature(key = "XXXXXXX", value = "XXXXXXX") %>%
  osmdata_sf()

# Tracks
transport_tracks <- aoi_bbox %>% opq() %>%
  add_osm_feature(key = "XXXXXXX", value = "XXXXXXX") %>%
  osmdata_sf()

# Railways
transport_path <- aoi_bbox %>% opq() %>%
  add_osm_feature(key = "XXXXXXX", value = "XXXXXXX") %>%
  osmdata_sf()

# Merge all transport
transport_merged = transport_motorway |>
  bind_rows(transport_tracks, transport_path) |>
  group_by(across(-geometry)) |> sf::st_cast("MULTILINESTRING")
  summarise(geometry = st_union(geometry), .groups = "drop")
sf::st_write(transport_merged, "./03_Spatial_Data/Risk/transport_merged.shp", delete_dsn=T) 

# Visualize 
tmap::tm_shape(aoi_country) + tmap::tm_borders(col="green", lwd=3) +
  tmap::tm_shape(aoi_states) + tmap::tm_borders(col="orange") +
  tmap::tm_shape(buildings_merged) + tmap::tm_symbols(size=0.35,lwd=0.5,fill="",col="white") +
  tmap::tm_shape(transport_merged) + tmap::tm_symbols(size=0.35,lwd=0.5,fill="purple",col="white") +
  tmap::tm_add_legend(type="symbols", col="purple", fill="purple", size=0.8, labels="Buildings") +
  tmap::tm_scalebar(position=c("RIGHT", "BOTTOM"), text.size = .5) +
  tmap::tm_compass(color.dark="gray60",text.color="gray60",position=c("RIGHT", "top")) +
  tmap::tm_graticules(lines=T,labels.rot=c(0,90),lwd=0.2) +
  tmap::tm_layout(legend.position=c("left", "top"), legend.bg.color = "white") +
  tmap::tm_title("Risk Covariates Map: Built Environment", size=.8) + 
  tmap::tm_basemap("OpenStreetMap") 
  #tmap::tm_basemap("OpenTopoMap") 
  #tmap::tm_basemap("Esri.WorldImagery") 
```

------------------------------------------------------------------------

#### Hydrography 

``` r
waterways_country = sf::st_read("./03_Spatial_Data/HYDRO/waterways_country.shp")  #|>
  #sf::st_intersection(aoi_bbox) |> 
  #dplyr::select(name,fclass) |>
  #dplyr::mutate(fclas=as.factor(fclass)) |> 
  #dplyr::mutate(name=as.character(name)) 

waterbodies_country = sf::st_read("./03_Spatial_Data/HYDRO/waterbodies_country.shp")
  sf::st_intersection(aoi_country) |>
  dplyr::select(name,fclass) |>
  dplyr::mutate(fclass=as.factor(fclass)) |> 
  dplyr::mutate(name=as.character(name)) 

water_merged = waterbodies_country |>
  bind_rows(waterbodies_poly_lines, waterways_merged) |>
  group_by(across(-geometry)) |>
  summarise(geometry=st_union(geometry), .groups="drop") 

water_merged = sf::st_cast(water_merged, "MULTILINESTRING") 
sf::st_write(water_merged, "./03_Spatial_Data/HYDRO/water_merged.shp", delete_dsn=T)
```

------------------------------------------------------------------------

#### Topography 

``` r
##################################
##################################
##################################
dem = raster::subset(STACK, "DEM") 
##################################
##################################
##################################


slope_tangent = raster::terrain(dem,
  opt="slope",
  unit="tangent",
  neighbors=8,
  filename="./03_Spatial_Data/TOPO/slope_tangent.tif"
  )

slope_tangent = terra::rast("./03_Spatial_Data/TOPO/slope_tangent.tif")
slope_percent = slope_tangent * 100 
slope_percent = terra::clamp(slope_percent, 0, 100) 
slope_percent = raster::raster(slope_percent) 
raster::writeRaster(slope_percent,"./03_Spatial_Data/TOPO/slope_percent.tif", overwrite=T)
```

------------------------------------------------------------------------

### Risk-allocated deforestation 

Two methods were explored for weighting variables and creating a generalized deforestation risk index. For efficiency of time, we adopted the following risk indexing approach, which was based on a weighted sum of subjectively scored covariate effects. Please note this approach was also recommended for its ease of updating in subsequent iterations.

We applied this risk index to inform a risk weighted allocation of a 10-year deforestation rate, first by multiplying the fraction of pixel risk by zonal forest loss, and second by factoring out annual zonal loss by multiplying by pixel risk values, as shown below.[^readme-1]

$$
\mathrm{AllocatedLoss}_{\mathrm{pixel}}
=
\left(
  \frac{\mathrm{risk}_{\mathrm{pixel}}}{\sum \mathrm{risk}_{\mathrm{zone}}}
\right)
\times
\mathrm{annual\_loss\_10yr}_{\mathrm{zone}}
$$

$$
\mathrm{allocated\_loss}_{\mathrm{pixel}}
=
\mathrm{risk}_{\mathrm{pixel}}
\times
\left(
  \frac{\mathrm{annual\_loss\_10yr}_{\mathrm{zone}}}{\sum \mathrm{risk}_{\mathrm{zone}}}
\right)
$$

Both formulas describe the same operation in different orders of multiplication: each pixel in a given zone Z receives a share of `annual_loss_10yr` based on its proportional risk. This ensures that higher-risk pixels are allocated higher deforestation loss, which is in line with Verra guidance regarding allocated deforestation risk maps.

------------------------------------------------------------------------

#### Visualize Deforestation Risk 

``` r
#### assemble covariates
states        = sf::st_read("./03_Spatial_Data/RISK/aoi_states.shp")
districts     = sf::st_read("./03_Spatial_Data/RISK/aoi_districts.shp")
places        = sf::st_read("./03_Spatial_Data/RISK/places_merged.shp")
buildings     = sf::st_read("./03_Spatial_Data/RISK/buildings_merged.shp")
transport     = sf::st_read("./03_Spatial_Data/RISK/transport_merged.shp")
waterways     = sf::st_read("./03_Spatial_Data/RISK/water_merged.shp")
slope         = stars::read_stars("./03_Spatial_Data/RISK/slope_percent.tif")


tmap::tmap_mode("plot")
tmap::tmap_arrange(tm3, tm4, tm6, tm5, ncols=2)
```

#### Stratification of Distributed Deforestation Pixels

[^readme-1]: We computed risk allocation based on each pixel’s risk value relative to the sum of all pixel risks in that zone.
