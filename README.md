# Alita

Zero Touch Dynamo package and geometry kernel for BIM workflows.

| Item | Today | Target |
|---|---|---|
| Host | Dynamo 4.1.1 (Revit 2027) | Dynamo 4.x (Revit 2027) |
| Runtime | .NET 10 | .NET 10 |
| Package name | Alita | Alita |
| Version | 1.1.1 | 1.1.1 |

Solution: `Alita_C.sln`

| Project | Output | Purpose |
|---|---|---|
| `Alita.Kernel` | `Alita.Kernel.dll` | Geometry, math, units. No Dynamo, no Revit. |
| `Alita.Dynamo` | `Alita.dll` | Zero Touch nodes shown in the Dynamo library. |
| `Alita.CEF` | WinExe | CEF/WinForms host. Not a Dynamo node library. |
| `Tester` | net6.0 console | Local tests. |
| `WinFormTester` | WinForms | Local tests. |

## Install in Dynamo 4.x (Revit 2027)

Copy the package to `%AppData%\Dynamo\Dynamo Revit\27.0\packages\Alita\` (Revit 2027 uses `27.0`, not `4.0`). Close Revit, then reopen Dynamo.

Mark that `27.0` folder as **trusted** in Dynamo: **Preferences → Manage Node and Package Paths**. Dynamo 4 only loads node DLLs from trusted locations.

Nodes do **not** appear next to Geometry / Math. They live under **Add-ons → Alita**. Use the **+** beside Add-ons if Alita is missing from that list.

```
Add-ons
  Alita
    Alita
    Data
    FileSystem
    Geometry
      ConvexHull
      Curve
      Geometry
      Line
      Point          Origin, BelongsTo, Collinear, IsInside, SortedXY, SortedYX, …
