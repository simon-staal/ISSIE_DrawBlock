# Inidividual Submission for tl319

## Admin and quick access links

*link to your teamN.md file*
[Common repo Team9 file](https://github.com/tomcl/hlp22docs/blob/main/Team9.md)

[Buswire (section 1)](src/Renderer/DrawBlock/BusWire.fs)

Section 1 on my file is lines: 246-856

Changes to the types used in Buswire were decided as a team with the people working on the other sections.

## Code Quality

**Highlights**
* The renderRadialWire function is particularly elegant as it creates a single react element for the entire wire, instead of segment by segment.
* The AbsSegments type is particularly useful for rendering wires, as it is suitable for each different type of rendering (radial, modern, jump). It also provides a logical conversion from the overall relative segment implementation that is compatible with the generation of react elements. 

## Analysis
### Bad function list

**segmentsToVertices**:
The original documentation of this function was misleading, as it stated it converted a wire to a connection when in reality it was converting a segment list into a list of vertices. It also used List.mapi when List.map would have sufficed.

**inferDirectionFromVertices**:
This function had no documentation whatsoever.

**xyVerticesToSegments**:
This function was made overcomplicated by the requirement for wires to have 7 segments. This meant that instead of simply the first and last segment of a wire being undraggable additional checks had to be done to decide whether any other segments were undraggable.

**issieVerticesToSegments**:
The documentation of this function was again misleading, as it stated it converted an issie Connection to a wire, when it reality it was converting it to a segment list.

**onSegment**, **orientation**:
Both functions are global, however they are both very small and are only used in one other function so could easily be a subfunction.

**segmentIntersectsSegment**:
The implementation of this function is poor. Using negative position values to indicate the routing method of a segment forces the function to get the absolute value. This is problematic as it limits segments to being in the top right quadrant in order for the function to work properly. It would also be simpler to check whether two segments intersect by finding the theoretical intersection point (as segments can only be horizontal or vertical), and then checking whether that point lies between the coordinates of both segments.

**makeSegPos**:
This has similar problems to onSegment and orientation. It is a small function that could be defined locally without causing significant clutter in the code.

**renderSegment**:
This function could have been simplified by breaking up subsections of its functionality into external functions, as it is currently relatively unreadable. Furthermore, its implementation could be improved by using List functions as opposed to repeatedly appending.

**singleWireView**, **view**:
Both functions are nearly entirely undocumented, this is especially bad as these two functions dictate the rendering and so their ease of use and understanding are critical.

### Other problems

The segments type:
* Contains a start and end position on every single segment, duplicating a large amount of information.
* The method to determine routing of a segment is whether or not it is negative which is extremely unintuitive and leads to messier code later on. 
* This was improved by changed segments to have a relative representation, only storing a length. Every segment is by defintion and a right angle to the previous segment, and the sign of the lenght indiciates whether the wires runs in an increasing or decreasing direction which is very logical.
* Segments have a 'mode' attribute, which is either 'Manual' or 'Auto' that indicates whether or not it is being manually routed.
* Wire contains both the start and final positions which allow us to produce absolute XY positions of segments whenever they are needed.

Segment list of wires:
* Originally there were only 2 types of segment lists, 3 and 5 seg, to route between various symbols. These were also implemented with the assumption of 7 segments in any given wire. 
* This led not only to hard coding that made inclusion of new cases significantly harder, but also required extra data in the situation that a wire could be represented with fewer segments.
* This was changed such that wires can have a variety of length of segment lists. 

Unused / Made redundant functions:
* Many of the original functions were either never used, or could very easily be made redundant. 
* segmentIntersectsSegment as well as a variety of global subfunctions it uses (which have no need to be global) were made redundant due to changes in Implementation.
* MapToSortedList was never used.
* MemoOf was never used.
* distanceBetweenTwoPoints was never used.

### Analysis of how/why code works

* Will demo all orientations of end points.
* Will demo radial wire rendering
* Will demo jump wire rendering
* Will demo modern wire rendering

Final function list:
segmentsToVertices - Implicitly shown in demo - changes from previous functionality are simply refactoring so that it works using the new relative segments.

makeInitialWireVerticesList - Explicitly shown in demo - has a manual case for all different endpoint orientations in all quadrants relative to the startpoint. Each different orientation and position can be shown. 

xyVerticesToSegments - Implicitly shown in demo - Calculates the length for each segment by calculating the difference in both X and Y. It then creates segments using this distance. It also ensures that the first and last segment are not draggable, preserving original functionality. 

issieVerticesToSegments - Implicitly shown in demo - converts the vertices from issie (two floats) to an XYPos that can then be used with xyVerticesToSegments to create segments as before.

makeInitialSegmentsList - Implicitly shown in demo - Creates the verticeslist by using makeinitialWireVerticesList with the correct endpoint orientation, this list is then passed to xyVerticesToSegents to create all initial segment list.

extractConnection - Implicitly shown in demo - Same functionality as original

extractConnections - Implicitly shown in demo - Same functionality as original

renderRadialWire - Explicitly shown in demo - Creates an SVG command that renders an entire radial wire apart from the first move and the final line.

renderModernSegment  - Explicitly shown in demo , have to manually test however as requires section 3 that is not yet implemented - renders a segment as well as any intersections along that segment. The intersection locations are dictated by the IntersectCoordinateList in the AbsSegment, in the case that this list is empty it will instead produce a simple straight line.

renderJumpSegment - Explicitly shown in demo , have to manually test however as requires section 3 that is not yet implemented - renders a segment as well as any jumps along that segment, is capable of supporting multiple jumps as in original function. The jump locations are dictated by the IntersectCoordinateList in the AbsSegment, in the case that this list is empty it will instead produce a simple straight line.

singleWireJumpView - Explicitly shown in demo , have to manually test however as requires section 3 that is not yet implemented - calls renderJumpSegment for each segment of the given wire as well as rendering the bit width text.

singleWireModernView - Explicitly shown in demo , have to manually test however as requires section 3 that is not yet implemented - calls renderModernSegment for each segment of the given wire as well as rendering the bit width text.

singleWireRadialView  - Explicitly shown in demo - Uses the SVG command to generate the ReactElements that renders the entire radial wire as well as rendering the bit width text. There is some incorrect behaviour in this function, due to the fact that the rendered arcs are all of radius 5. Specifically if there are 3 segments and the middle segment has a length of less than 5 the two arcs will pass one another and subsequently be joined by another straight line. This however is a relatively minor problem as the issue is likely to only come up when specifically attempting to recreate it.

view - Implicitly shown in demo - Calls singleWire_____View on each wire in the model, according to the Model.Type

## Extensions

1.  Radial display type for wires

     a. singleWireRadialView this is a new function that creates a custom path from a string created in renderRadialWire and makes an SVG element from it

     b. renderRadialWire this is a new function that generates a path command string according to the vertices of the wire
     
     c. view changed to support switching between wire displays according to Model.Type 
     (NOTE: the code to change the Model.Type is currently not correctly implemented from section 3, as such it will always be the Jump type, in order to test different display types line 844 can be changed to different singleWireRadialView props)

2. Modern display type for wires
     a. singleWireModernView this is a new function that creates a react element that contains all of the segments and intersections for any given wire. 
     
     b. renderModernSegment this is a new function that creates the react element of any given segment as well as circles for any intersection. The location of these circle is passed via the Segment.IntersectCoordinateList
     (NOTE : at time of writing the code to populate the IntersectCoordinateList of segments  is not fully functional as section 3 has not completed it, in order to test this we can manually place intersections at fixed distances on every segment. This does cause incorrect rendering in the case that the segment is shorter than the manually entered distance, however when IntersectCoordinateList is populated properly in section 3, this scenario would become impossible. Thus this problem would be avoided and it would have entirely correct functionality)
     
     c. view changed to support switching between wire displays according to Model.Type 
     (NOTE: the code to change the Model.Type is currently not correctly implemented from section 3, as such it will always be the Jump type, in order to test different display types line 844 can be changed to singleWireModernView props)


3. Arbitrary orientations of endpoint
   a. makeInitialWireVerticesList the overall implementation was simplified to support any orientation of the endpoint in any location, it also simplified the logic such that it directly passes the complete and final list of vertices to ensure a clear division of functionality.

   b. makeInitialSegmentsList as a result of makeInitialWireVerticesList being simplified a significant amount of the logic for this function could be removed so that it directly does what it states, creates an initial segment list. 

   c. Each singleWire_____View function was also changed to correctly render the bit width in relation to the new orientation.

   d. these functions were used in conjunction with section 2 in order to allow for an arbitrary orientaiton of starting point as well.
   