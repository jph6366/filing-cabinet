
# 101 Geospatial Questions

## How convert screen coordinates to plane coordinates and then to geographic coordinates?


1. **Screen to Plane Coordinates**:  
    Use the inverse transformation matrix of the plane. This takes the screen coordinates (typically in pixels) and maps them onto the plane’s coordinate system.
2. **Plane to Geographic Coordinates**:  
    Once in plane coordinates (2D XY), you have a couple of options based on your needs:

- **Tile Coordinates**: If working within a Web Map framework, you can transform plane coordinates to tile coordinates. This allows you to utilize the Web Map tile logic and scales directly with zoom levels.
- **Ellipsoidal Geographic Coordinates**: For more precise geospatial work, apply a tangent plane transformation using geodesic formulas. This involves projecting from Earth-centered, Earth-fixed (ECEF) coordinates onto the tangent plane’s East-North-Up (ENU) system, centered at a reference point on the Earth’s ellipsoid.

Each of these transformations can be applied in both directions, allowing for conversions from geographic to screen coordinates and vice versa.

 ## What is the most accurate method of measuring distance between two points on Earth?

### Reading
- [calculations for lati­tude/longi­tude points, with the formulas and code fragments for implementing them](https://www.movable-type.co.uk/scripts/latlong.html)
- [Vincenty solutions of geodesics on the ellipsoid](https://www.movable-type.co.uk/scripts/latlong-vincenty.html)
- [algorithms for the computation of geodesics on an ellipsoid of revolution are given](https://link.springer.com/article/10.1007/s00190-012-0578-z)
  - Charles Karney has improved on Vincenty’s method with a method which has errors in nanometers (and always converges on antipodal points)
- [GeographicLib is collection of libraries by Charles Karney which started in 2008.](https://geographiclib.sourceforge.io/)
  - The C++ library now offers:
    - geodesic and rhumb line calculations;
    - conversions between geographic, UTM, UPS, MGRS, geocentric, and local cartesian coordinates;
    - gravity (e.g., EGM2008) and geomagnetic field (e.g., WMM2020) calculations.
- [centimeter-level, robust GNSS-aided inertial post-processing for mobile mapping without local reference stations](https://www.applanix.com/news/post-processed-centerpoint-rtx-with-pospac-8.pdf)
- [robust inertial post-processing aided by trimble propoint gnss technology for urban hd mapping and autonomous navigation](https://downloads.ctfassets.net/9k5dj5b59lqq/1QRYJYxlYVTOe5K51PKlNH/f71bd4a058cd0084fb3023fb8746c0cb/White-Paper_Robust-Inertial-Post-Processing-Aided-by-Trimble-Propoint-GNSS-Technology-For-Urban-HD-Mapping-and-Autonomous-Na.pdf)
- [Applanix SmartBase](https://assets.ctfassets.net/9k5dj5b59lqq/5Ff15GhCa0oTuW9lsOHyYc/a67dc562dcc87eb08ac46eb06b364225/applanix_smartbase.pdf) & [Applanix InFusion+](https://assets.ctfassets.net/9k5dj5b59lqq/77Pf1UjfXpZ6kE5xPMFrxY/8cd418e088d198b2da171615680065cf/Applanix-In_Fusion-Plus-White-Paper.pdf) white papers, software for improved robustness, accuracy, and productivity of mobile mapping and positioning


 #### The first question one must ask is what coordinate reference system (CRS) are these two points represented in
   - Assuming 2D Geographic coordinates (longitude, latitude)
 #### The followup question should then be how precise are our coordinates?
   - Meaning what degree to which is the longitude and lattitude pins down an actual point on Earth.
   - In Computer Science, a **64 bit floating point precision** is required to represent geodectic coordinates accurately. This includes both geocentric coordinates, which are typically very large numbers, and geographic coordinates, which are smaller but very precise numbers. Wherever possible use 64-bit floating point types for representing geodetic coordinates
     - In C++ use double instead of float, likewise use glm::dvec3 instead of glm::vec3.
     - In Unity use Unity.Mathematics.double3 instead of Vector3.
     - In JavaScript numbers are always stored as double precision floats. However, take care when using typed arrays to avoid Float32Array.
     - The blog post Precisions, Precisions goes over some strategies for rendering global scale coordinates on the GPU with high precision.
       
   - Approximately one degree of latitude covers

     10<sup>7</sup>/ 90 = 111,111 m

   - And a degree of longitude covers is about the same
  
   - Thus, it is always safe to figure that the

   **sixth decimal place in one decimal degree has 111,111 / 10<sup>6</sup> = about 1/9 meter = ~4 inches of precision**
   
   #### What about about other decimal places?
    
    - the first decimal place = ~11.1 km
    - the second decimal place = ~1.1 km
    - the third decimal place = ~110 m
    - the fourth decimal place = ~11 m
    - the fifth decimal place = ~1.1 m
    - the sixth decimal place = ~11 cm
    - the seventh decimal place = ~1.1 cm
    - the eighth decimal place = ~1.1 mm
    - the ninth decimal place = ~110 microns
   ...
   - Thirteen decimal places will pin down the location to...

   **111,111 / 10<sup>13</sup> = 1 angstrom ; around half the thickness of a small atom**

  #### What's the difference between geocentric and geodetic latitude?

  - I also forgot to mention the Earth is oblate spheroid rather than a sphere so there are two different systems for desecribing latitude
  - A point described using geocentric latitude and a point described using geodetic latitude can differ by tens of kilometers.

![image](https://github.com/user-attachments/assets/7e195e42-1857-49ce-88c1-5eb1c11c68be)
 
  - Only the **geodetic surface normal** should be used when precision matters

 #### Now one can infer the level of precision of our coordinates, so how does one go about measuring the distance between two points on Earth?

- Well if one were measuring on a sphere then the **Spherical Law of Cosines** or the **Haversine Formula**. Using these on an ellipsoid typically result in less than 0.3% errors.
  - Although the Law of Cosines is preferable to the Haversine Formula due to the formula [not entailing the evaluation of trigonometric functions](https://gis.stackexchange.com/questions/4906/why-is-law-of-cosines-more-preferable-than-haversine-when-calculating-distance-b?rq=1#:~:text=A%20historical%20footnote%3A).

  - pseudo-code for cosine formula
  
          distance = acos(SIN(lat1)*SIN(lat2)+COS(lat1)*COS(lat2)*COS(lon2-lon1))*6371
  
  - pseudo-code for haversine formula
  
          dLat = (lat2-lat1)
          dLon = (lon2-lon1)
          a = sin(dLat/2) * sin(dLat/2) + cos(lat1) * cos(lat2) * sin(dLon/2) * sin(dLon/2)
          distance = 6371 * 2 * atan2(sqrt(a), sqrt(1-a))    
    
- But if one is measuring on an oblate spheroid, then one can opt to use **Vincenty's Formula**
  - **Vincenty’s solution for the distance between points on an ellipsoidal earth model is accurate to within 0.5 mm distance,0.000015″ bearing,** on the ellipsoid being used.
  - Vincenty’s inverse solution can fail on nearly antipodal points, two points of a sphere are called antipodal or diametrically opposite if they are the endpoints of a diameter, a straight line segment between two points on a sphere and passing through its center. This can happen with distances greater than 19,936 km, or within around 75 km of the antipodal point.

  ![image](https://github.com/user-attachments/assets/754e9502-090a-4a78-a997-2d8103c55c5d)

- **Millimetre precision is stretching the limits of geodetic survey techniques**: very few points will have been surveyed to such precision
  - Further, the familiar WGS-84 (World Geodetic System 1984) has no ‘physical realisa­tion’ – it is not tied to geodetic groundsta­tions, just to satellites – and is defined to be accurate to no better than around ±1 metre
  - Working to an accuracy of better than ±1 metre, plate tectonic movements become significant. Depending on the datum it’s defined in, any given point will most likely have shifted many millimetres from where it was last year.
    - The [**International Terrestial Reference System and Frame**](https://itrf.ign.fr/en/homepage) was developed where the latitude/longitude of a coordinate of a position will vary over time.
  - So for millimetre accuracy to be relevant, you need to know a fair bit about the reference frames, datums, and epochs of your coordinates, and other param­eters of your measure­ments. 
![image](https://github.com/user-attachments/assets/80e662e7-bb13-4b9d-be25-ccd36fa5e269)


 #### Map-based Localization

- A map-based localization system uses optical sensors to match features in captured imagery or LiDAR scans against an existing georeferenced
database of features (ie. a map). This process is also referred to as **“map aiding”**.
 - If the map is accurate and georeferenced with respect to a global coordinate system, the resultant vehicle positions are also
accurate and georeferenced with respect to the global coordinate system.
   - If a map does not exist, the most sophisticated MBL
solutions can build a local map from the imaging sensors while the vehicle is driving, and then determine the vehicle’s relative
position within that map. Such a process is referred to as **Simultaneous Location and Mapping or SLAM**.


# miscellaneous
  
 - Explain how WMS, WMTS, and XYZ Tiles work

 - Given a coordinate in latitude and longitude and its CRS (coordinate reference system), how can you reproject the coordinates to a new CRS?

 - What is the difference between vector data sets and raster data sets?

- Describe the differences between spatial indexes and spatial joins. What about spatial overlays?

## What is the State Plane Coordinate System (SPCS)?

#### Reading
[NOAA Special Publication NOS NGS 13](https://geodesy.noaa.gov/library/pdfs/SP_NOS_NGS_13.pdf "The State Plane Coordinate System - 2018")

### Notes

- A set of **124 geographic zones over the US**. The system is highly accurate within **each zone with an error of less than 1:10,000**, meaning the difference between a length of 10,000 meters on the ellipsoid and its representation on the map would be about 1 meter. It is roughly four times more accurate than **UTM**.
- **SPCS 2022** has up to **three zone layers in each state**, and the number of zones will vary greatly between states. Every U.S. state and territory will have a statewide zone. Most states will also have a multiple-zone layer that covers the entire state, and some states will also have a multiple-zone layer that covers only part of the state. In addition, there will be three “special use” zones that each cover more than one state.
  - The reason for large variation in the number of layers and zones is that stakeholders in many states actively participated in the SPCS 2022 design process. **41 states submitted 67 requests** and/or proposal forms for SPCS 2022 zones. Requests were for zones designed by NGS, and proposals were for zones designed by stakeholders. Proposals were reviewed and approved by NGS, **resulting in 28 states designing their own zones**.
- **SPCS 83** was created by the National Geodectic Survey in 1983; SPCS 83 consists of 125 zones based on the Lambert Conformal Conic, Transverse Mercator, and Oblique Mercator projections.
- In the 1930s, SPCS provided a way to perform "geodectic" survey using plane trigonometry, making it among the earliest practical means to access the National Spatial Reference System (NSRS)
- As early as 1817, in New York Harbor there was a **Survey of the Coast** as its first field project. In 1832, work spread and expanded along to the Gulf Coast and parts of the West Coast by 1851.
- In 1871, the **Transcontinental Arc of Triangulation** was undertaken to conduct a precise survey across the country.
- In 1930, the survey network spanned the entire country and formed a "lattice" pattern. Great Depression relief programs made great progress in expanding the survey network and now there is more than 1,500,000 positioned points in the NGS's database.

 **Mapping Concepts and Standards:**
  - [x] _Polyconic Projection_ by Ferdinand Hassler, the first superintendent of the Survey of the Coast, 1987
  - [x] _World Polyconic Grid_ by Army Map Service
  - [x] _Conformal Map Projections_ , two conformal projections were adopted around 1920, the Lambert Conformal Conic and the Gauss-Krüger form of the Transverse Mercator

  - Conformality enforces the condition that, at a point, angles are preserved and scale error is the same in all directions. These qualities preserve shape locally, and it makes them particularly useful for calculations involving directions, azimuths, and distances.
  - Both adopted properties of conformality and equal area preservation (which are mutually exclusive) are derived from Lambert's Composition of Terrestial and Celestial Maps

## Convert a GPS coordinate to a pixel position in a Web Mercator tile

https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames#:~:text=Example%3A%20Convert%20a%20GPS%20coordinate%20to%20a%20pixel%20position%20in%20a%20Web%20Mercator%20tile%5Bedit%20%7C%20edit%20source%5D

## What is Slippy Map?

#### Reading
[OpenStreetMapWiki/Slippy_map](https://wiki.openstreetmap.org/wiki/Slippy_map)
[OpenStreetMapWiki/Slippy_map_tilenames](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames)
### Notes

- Slippy map is a tiled web map or tile.
   - A web map displayed seamlessly joining dozens of individually requested data files, called "tiles".
   - In general, modern web maps which let you zoom and pan around (the map slips around when you drag the mouse).
- In technical details, slippy map is an AJAX component.
   - JavaScript runs in the browser, which dynamically requests maps from a server in the background (without reloading the whole HTML page) to give smooth slippy zoomy map browsing experience.
      - The map image is built up of tiles that are rendered from the tile server, the process of rendering, going from vector to raster map data, baking style choices into bitmap images, is a fairly resource-intensive process.
- Most tiled web maps follow certain Google Maps conventions:
   - Tiles are 256x256 pixel
   - At the outer most zoom level, 0, the entire world can be rendered in a single map tile.
   - Each zoom level doubles in both dimensions, so a single tile is replaced by 4 tiles when zooming in. This means that about 22 zoom levels are sufficient for most practical purposes.
   - The Web Mercator projection is used, with latitude limits of around 85 degrees.
   - The de facto OpenStreetMap standard, known as Slippy Map Tilenames or XYZ, follows these and adds more:
      - An X and Y numbering schem
      - PNG images for tiles
      - Images are served through a Web server, with a URL like http://.../Z/X/Y.png, where Z is the zoom level, and X and Y identify the tile.
- **The Numbering Schemes:**
   - Google Maps / OpenStreetMap: (0 to 2zoom-1, 0 to 2zoom-1) for the range (−180, +85.0511) - (+180, −85.0511)
   - Tile Map Service: (0 to 2zoom-1, 2zoom-1 to 0) for the range (−180, +85.0511) - (+180, −85.0511). (That is, the same as the previous with the Y value flipped.)
   - QuadTrees, used by Microsoft.
