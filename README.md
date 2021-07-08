# Globelet

Lightweight vector maps on a globe

Inspired by [Leaflet][]: a simple, light-weight mapping library, without the 
distortion of flat maps. Show your GIS data in 3D, as it would appear from 
space.

See a [simple example][] with geojson data displayed over the [Basic Style][]
from [OpenMapTiles][].

Like Leaflet, we design for *simplicity*, *performance*, and *usability*.

Need lots of features, like 3D buildings? Try [CesiumJS][]. Globelet will only do
a few things, but it will do them well.

[Leaflet]: https://github.com/Leaflet/Leaflet
[simple example]: https://globeletjs.org/examples/geojson/index.html
[Basic Style]: https://github.com/openmaptiles/maptiler-basic-gl-style
[OpenMapTiles]: https://openmaptiles.org/
[CesiumJS]: https://github.com/AnalyticalGraphicsInc/cesium

![tests](https://github.com/GlobeletJS/GlobeletJS/actions/workflows/node.js.yml/badge.svg)

## How to add GlobeletJS code to your webpage
GlobeletJS is provided as an ESM import. Define your script tag as
`type="module"`, then import the module:
```html
<script type="module">
  import * as globeletjs from "https://unpkg.com/globeletjs@<VERSION>/dist/globelet.js";

  // Add code to initialize a globe here...
  // ...
</script>
```

Or if you prefer, you can use the older-style [IIFE][] bundle:
```html
<script src="https://unpkg.com/globeletjs@<VERSION>/dist/globelet-iife.js">
```

Either bundle will give you a global variable `globeletjs`, which has an 
`initGlobe` method. See the next section for how to use this method.

Make sure to also link to the stylesheet (/dist/globelet.css) in the `<head>`
of your HTML file.
```html
<link 
  rel="stylesheet" 
  type="text/css" 
  href="https://unpkg.com/globeletjs@<VERSION>/dist/globelet.css">
```

[IIFE]: https://developer.mozilla.org/en-US/docs/Glossary/IIFE

## How to initialize a globe
The `globeletjs` object has an `initGlobe` method that can initialize a new 
globe as follows:
```javascript
const params = {
  container: 'globe',
  style: "./klokantech-basic-style-geojson.json",
  center: [-100, 38.5],
  altitude: 6280,
};

const globePromise = globeletjs.initGlobe(params);
```

The `params` object supplied to initGlobe can have the following properties:
- `container` (REQUIRED): The [ID][] of an [HTML DIV element][] where the 
  globe will be displayed
- `style` (REQUIRED): A link to a [MapLibre style document][Maplibre] 
  describing the map to be rendered. Please see below for some notes about
  [supported map styles](#supported-map-styles).
- `mapboxToken`: Your API token for Mapbox services (if needed)
- `width`: The width of the map that will be projected onto the globe,
  in pixels. Defaults to `container.clientWidth + 512`
- `height`: The height of the map that will be projected onto the globe,
  in pixels. Defaults to `container.clientHeight + 512`
- `center`: The initial geographic position of the camera, given as
  [longitude, latitude] in degrees. Default: [0.0, 0.0]
- `altitude`: The initial altitude of the camera, in kilometers.
  Default: 20000

The returned Promise resolves to an API handle, which you can use to interact
with the globe, as described in the next section.

[ID]: https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id
[HTML DIV element]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div
[MapLibre]: https://maplibre.org/maplibre-gl-js-docs/style-spec/

## How to interact with the globe API
The Promise returned by `initGlobe` resolves to an API object, which you can
use to control the globe.

```javascript
globePromise.then(globeAPI => {
  globeAPI.startAnimation();

  // ...etc. Do more things with globeAPI here...
});
```

`globeAPI` exposes the following properties and methods:
- `mapLoaded()`: Returns a fractional number from 0.0 to 1.0 indicating the
  fraction of the tiles needed for the current view that are fully loaded
- `select(layer, dxy)`: Selects map features from the specified layer, within
  pixel distance `dxy` from the current cursor position
- `showLayer(layer)`: Turns on rendering for the specified map layer
- `hideLayer(layer)`: Turns off rendering for the specified map layer
- `getZoom()`: Returns the current zoom level of the map
- `startAnimation()`: Starts animation
- `stopAnimation()`: Stops animation
- `update(time)`: Updates the camera position based on current position and
  velocity and cursor inputs since the last update. Input is a timestamp in
  milliseconds as supplied by [requestAnimationFrame][]. For animation loops
  managed by the parent program
- `cameraPos`: Link to the camera position as reported by [spinning-ball][] 
- `cursorPos`: Link to the cursor position as reported by [spinning-ball][]
- `isMoving()`: Returns the value of the camMoving flag in [spinning-ball][]
- `wasTapped`: Returns the value of the wasTapped flag in [spinning-ball][]
- `addMarker(options)`: Adds a marker to the globe. See markers section below
- `removeMarker(marker)`: Removes a given marker from memory and from the DOM
- `destroy()`: Clears memory / removes elements from document

[requestAnimationFrame]: https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame

## How to add and remove markers
A marker can be added to the globe as follows:
```javascript
const marker = globeAPI.addMarker(options);
```

where `options` is an object with the following properties:
- `element`: A link to an HTML Element that will be used to visualize the
  marker position. If not supplied, a default SVG will be used
- `type`: The type of marker. If `type === "spot"`, the marker element will
  default to a circular SVG; otherwise it defaults to a standard placemarker 
  SVG
- `lonLat`: A 2-element Array containing longitude and latitude of the desired
  marker position, in radians

The returned `marker` object has the following properties:
- `element`: Back-link to the HTML element used to represent the marker
  position
- `lonLat`: A Float64Array containing the longitude and latitude of the marker
  position, in radians
- `screenPos`: A 2-element Array containing the current screen position of the
  marker, in pixels from top left

At each animation frame (or each call to globeAPI.update()), GlobeletJS will 
automatically update `screenPos`, and use it to set the element's displayed 
position via `style.left` and `style.top`.

A marker can be removed from the globe as follows:
```javascript
globeAPI.removeMarker(marker);
```

## Supported map styles
Many features described in the [style specification][MapLibre] are not yet
supported. This is partly by design--GlobeletJS is intended to be an 80/20
map solution, implementing 80% of the specification with 20% as much code as
other software. But we are adding support for more features over time.

The map rendering is delegated to the [tile-setter][] module, which is
limited by some of its dependencies. See the ["un-supported features" list for
tile-stencil][tile-stencil-limitations] and the [tile-gl TODO list][tile-gl-todo]
for an (incomplete) list of what is not supported.

We welcome your feedback on what additional features you would like to see
supported!

[tile-stencil-limitations]: https://github.com/GlobeletJS/tile-stencil#un-supported-features
[tile-gl-todo]: https://github.com/GlobeletJS/tile-gl#todo

## About the code (for advanced users)
For development on Node.js, you can install the package from NPM:
```bash
npm install --save globeletjs
```

GlobeletJS works by tying together several other more specialized modules.
1. We render vector map data to a rectangular texture using [tile-setter][]
2. Then we wrap the map around a globe using [satellite-view][]
3. To animate the camera position, we incorporate [spinning-ball][]

Also, to save on typing, we delegate the low-level WebGL calls to [yawgl][]

[tile-setter]: https://github.com/GlobeletJS/tile-setter
[satellite-view]: https://github.com/GlobeletJS/satellite-view
[spinning-ball]: https://github.com/GlobeletJS/spinning-ball
[yawgl]: https://github.com/GlobeletJS/yawgl