```

Search also works: `Point.Origin`, `Line.Angle`, `Circle.ByCenterRadius`.

Cursor rules live in `.cursor/rules/`. Roadmap: [ROADMAP.md](ROADMAP.md).

---

# Dynamo nodes

Library path is `Namespace.Class.Method`. Hidden types (`[IsVisibleInDynamoLibrary(false)]`) are not listed as user nodes.

Private constructor on every Zero Touch class. Geometry types are `Autodesk.DesignScript.Geometry` unless noted.

## Alita.Geometry.Point

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `Origin` | — | `Point` | World origin `(0,0,0)`. |
| `Equal` | `point1`, `point2`, `tolerance=0.001` | `bool` | True if XYZ deltas are within tolerance. **Bug:** uses `<=` per axis, not absolute distance. |
| `Collinear` | `points`, `Tollerance` | `List<Vector>` | Intended collinear test. **Bug:** returns vectors, not booleans. |
| `IsInside` | `solid`, `point` | `bool` | `point.DoesIntersect(solid)`. |
| `BelongsTo` | `geometry`, `point`, `Tollerance=0.05` | `bool` | Point on/near curve, circle, rectangle, nurbs, polycurve, or generic geometry. |
| `SortToReference` | `points`, `point` | `List<Point>` | Sort by distance to a reference point. |
| `Unique` | `points`, `tolerance` | `List<Point>` | Deduplicate. **Bug:** adds a point when `Equal` is true. |
| `SortedXY` | `points` | `List<Point>` | Sort Y then X. Mutates the input list. |
| `SortedYX` | `points` | `List<Point>` | Sort X then Y. Mutates the input list. |

## Alita.Geometry.Line

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `Angle` | `line1`, `line2`, `counterclockwise=true` | `double` | Angle in degrees via arc length. |
| `MidLine` | `line1`, `line2` | `Line` | Midline of two parallel lines; `null` if not parallel. |
| `Line2DEquation` | `line` | `List<double>` | `{slope, yIntercept}` or `{∞, x}` for vertical. |
| `Distance` | `line1`, `line2` | `double` | 3D line-line distance via Kernel. |
| `MidPoint` | `line` | `Point` | Point at parameter 0.5. |
| `BiSectorAngle` | `line1`, `line2` | `double` | Half of `Angle`. |
| `Parallel` | `line1`, `line2`, `tolerance=1e-5` | `bool` | Angle ≈ 0° or 180°. |
| `Perpendicular` | `line1`, `line2`, `tolerance=1e-5` | `bool` | Angle ≈ 90° or 270°. |
| `ColinearSegments` | `line1`, `line2`, `tolerance=0.0001` | `bool` | Same 2D line equation. |
| `Equal` | `line1`, `line2`, `tolerance=0.0001` | `bool` | Start and end points match (not reversed). |
| `MakeUpward` | `line` | `Line` | Reverse if angle with +X ≥ 180°. |
| `MakeDownward` | `line` | `Line` | Reverse if angle with +X is in `[0, 180)`. |
| `SortedXY` | `lines` | `List<Line>` | Sort by start Y then X. Mutates input. |
| `SortByLength` | `lines` | `List<Line>` | Longest first. Mutates input. |
| `Unique` | `lines`, `tolerance=0.0001` | `List<Line>` | Drop duplicates. Unsafe while iterating. |
| `DrawParallel` | `line1`, `distance` | `Line` | `Offset(distance)`. |
| `DrawPerpendicular` | `line`, `point`, `length=10` | `Line` | Perp through `point` using GShark. |
| `MakeParallel` | `line1`, `line2` | `List<Line>` | Hidden stub (`null`). |
| `Showcoupleline` | `CoupleLine` | `List<Line>` | Kernel couple → two Dynamo lines. |
| `Showsegment` | `List<Segment>` | `List<Line>` | Kernel segments → Dynamo lines. |
| `CoupleLines` | `line`, `lines`, `distance` | `List<Line>` | Parallel neighbors within distance. |
| `MergeCollinearSegments` | `lines`, `distance` | `List<Line>` | Hidden stub (`null`). |
| `uniquelines1` | `lines` | `List<Line>` | Hidden prototype. |
| `uniquelines2` | `List<List<Line>>` | `List<List<Line>>` | Hidden prototype. |
| `uniquelines3` | `List<List<Line>>` | `List<List<Line>>` | Hidden prototype. |
| `Proximity` | `line`, `lines` | `List<Line>` | Nearest parallel neighbors (experimental). |
| `test1` | `line`, `lines`, `distance` | `List<Line>` | Hidden prototype. |
| `test2` | `lines`, `distance` | `List<Line>` | Hidden prototype. |
| `test3` | `lines`, `thickness` | `List<List<Line>>` | Hidden prototype. |

`test1` / `test2` / `test3` / `uniquelines*` / `MakeParallel` / `MergeCollinearSegments` are hidden from the Dynamo library (`[IsVisibleInDynamoLibrary(false)]`).

## Alita.Geometry.Vector

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `CoDirection` | `v1`, `v2` | `bool` | Normalized dot ≈ 1. |
| `Perpendicular` | `v1`, `v2` | `bool` | Normalized dot ≈ 0. |
| `CounterDirection` | `v1`, `v2` | `bool` | Normalized dot ≈ -1. |

`ToLine` (display vector as colored line) is commented out.

## Alita.Geometry.Curve

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ToLine` | `curve` | `Line` | Chord from start to end. |

## Alita.Geometry.Geometry

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `Unique` | `geometries`, `tolerance` | `List<object>` | Hidden. Infinite recursion — do not use. |
| `Equal` | `geo1`, `geo2`, `tolerance` | `bool` | Dispatches to Point/Line `Equal`. Other types → `false`. |

