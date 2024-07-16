# WireTool for Lens Studio
by Max van Leeuwen


## [Download WireTool from Gumroad!](https://maxvanleeuwen.gumroad.com/l/wiretool)  


<br><i>(Click for video)</i><br>
[![WireTool](https://img.youtube.com/vi/k_39vDHRVzs/0.jpg)](https://www.youtube.com/watch?v=k_39vDHRVzs)

<br>

WireTool creates a continuous 3D wire based on a collection of points. It's useful for static 'tube' meshes or animated trails. Simply place this script in your scene and from any other script,<br>call `new WireTool.Create()`.<br>See the list of all possible customizations below.

And check out all the examples that come with the download!

To keep it as performance-friendly as possible (especially when rendering lots of wires simultaneously), see the 'Tips & Tricks' section! There are also some helpful notes on working with shaders there.

<br>

## Quickstart

```javascript
// a new instance
var wire = new WireTool.Create()
wire.radius = 1 // custom radius


  // function that prints the current point count
  function printPointCount(){
    print(wire.getLocalPoints().length)
  }
  // bind to callback (for demonstration purposes)
  wire.onPointsAdded.add(printPointCount)


// add two points, x=0 and x=10 - this renders a horizontal tube
wire.addPoints(new vec3(0, 0, 0), new vec3(10, 0, 0))

// (the total amount of points (2) will be printed to the logger because of the callback)
```

<br>

## Usage

### Drawing

- `.addPoints(...points)` - append points to this wire (`...vec3/objects`, any number of arguments)
- `.replace(...points)` - clear and add new points (`...vec3/objects`, any number of arguments)

### Callbacks

Bind functions to callbacks using `.add(function)` and `.remove(function)`.

- `.onPointsAdded` - event: on new points added to wire (and passed the `stepDistance` filter)
  - Callback arguments: an array of the passed-in point objects (local), containing: `{ position: vec3, distance: number, color: vec4, radius: number }`

### Customization

- `.setPointLimit(number)` - set a point limit, oldest points will be removed when exceeded (int, default: 500)
- `.setLengthLimit(number)` - set a length limit, oldest points will be removed when exceeded (default: Infinity)
- `.setSegments(number)` - change the wire's segment count (default: 7)
- `.setMaterials(materials)` - add material(s) to the mesh, removes existing (`...Asset.Material`, any number of arguments)
- `.enableNormals(bool)` - enable/disable normals (default: true)
- `.enableUVs(bool)` - enable/disable UVs (default: true)
- `.enableRadiusAttribute(bool)` - enable/disable custom radius vertex attribute, useful for vertex shaders (default: false)
- `.enableColorAttribute(bool)` - enable/disable color attribute (default: false)
- `.enablePreview(bool)` - enable/disable previewing the newest point when not yet added to wire (default: true)
- `.enableEndCaps(bool)` - enable/disable end caps on the mesh (default: false)
- `.enableEndCapsOutwardNormals(bool)` - enable/disable outward normals on the end caps, useful for vertex shaders (default: false)
- `.enable2SegmentsOutwardNormals(bool)` - enable/disable outward normals when the segment count is 2, useful for vertex shaders (default: false)
- `.enableEdgeLoop(bool)` - enable/disable an extra edge loop (when segments > 2 and UVs are enabled), making the UV.x attribute wrap instantly from 1 back to 0 (default: true)
- `.enableCurvedSubdivisions(bool)` - enable/disable smoothing on subdivisions if `subdivide > 0` (default: true)
- `.radius` - mesh radius for newly added points (default: 5cm)
- `.stepDistance` - minimum distance before creating a new point
- `.subdivide` - subdivide given path points (applied before `stepDistance` filtering!), handy for higher resolution vertex shader shaders (default: 0)

### Utility

- `.getLocalPoints()` - returns all wire positions in local space (previewing point excluded) (`vec3` array)
- `.localToWorld(position)` - convert local position on this wire to a world position (`vec3`)
- `.getPreviewPoint()` - returns preview position, if any (`vec3`)
- `.getPointLimit()` - returns current point limit (int)
- `.getLengthLimit()` - returns current length limit
- `.getSegments()` - returns segment count (int)
- `.getMaterials()` - returns materials (`Asset.Material` array)
- `.getLength()` - returns total wire length, including preview
- `.getPartLength()` - returns the length of the currently rendered part of the wire, including preview
- `.getClosestPoint(position)` - returns nearest interpolated point on the wire to a world position (returns an object containing: `{ position: vec3 (world), direction: vec3 (world), color: vec4, radius: number }`)
- `.getSceneObject()` - returns the generated `SceneObject`
- `.getRenderMeshVisual()` - returns the generated `Component.RenderMeshVisual` containing this wire's mesh asset
- `.getRenderMesh()` - returns the generated `Asset.RenderMesh`
- `.getBuilder()` - returns the generated `MeshBuilder`
- `.destroy(delay)` - destroys this wire and its `SceneObject`, optional delay argument (s)
- `WireTool.clear()` - clear all existing wires

<br>

### Debugging

- `.rebuildMesh()` - force rebuild mesh

<br>

## Tips & Tricks

- It is recommended to keep the `stepDistance` value as large as possible if wires are drawn on runtime, as this will prevent too much geometry being generated. If you instead want the wire to always accept every point you give it, set `stepDistance` to 0.
- Disabling settings that are unnecessary for your use case (e.g., UVs, normals, previewing, end caps, etc.) is a small performance improvement that could be worth it if you're generating many wires simultaneously.
- It is best to enable/disable settings before adding any points, as the existing geometry will be re-drawn which is not ideal for performance if there already are lots of points.
- The `UV.y` coordinate on the mesh is its distance along the wire. You can remap this in a shader to get a length attribute along the wire! Here is a quick Code Node one-liner where `wireLength` is retrieved with `getLength()`, and `range` is a custom value:
  ```glsl
  float alongLine = system.remap(system.getSurfaceUVCoord0().y, wireLength, wireLength - range, 0., 1.);
  ```
- The UVs of the start/end caps are planar projected, and their x and y values are in different ranges so they can be separated from the rest of the mesh in shader!
  - `UV.x` is [1, 2] for the starting cap and [2, 3] for the end cap
  - `UV.y` is in range [wireLength, wireLength + 1], so shaders using the wire length for vertex displacement will still look right
- To use the custom `radius` attribute in shaders, enable it using `enableRadiusAttribute(true)` and then use the `Custom Vertex Attribute` node set to `radius` (float). The radius is given in local space. If you want to use a constant radius along the full wire, it is better for performance to simply use a static value in the shader instead.
