# Title

Contributors: Andrew Pullin, Austin Buchan, Duncan Haldane

## Definition of Need

Given the geometric description of one or more part files, we want to efficiently pack these parts into regions. This maps to out assumption of automated 2D manufacturing from sheet goods, targeting laser cutters, CNC routers, and drag knife cutters.  

Many CAD file formats already exist. Rather than try to find one to suit out exact needs, a minimal implementation of clear encoding of the geometry description and task description will be made in the JSON format. This format is chosen due to easy loading, saving, and correspondence with the Python programming language.  

It is worth reflecting the relative simplicity of the output, contrasted to the huge space in which is embedded: for a layout of _n_ total part instances, the solution can entirely be described by a single point in **R**^(4n). From our practical programming view, this is equivalent to a tuple of length $4n$, or rather $n$ 4-tuples of **{** part type, x, y, orientation **}**.

### Target

The ultimate goal for the implementation of this target to to produce a tool that is useful, open, robust, and reliable. An immediate comparison could be drawn to the slicing engines in consumer grade 3D printers that are now widely available. Powerful slicing and toolpath generation code runs quietly behind the translation of any STL to G-Code, and the implementation is now so good that novice users can instantly benefit from the silent invokation by some front end tool.  

Graphics Processing Unit (GPU) and associated drawing pipelines offers the potential to signifigantly accelerate some solution methods. Rasterization is a given and benefits immediately from parallelism in the hardware. Through use of already extant methods, such as occlusion queries, and creative use of depth and alpha blending buffers, the scoring of scoring of solutions could be greatly accelerated.

## Proposed Formats

### Description of Parts

The following listing shows a JSON format of a description of parts and their associated geometries. Each part is defined as a polygon with a boundary implemented as a vertex array, where closure between the last and first vertices will be assumed. Each shape will have exactly one bounary vertex array of at three or more points.  

Shapes will be permitted to have cut-outs or "holes" in them. Any number of holes will be supported. Holes may not intersect with each other, although such a case should be easy to reconcile via a "sanitation" pass, where all holes are union-ed together.  

A parse-time check of the validity of each shape will be neccesarily. The [shapely library for Python](http://toblerity.org/shapely/manual.html)should have the entirety of this implemented. It should suffice to implement each shape as a \textit{shapely.geometry.Polygon} object in python, and a validity test will be a given.

Sample parts geometry description:

    {  "parts":[
            {
                "outline": [[0.0,0.0],[1.0,0.0],[1.0,1.0],[0.0,1.0]],
                "holes":   [],
                "vertexLabels":  ["A", "B", "C", "D"],
                "edgeLabels": ["first", "second","third","fourth"],
                "type":    "Polygon",
                "id"  :    "square"
            },
            {
                "outline": [[0.2,0.0],[0.4,0.0],[0.3,2.0]],
                "holes":   [],
                "vertexLabels":  ["A", "B", "C"],
                "edgeLabels": ["first", "second","third"],
                "type":    "Polygon",
                "id"  :    "triangle"
            }
        ]
    }

### Description of the Task

The following listing shows a JSON format for specifying a layout task to be computed.\\

All shapes and the overall boundary shape are defined in external files, and referenced here. This is intended to be an intermediate file format, and one that will ultimately be hidden from the user.\\

PROMPT: Should this be handled more like an invocation of gcc? gcc is passed all object code for the link step.

Sample layout task:

    { "Layout":{
            "stock": {
                "id":"polyID",
                "source":"22x28_cardstock.txt"
                },
            "elements":[
                {"source":"shape1_jb.txt", "id":"triangle", "count":5},
                {"source":"shape1_jb.txt", "id":"square", "count":3}
                ]
        }
    }

### Description of a Solution

The following listing shows what the representation of the output of the solver could look like. Parts are instantiated and placed from their source design files.

The numbers for this solution are entirely invented. A forthcoming test will be a function that can plot or render this solution as-is, and export the geometry to a manufacturable format, such as DXF. The \textit{ogr2ogr} tool from the GDAL project should be able to handle this with a minimum of glue code.  

Sample solution output:

    {   "genesis":"layout1.txt",
        "opts":"-gpu",
        "solution":[
                {"source":"shape1_jb.txt", "id":"triangle", "origin":[1.1, 2.2], "rotation":-30.0},
                {"source":"shape1_jb.txt", "id":"triangle", "origin":[3.0, 4.0], "rotation": 30.0},
                {"source":"shape1_jb.txt", "id":"triangle", "origin":[5.0,6.0],  "rotation":-179.0},
                {"source":"shape1_jb.txt", "id":"triangle", "origin":[7.0, 3.0], "rotation":179.0},
                {"source":"shape1_jb.txt", "id":"triangle", "origin":[1.0, 4.3], "rotation":22.2},
                {"source":"shape1_jb.txt", "id":"square", "origin":[9.9, 9.9],   "rotation":-1.0},
                {"source":"shape1_jb.txt", "id":"square", "origin":[10.9, 10.9],   "rotation":-10.0},
                {"source":"shape1_jb.txt", "id":"square", "origin":[11.9, 11.9],   "rotation":-20.0}
            ]
    }

## Example Usage

Tools will be written mostly or entirely in Python, for ease and portability.

## Solution Methods

### Bottom Left Fitting

The bottom left fitting (blf) algorithm is expected to work when the total layout is not largely constrained.

###Simplistic Simulated Annealing

A simple simulated annealing (SA) approach will be easy to implement as a first test. \textit{shapely} will allow for easy computation of the area of intersections between polygon objects. Taking the total overlap area as the "score", we simply need to iterate and lock in on better solutions.  

The expected fatal flaw with this method is that the broad organization of parts in the layout will not be able to change, since it is likely that a higher "cost" configuration would have to be walked through to get such a reorganization.

### Simplistic Genetic Algorithm

Randomness, new solution paths. This is SA, but with major mutations in broad layout changes. Combining two good solution paths is not conceptually clear, and may not help.

### Attraction and Repulsion

Gravity, torque, spring networks.

### Hybrid Methods

Many hybrid methods are possible, and should be explored. For example, a process of blf with annealing of each incoming part, including the total bounded area currently consumed, would be reasonably expected to outperform simple blf.