## Alita.Geometry.PolyCurve

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ToLine` | `polycurve` | `Line` | Chord from start to end. |
| `Perimeter` | `polycurve` | `double` | Sum of segment lengths. |
| `Area` | `polycurve` | `double` | Patch area if closed; `-1` if open. |

## Alita.Geometry.Polygon

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ToLine` | `polycurve` | `Line` | Chord (input type is PolyCurve). |
| `Perimeter` | `polycurve` | `double` | Same as PolyCurve.Perimeter. |
| `Area` | `polycurve` | `double` | Patch area if closed; `-1` if open. |
| `Center` | `polygon` | `Point` | `polygon.Center()`. |
| `Edges` | `polygon` | `List<Curve>` | Boundary curves. |
| `Angles` | `polygon` | `List<double>` | Consecutive edge direction angles. |
| `Vertex` | `polygon` | `List<Point>` | Start of each edge. |
| `MidPoints` | `polygon` | `List<Point>` | Midpoint of each edge (assumes lines). |
| `OrientedBoundingRectangle` | `polygon` | `Rectangle` | Min-area OBB via convex hull. |
| `ConvexHull` | `polygon` | `Polygon` | 2D convex hull of vertices. |
| `BoundingCircle` | `polygon` | `Circle` | Small enclosing circle. |
| `Diameter` | `polygon` | `List<Line>` | Non-adjacent corner pairs. |
| `IsConvex` | `polygon` | `bool` | **Bug:** almost always returns `true`. |
| `Triangulate` | `polygon` | `List<Polygon>` | Hidden stub (`null`). |

## Alita.Geometry.ConvexHull

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ConvexHull2D` | `points` | `Polygon` | Monotone-chain hull after flattening to XY. |
| `BoundingCircle` | `points` | `Circle` | Small enclosing circle, mapped back to original plane. |
| `OrientedBoundingRectangle` | `points` | `Rectangle` | Rotating-calipers style OBB. |

## Alita.Geometry.Shape.Circle

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ByCenterRadius` | `center=Origin`, `radius` | `Circle` | Circle on XY through center. |
| `ByPlaneRadius` | `plane=XY`, `radius` | `Circle` | Circle on plane. |
| `ByBestFitThroughPoints` | `points` | `Circle` | Best-fit circle. |
| `ByThreePoints` | `p1`, `p2`, `p3` | `Circle` | Circle through three points. |
| `Radius` / `Diameter` / `Center` / `Perimeter` / `Area` | `circle` | number or `Point` | Queries. |
| `Plane` | `circle` | `Plane` | **Bug:** `List.Append` does not add items; best-fit may fail. |
| `Topology` | `circle` | multi | Center, Radius, Diameter, Perimeter, Area, Plane, Bounding Box. |
| `OBB` | `circle` | `PolyCurve` | **Bug:** `List.Append` used; bounding box likely empty. |
| `Equal` | `circle1`, `circle2` | `bool` | Same center and radius (reference equality on points). |
| `Same` | `circle1`, `circle2` | `bool` | Same radius only. |

Class-level `[NodeName]` / `[InPortNames]` attributes are NodeModel leftovers. Remove them; they do not apply to Zero Touch.

## Alita.Geometry.Shape.Rectangle

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ByCenterLength` | `center=Origin`, `length` | `Polygon` | Regular 4-gon (square) from a circle. |
| `ByPoints` | `points` | `Polygon` | Polygon from points. |
| `Perimeter` / `Center` / `Area` / `Plane` | `rectangle` | number / `Point` / `Plane` | Queries. |
| `Diameter` | `rectangle` | `List<Line>` | Two diagonals. |
| `SymmetryAxis` | `rectangle` | `List<Line>` | Midpoint-to-midpoint axes. |
| `AngleWithXAxis` | `rectangle` | `double` | Angle of lowest long edge vs +X. |
| `AngleWithXYPlane` | `rectangle` | `double` | Normal vs XY. |
| `LowestLong` / `LowestShort` | `rectangle` | `Line` | Lowest long/short edge in XY sort. |
| `MinPoint` / `MaxPoint` | `boundingrectangle` | `Point` | Sorted-XY first / last corner. |
| `Remake` | `rectangle` | `Dictionary` | Ordered `Points` + rebuilt `Rectangle`. |
| `Topology` | `rectangle` | `Dictionary` | Length, Width, Ratio, Area, Perimeter, Diameters, axes, angles, center, plane, edges, min/max. |

`Topology` / `Remake` should use `[MultiReturn]`.

## Alita.Geometry.Shape.Square

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ByCenterLength` | `center`, `length` | `Polygon` | Regular 4-gon. |
| `ByPoints` | `points` | `Polygon` | Polygon from points. |
| `Perimeter` / `Center` / `Area` | `polygon` | number / `Point` | Queries. |

