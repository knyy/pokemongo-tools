# Tools for generating compiled datasets for EX-raid map
Demo: https://xiankai.github.io/sg-pokemongo-ex-raid-map/  
Linked Repo: https://github.com/xiankai/sg-pokemongo-ex-raid-map

# Installation
This assumes basic knowledge of `node`, `npm` or `yarn`.

1. `yarn add pokemongo-tools`
2. `yarn` to install dependencies. 
    - You must be on node 6 to compile @mapbox/s2-node on OSX, but you should switch to node 8+ for better performance afterwards.
    - For reference, it takes 55s to process 1.8k gyms on v6.12.2 yet only 17s on v9.3.0

# overpass.c
OSM query template for https://overpass-turbo.eu/, copied from [this reddit thread](https://www.reddit.com/r/TheSilphRoad/comments/7pq1cx/how_i_created_a_map_of_potential_exraids_and_how/)

Also modified to only show valid park tags.

I originally had one myself but it is outdated with the current research thus far and thus invalid.

It has the `c` extension for formatting as the syntax is derived from C.

After running the query you should have the option to Export as a GeoJSON file. There is also the option to backdate the query to what the OSM map was at a certain date.

# s2.js
This will generate the `s2_L12.geojson` and `s2_L10.geojson` required. You should only need to run this once.

Given 2 coordinates to define a rectangular boundary for your desired area, it will produce a GeoJSON file containing S2 level 12 and level 10 cells, labelled with a grid with A-Z for columns and numbers for rows.

## Input

### coordinates.txt
```
1.2404, 104.0152
1.4714, 103.6318
```

### Usage (For L12 cells by default)
`node s2.js`

### Generate for a different cell level
`node s2.js 10`

### Optionally use a different key (default is `order`) for the GeoJSON property
`node s2.js 10 s2Cell`

# all.js
This will generate the `all.geojson` required.

## Input
The following CSV files are required. 

Some general notes about gym data and CSV files:
- There may be multiple gyms with the same name. You should make sure the names are unique (possibly by appending (2) to the duplicate), because the script will rely on string matching.
- `"` should be used to enclose gym names, `,` for delimiters.
- You can include `"` in names by using `""` instead.
    - eg. `"New ""Old"" TPY 240 Playground",1.3408059999999997,103.850371`

### gyms.csv
- This should rarely change.
- Headers **should not be present**.
- You can name these headers anything, but your columns must be in the correct order.
```
| Gym Name           | Latitude           | Longitude  |
|--------------------|--------------------|------------|
| Nicoll Highway MRT | 1.3002109999999998 | 103.863283 |
| Marymount Station  | 1.347791           | 103.840229 |
```

### exraids.csv
- This will likely be updated frequently as your data arrives in. Each column gets its own date, and empty cells means that the location did not have a raid for that wave. 
- **The first row should first contain any string ("Gym Name"), and then ex raid dates (in YYYY-MM-DD format)**.
- In the columns, either put the start timing of the raid (in 24 hour format) or any other non-empty string (if you don't know, or don't want to add the raid time). All raids are assumed to be 45 minutes long.
```
| Gym Name           | 2017-12-03 | 2017-12-18 | 2018-01-09 |
|--------------------|------------|------------|------------|
| Nicoll Highway MRT |            | 10.30      | 16.30      |
| Tablet of Flora    |            |            | 12.30      |
| Foliage Garden     | x          |            |            |
```

### *.park.geojson (generated from `overpass.c`)
Originally I made this script to allow checking against multiple versions of OSM park areas, each of which followed the naming convention of `Jul 2016.park.geojson`, `Jan 2017.park.geojson`, etc.

If you only wish to use park data from one source, just name it anything as long it has the extension `.park.geojson`.

For a more comprehensive source of park map data, look at https://github.com/MzHub/osmcoverer

### s2_L12.geojson (generated from `s2.js`)
### s2_L10.geojson (generated from `s2.js`)

### Usage
The above file names are assumed to be in the same directory.

`node all.js`
