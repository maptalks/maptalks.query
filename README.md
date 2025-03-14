# maptalks.query

[maptalks](https://github.com/maptalks/maptalks.js) layers data query tool
*  Request [maptalks-gl](https://github.com/maptalks/maptalks-gl-layers) version>0.98.0, current version 0.98.0 has a bug  [vt _convertGeometry](https://github.com/maptalks/issues/issues/717) when transform vt point to geojson(polygon)
*  Spatial Query rely on [jsts](https://github.com/bjornharrtell/jsts)
    

## Features

* support simple data filter. such as keyword query 
* support spatial query
* support layers:
  + VectorLayer
  + PointLayer
  + LineStringLayer
  + PolygonLayer
  + VectorTileLayer
  + GeoJSONVectorTileLayer
  + ...

* Time slicing to solve the problem of large data volume lag

## Examples

* [simple data filter](https://maptalks.org/maptalks.query/demo/base.html)
* [simple spatial query](https://maptalks.org/maptalks.query/demo/base-spatial.html)
* [vt data filter](https://maptalks.org/maptalks.query/demo/vt.html)
* [vt spatial query](https://maptalks.org/maptalks.query/demo/vt-spatial.html)
* [spatial query operator](https://maptalks.org/maptalks.query/demo/spatial-op.html)
* [buffer query](https://maptalks.org/maptalks.query/demo/buffer.html)
* [multi layers query](https://maptalks.org/maptalks.query/demo/mutl-layer.html)
* [mock map identify](https://maptalks.org/maptalks.query/demo/mock-identify.html)
* [tileclusterlayer query](https://maptalks.org/maptalks.query/demo/tileclusterlayer.html)

## Install

### NPM

```sh
npm i maptalks-gl
npm i maptalks.query
```

## CDN

```html
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/maptalks-gl/dist/maptalks-gl.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/maptalks.query/dist/maptalks.query.js"></script>
<!-- if need spatial query -->
<script src="https://cdn.jsdelivr.net/npm/jsts@2.11.3/dist/jsts.min.js"></script>
```

## API

### `injectJSTS(jsts)`

Inject jsts namespace. If you need spatial query, this is necessary

```js
import * as jsts from 'jsts';
import {
    injectJSTS
} from 'maptalks.query';

injectJSTS(jsts);
```

### `Query` class

layers query tool Class

#### `constructor(map, options)`

```js
 new Query(map, options)
```

* `map`: maptalks Map
* `options`:config object
  + `options.log`: Whether to output logs when calculation errors occur

```js
 const query = new Query(map, {
     log: true
 })
```

```js
import {
    Map,
    TileLayer,
    Polygon
} from 'maptalks-gl';
import {
    Query,
    injectJSTS
} from 'maptalk.query';
import * as jsts from 'jsts';

injectJSTS(jsts);

const mapView = {
    'center': [120.54069005, 31.14989446],
    'zoom': 9.49995108663881,
    'pitch': 0,
    'bearing': 0
};
// create map
const map = new Map('map1', {
    ...mapView,
    baseLayer: new TileLayer('base', {
        urlTemplate: 'https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png',
        subdomains: ['a', 'b', 'c', 'd'],
        attribution: '&copy; <a href="http://osm.org">OpenStreetMap</a> contributors, &copy; <a href="https://carto.com/">CARTO</a>'
    })
});
//create query instance 
const query = new Query(map, {
    log: true
});
```

#### methods

* `query(options)` simple data filter query. Keyword search, etc, return `Promise`
  + `options.filter`:data filter function
  + `options.layers`: The layer to be queried, if empty, will query all layers on the map

```js
function getPropties(geo) {
    let properties = geo.getPropties ? geo.getPropties() : geo.properties;
    properties = properties || {};
    if (properties.__original_properties) {
        return properties.__original_properties;
    }
    return properties;
}

query.query({
    filter: (geo, layer) => {
        const properties = getPropties(geo);
        const name = properties.name;
        //mock simple keyword query
        return name && name.includes('hello world');
    },
    layers: [layer, layer1, ...otherlayers]
}).then(result => {
    console.log(result);
    //do some things
}).catch(error => {
    console.error(error);
});
```

* `spatialQuery(options)` Geometric spatial query . return `Promise`

  + `options.geometry`: Input query geometry, maptalks. Polygon/maptalks. Circle/maptalks.
  Rectangle...
  + `options.filter` data filter function if need
  + `options.layers`: The layer to be queried, if empty, will query all layers on the map 
  + `options.op` Geometric Shape Relationship Query Operation, default value is ` Query.intersects`  
    op support list:

    - Query.contains
    - Query.crosses
    - Query.disjoint
    - Query.equals
    - Query.intersects
    - Query.contains
    - Query.overlaps
    - Query.within

```js
injectJSTS(jsts);

const polygon = new Polygon([...]);
query.spatialQuery({
    geometry: polygon,
    layers: [layer, layer1, ...otherlayers]
    op: Query.intersects
}).then((result) => {
    console.log(result);
    //do some things
}).catch(error => {
    console.error(error);
});
```

* `dispose()`

```js
query.dispose();
```