## Alita.Geometry.Shape.Triangle

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ByCenterLength` | `center=Origin`, `length` | `Polygon` | Delegates to `ByCenterHeight`. |
| `ByCenterHeight` | `center=Origin`, `height` | `Polygon` | Regular 3-gon. Radius = `height * 2`. |
| `ByPoints` | `points` | `Polygon` | Triangle from points. |
| `Perimeter` / `Center` / `Area` / `Plane` | `polygon` | queries | Standard. |
| `Edges` / `Angles` / `Vertex` / `MidPoints` | `polygon` | lists | Delegates to `Polygon`. |
| `Medians` | `polygon` | `List<Line>` | Vertex to opposite midpoint. |
| `MedianTriangle` | `polygon` | `PolyCurve` | Midpoint triangle. |
| `Centroid` | `polygon` | `Point` | Median intersection. |
| `Circumcenter` | `polygon` | `Circle` | Best-fit through vertices. |
| `NinePointCircle` | `polygon` | `Circle` | Best-fit through midpoints. |
| `Topology` | `polygon` | multi | Center, Centroid, Orthocenter, Circumference, Area, Plane, Vertex, Edges, Angles, MidPoints, Medians, BiSectors, Heights, Circumcenter, InCicrle, NinePointCircle, MedianTriangle. |

**Known:** `Orthocenter`, `BiSectors`, `Heights`, `InCicrle` reuse centroid/medians/circumcenter. They are placeholders.

## Alita.Is.Geometry.Shape.Triangle

Boolean classifiers. Input is a Dynamo `Polygon` converted to Kernel `Triangle`.

| Node | Output meaning (as coded) |
|---|---|
| `Scalene` | Returns Kernel `IsIsosceles` — **name/logic mismatch**. |
| `Equilateral` | `IsEquilateral` |
| `Acute` | `IsAcute` |
| `Right` | `IsRight` |
| `Obtuse` | `IsObtuse` |
| `Oriented` | `IsOriented` |

## Alita.Data.Number

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ToWords` | `number`, `locale=""` | `string` | Integer to words (Humanizer). |
| `ToOrdinalWords` | `number`, `locale=""`, `fullWord=false` | `string` | `1` → `1st` or `first`. |
| `ToRoman` | `number` | `string` | Roman numerals. |
| `ToHeading` | `number`, `fullHeading=false` | `string` | Compass heading. |
| `ToHeadingArrow` | `number` | `string` | `↑ → ↓ ←`. |

## Alita.Data.String

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `LongestCommonSubstring` | `string1`, `string2` | `string` | LCS. |
| `ParseRegularExpression` | `stringToReplace`, `regexString`, `replacement` | `string` | **Bug:** ignores `regexString` and `replacement`; always strips non-alphanumeric. |
| `ToTitle` | `str` | `string` | Title case. |
| `ToSentence` | `str` | `string` | Sentence case (`.` or `\r` aware). |
| `ToQuantity` | `str`, `quantity` | `string` | `"item"` + count. |
| `Truncate` | `str`, `length`, `truncationString="…"` | `string` | Truncate. |
| `Pluralize` / `Singularize` | `str` | `string` | Humanizer. |
| `Titleize` / `Pascalize` / `Camelize` / `Underscore` / `Dasherize` | `str` | `string` | Case transforms. |
| `Humanize` | `obj` | `string` | String, `DateTime`, or `TimeSpan`. |
| `FormatWith` | `str`, `args` | `string` | Format string. |
| `MOcKtExt` | `str` | `string` | Alternating mock case. |

## Alita.FileSystem

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `Compress` | `directoryName` | `string` | Zip folder to `{directoryName}.zip`. |

## Alita.Help.About

| Node | Output | What it does |
|---|---|---|
| `AboutAlita` | `string` | Package name and assembly version. |

## Alita.Helpers / ImportExport / System

