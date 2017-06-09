# Working with openstreetmap data

## osm data extracts:
http://download.geofabrik.de/index.html





## Filtering with osmfilter

#### Installation 

```makefile
# installing brew (if you haven't)
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null

# installing osmfilter
$ brew install osmfilter
```

#### Documentation
http://wiki.openstreetmap.org/wiki/Osmfilter

#### Usage

```makefile
# get all streets
$ osmfilter myData.osm --keep="highway=residential =primary =secondary =tertiary =unclassified" -o=myOutput.osm

# get all named streets and drop nodes (smaller filesize, but can't be displayed in QGIS without nodes)
osmfilter germany-latest.o5m --keep="highway=residential =primary =secondary =tertiary =unclassified and name=*" --drop-version -o=germany-highways.o5m
```

what features can I filter?  
http://wiki.openstreetmap.org/wiki/Map_Features

#### finding named streets
```makefile
# first create subset with all street types desired
osmfilter germany-latest.o5m --keep="highway=residential =primary =secondary =tertiary =unclassified" -o=germany-highways.o5m

# next, get everything with the desired name
osmfilter germany-highways.o5m --keep="name=*Goethe*" -o=germany-goethe.osm
```



## Conversion with osmconvert

#### Installation 
Download from `https://wiki.openstreetmap.org/wiki/Osmconvert#Writing_CSV_Files`

#### Documentation
http://wiki.openstreetmap.org/wiki/DE:Osmconvert

#### Usage
```makefile
# convert .pbf to .o5m
osmconvert berlin-latest.osm.pbf -o=berlin-latest.o5m

# basic usage, will create tab-seperated file with "@oname", "@id"
# and "name" columns without header
$ osmconvert myData.osm --out-csv >myOutput.csv

# output with header, custom comma seperator and with "name" 
$ osmconvert myData.osm --csv-headline --csv-separator="," --csv="name postal_code" >myOutput.csv
```
#### Creating regional subsets
You can use lat/lon boundaries or a custom .poly files to create a subset for a certain region. You can create the .poly file in QGIS with `osmpoly_export` (install via the "addons" menu.)

See the [tutorial](https://docs.qgis.org/2.8/en/docs/training_manual/create_vector_data/create_new_vector.html) on creating a shapefile. Then select the shape with the "Select Single Feature" tool (now highlighted in yellow) and click the "OSM Poly" button to export as .poly file. Move the file to where your osm file is located and use the terminal:
```makefile
# create a regional subset using a custom made .poly file
osmconvert germany-latest.osm.pbf -B=hamburg.poly -o=hamburg.o5m
```


## Dates in street names
Regular expressions for filtering street names with dates in them
```js
// german dates
.*\d{1,2}\..*(Jan|Feb|Mär|Apr|Mai|Jun|Jul|Aug|Sep|Okt|Nov|Dez).*

// french dates
.*\d{1,2}.*(Jan|Fév|Mar|Avr|Mai|Jui|Aoû|Sep|Oct|Nov|Déc).*
```



## Visualizing osm data in Qgis
http://learnosm.org/de/osm-data/osm-in-qgis/





## In-Depth filtering with node-osmium

Install node-osmium via `npm install osmium csv-write-stream`

```js
// example for counting street names

var osmium = require("osmium"),
  fs = require("fs"),
  csvWriter = require('csv-write-stream'),
    reader = new osmium.Reader(`myData.osm`, {
    way: true
  }),
  buffer = null,
  handler = new osmium.Handler(),
  tags = null,
  streetmap = {};

// create output csv-file with custom headers
var writer = csvWriter({ headers: ["street", "count"]});
writer.pipe(fs.createWriteStream(`output/myOutput.csv`));

// define a handler for "way"-tags (osm lines ~= streets)
handler.on("way", function(way) {
  tags = way.tags();
  tags.name !== undefined && appendName(streetmap, tags.name);
});

// loop through the osm file
while ((buffer = reader.read())) {
  osmium.apply(buffer, handler);
}

// after looping through osm file, write counting result to output file
for (street in streetmap) {
  writer.write([street, streetmap[street]]);
}
writer.end();


// UTILITY
function appendName(map, name) {
// increase counter for street name if present. If not, create it with
// value of 1
  if (map[name]) {
    map[name]++;
  } else {
    map[name] = 1;
  }
}
```