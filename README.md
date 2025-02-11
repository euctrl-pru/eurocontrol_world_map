---
title: "Map of Ukraine with Crimea"
execute:
  keep-md: true
format:
  html:
    code-fold: false
---



Ukraine complained officially to have Crimea attached to their map.
We use Natural Earth which used _de facto_ control of the territory to define borders.

So Philippe Riviere provided a beta [World Atlas Maker](https://observablehq.com/@fil/private-world-atlas-maker-v5) to get an alternative GeoJSON, `alternatives.geo.json`, which has Crimea as part of Ukraine (and many other quirks you can customize [we make use of getting Guyane as standalone].)

You can also download a properties files with interesting attributes for the various geographical entities, `properties.csv`.[^1]



Using the tools & techniques described in the _Command-Line Cartography_ series from Mike Bostock [[1][p1], [2][p2], [3][p3], [4][p4]] we are going to prepare the data.

[p1]: <https://medium.com/@mbostock/command-line-cartography-part-1-897aa8f8ca2c>
[p2]: <https://medium.com/@mbostock/command-line-cartography-part-2-c3a82c5c0f3>
[p3]: <https://medium.com/@mbostock/command-line-cartography-part-3-1158e4c55a1e>
[p4]: <https://medium.com/@mbostock/command-line-cartography-part-4-82d0d26df0cf>

[^1]: in fact I added `iso_n3` to the values of `viewof clean_properties` before exporting the CSV file.


## Tools installation

As per Mike's series I (blindly) installed all the tools mentioned there:

```bash
npm install -g shapefile
npm install -g d3-geo-projection
npm install -g ndjson-cli
npm install -g d3
npm install -g topojson
npm install -g d3-scale-chromatic
npm install -g d3-dsv
```

## GIS manipulation

NDJson from GeoJSON
```bash
$ ndjson-cat alternatives.geo.json \
    | ndjson-split 'd.features' > alternatives.ndjson
```

CSV to Json:

```bash
$ csv2json properties.csv -o properties.json
```

Keep only a subset of the properties

```bash
$ ndjson-cat properties.json \
    | ndjson-split 'd.slice(1)' \
    | ndjson-map \
    '{id: d.id, iso_n3: d.iso_n3, \
      lon: d.lon, lat: d.lat, \
      name: d.name}' > alternatives.properties.ndjson
```

Put together geo and properties

```bash
$ ndjson-join 'd.id' \
    alternatives.ndjson alternatives \
    properties.ndjson > alternatives.properties-join.ndjson
```

geo with custom properties:

```bash
$ ndjson-map \
    'd[0].properties = {id: d[0].id, fid: d[1].iso_n3, lon: d[1].lon, lat: d[1].lat, name: d[1].name}, d[0]' \
    < alternatives.properties-join.ndjson \
    > alternatives.final.ndjson
```

Convert back to GeoJSON

```bash
$ ndjson-reduce \
    'p.features.push(d), p' '{type: "FeatureColltion", features: []}' \
    < alternatives.final.ndjson \
    > world.json
```

The final changes, like define FRA's and GUS's `fid` were done in [the Observable notebook](https://observablehq.com/@espinielli/world-map-for-aviation-flows-eurocontrol).