| Class | Node | Inputs | Output | What it does |
|---|---|---|---|---|
| `Helpers` | `Toggle` | `obj`, `toggle` | `object` | Pass list through if true; else empty string list. |
| `Helpers` | `ThisOrThat` | `obj1`, `obj2`, `toggle` | `object` | Pick list 1 or 2. |
| `ImportExport` | `ScreenshotMainWindow` | `filepath` | void | JPEG of primary screen. |
| `System` | `CurrentUserTempFolder` | `refresh=true` | `string` | `%TEMP%`. |
| `System` | `CurrentUserAppData` | `refresh=true` | `string` | `%AppData%`. |
| `System` | `CurrentUserDomainName` | `refresh=true` | `string` | Domain. |
| `System` | `SendToClipboard` | `str` | void | Clipboard. |
| `System` | `CurrentUserName` | `refresh=true` | `string` | Windows user. |
| `System` | `MachineName` | `refresh=true` | `string` | Computer name. |
| `System` | `JiggleMouse` | `runIt=false`, `interval=0` | void | Hidden. Moves cursor on a timer. |

## Alita.GenerativeDesign

| Node | Inputs | Outputs | What it does |
|---|---|---|---|
| `PackViewports` | `container`, `viewportRectangles`, `viewportIds` | `viewportsThatFit`, `proposedLocations`, `viewportRectangles` | Pack rectangles into a titleblock-like container. **Bug:** container height uses X span twice. |
| `RandomDistribution.NextGaussian` | `standardDeviation=1` | `double` | Gaussian in `(-1, 1)`. |

`CygonRectanglePacker` / `RectanglePacker` / `OutOfSpaceException` are hidden helpers.

## Alita.Shared.BIMTrack

| Node | Inputs | Output | What it does |
|---|---|---|---|
| `ParseHubUser` | `json` | `List<BIMTrackHubUser>` | Deserialize BIM Track hub users. |

`BIMTrackUser` is hidden. `BIMTrackHubUser` is visible (DTO).

## Alita.UI (NodeModel, not Zero Touch)

| Node | What it does |
|---|---|
| `TinSurface DropDown` (`DummyDropdown`) | Hidden dummy. |
| `Drop Down Example` (`DropDownExample`) | Hidden example. |
| `Elevation Types` (`ElevationTypesDropDown`) | Hidden leftover GIS dropdown. |
| `Elevation Types` (`ElevationTypesDropDown`) | Hidden leftover GIS dropdown. |

`CustomGenericEnumerationDropDown` in `UI/1.cs` is a base class. `UI/2.cs` and `UI/3.cs` are supporting UI code.

## Hidden / not in library

| Type | Reason |
|---|---|
| `Alita.General.ConvertList` | Helper, hidden. |
| `Alita.Geometry.Convert` | Dynamo ↔ Kernel ↔ GShark converters. |
| `Alita.Geometry.Geometry.Unique` | Broken recursion; hidden until P3 fix. |
| `Alita.Geometry.Line` prototypes | `test1/2/3`, `uniquelines1/2/3`, `MakeParallel`, `MergeCollinearSegments`. |
| `Alita.Geometry.Polygon.Triangulate` | Stub. |
| `Alita.System.JiggleMouse` | Not for package release. |
| Dummy UI | `DummyDropdown`, `DropDownExample`, `ElevationTypesDropDown`. |
| `Alita.Maths.MarkovChain` | Commented out. |

## Kernel (not Dynamo nodes)

`Alita.Kernel` is consumed by nodes. It is not loaded as a node library unless listed in `pkg.json`. Types include Point, Vector, Line, Segment, CoupleLine, Circle, Ellipse, Ellipsoid, Sphere, Box, Plane, Ray, Triangle, Tetrahedron, Polygon, Matrix, Quaternion, Rotation, units (Length, Area, Volume, …), `ExpressionEvaluator`, and Maxima helpers.

---

# Build (current tree)

```powershell
nuget restore Alita_C.sln
msbuild Alita_C.sln /p:Configuration=Debug /p:Platform="Any CPU"
```

Needs Visual Studio 2022 / Build Tools, .NET Framework 4.8 targeting pack, and NuGet packages under `packages/`. Dynamo 4.x migration steps are in [ROADMAP.md](ROADMAP.md).
