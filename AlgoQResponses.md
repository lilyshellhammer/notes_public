# Question:
Write pseudocode for an (efficient and correct) algorithm that answers each of the following questions for a pair of BVHs A and B. (You are allowed to assume that the query is available for any kind of leaf nodes.)  Give the time complexity of each algorithm.  For full points, you will need to write legibly in complete sentences.
1. [Do A and B intersect, and where?](#area-intersection)
2. [Do the boundaries of A and B intersect, and where?](#boundary-intersection)
3. [Does A contain B?](#containment)
4. [How close are A and B to being tangent, and at what pair of points?](#tangency)
5. [What is point on B closest to any point on A?](#closest-distance)

## Overview
In this document, I summarize methods for checking area intersection, boundary intersection, containment, tangency, and closest distance using (mainly) BVHs. The two methods I write up that do not deal specifically with BVH's are the polygon containment test and the level set containment test. 
Previously, we calculated a discrete representation for the boundary at the beginning of our optimization and recalculated the level set and the BVH at every step. The new methods (other than the level set containment test) don't require a level set (the black box solution to the overlapping/containing/distanceTo nodes *may*, but not BVH traversal) and don't need recalculation of the boundary or the BVH unless the object is rotated. The BVH can be scaled and rotated.

### BVH Calculation
In every example, the main BVH's are calculated as such:
Pseudocode
```
createBBoxNew(grid, xmin, ymin, xmax, ymax)
loop over grid between mins and maxes
  find new max and min x and y 0's or negative numbers //makes a bounding box tight around the shape within the passed
                                                  //in coordinates
  return bbox
  
bvhCreator (node, grid, position of bbox, iter)
  createBBoxNew(grid, position of bbox)
  increase iter (determines X or Y split)
  if the leaf nodes would be smaller than what is allowed
    set as leaf Node
  else if iter == even // X split
    set current node as not a leaf node
    call bvhCreator on the right and left, dividing the bbox in half along X axis
  else if iter == odd // Y split
    set current node as not a leaf node
    call bvhCreator on the right (bottom) and left (top), dividing the bbox in half along Y axis
    
```
Close-to-real code
```C++
struct node{
  bool isLeaf;
  node* left, right;
  BBox bbox;
};

void bvhCreator(node n, grid g, int xmin, int ymin, int xmax, int ymax, int iter){
  int res = max( xmax - (xmax - xmin)/2, ymax - (ymax - ymin)/2 ); //find if leaf nodes are smaller than the smallest allowed nodes
                                        //this can be reformulated to any specific size def's of current or child nodes
  iter++;
  n.bbox = createBBox(grid a, int xmin, int ymin, int xmax, int ymax);
  
  if(res < smallestBboxSize)
    n.isLeaf = true;
  else if(iter % 2 == 0)          //X axis split
    n.left = new node;
    n.right = new node;
    n.leaf = false;
    bvhCreator(n.left, g, xmin, xmax - (xmax-xmin)/2, ymin, ymax, iter);
    bvhCreator(n.right, g, xmax - (xmax-xmin)/2, xmax, ymin, ymax, iter);
  else                         //Y axis split
    n.left = new node;
    n.right = new node;
    n.leaf = false;
    bvhCreator(n.left, g, xmin, xmax, ymin, ymax - (ymax-ymin)/2, iter);
    bvhCreator(n.right, g, xmin, xmax, ymax - (ymax-ymin)/2, ymax, iter);
}
```
Time Complexity
O(logn)

## Area Intersection
If A and B shapes intersect, there exists some area of both that overlap. Since all area in a shape is contained within at least one leaf node bounding box, that means the leaves containing the overlapping area are at least overlapping. If no leaf nodes intersect, this contradicts what we know about how bboxes and BVH are built.
In this function we want to know two things. Does area intersection happen and at which places? In this case I am returning just the bboxes where this happens as a list of coordinates where this happens could be extremely long and wouldn’t provide us with any critical information (for now, we may want to change how we return the PLACE of the intersection)

Pseudocode
```
bool overlap(bbox a, bbox b) //if 
  return (a.right < b.left || a.left > b.right || a.top < b.bottom || a.bottom > b.top)

coord* findIntersection (Node a, Node b) //top level function call will return true if any leaf nodes overlap
 if overlaps(a.bbox, b.bbox)
    if they are both leaves
          call BLACKBOX, returns bbox or empty list depending on if there is overlap
    if both have children
         if any calls to findIntersection(one's children, other's children) return a list of bbox
            append all valid returned bboxes together and return list
    if only one has children
        if any calls to findIntersection(one's children, other node) return a list of valid bbox
            append all valid bbox together and return list
  else no overlap so return []


```
Close-to-real code
```C++
******** INCOMPLETE
bool findIntersection(Node a, Node b){
  if(isLeaf(a) && isLeaf(b)){
    if (overlaps(a.bbox, b.bbox)){
      CALL BLACKBOX
      return true  //this is where we would use the checkSmallestBBoxOverlap blackbox
    }
    else return false;
  }
  if(overlaps(a.bbox, b.bbox){
    if(!isLeaf(a), !isLeaf(b))
        return( findIntersection(a.right, b.right) ||
                findIntersection(a.left, b.right ) ||
                findIntersection(a.right, b.left ) ||
                findIntersection(a.left, b.left )) ); //if there is one of children returns true, return true      
    else if(isLeaf(a), !isLeaf(b)) //if a is leaf but b isn't
        return( findIntersection(a.right, b) ||
                findIntersection(a.left, b ) );
    else if(!isLeaf(a), isLeaf(b)) //if b is leaf but a isn't
        return( findIntersection(a, b.right) ||
                findIntersection(a, b.left ) );
  } else return false;
}
```
  
## Boundary Intersection
If the boundaries of A and B intersect, there exist boundary-containing leaf nodes that overlap. 
The bounding boxes with only "inside" area don't tell us anything about boundary behavior. My proposed solution to this is to "flag" leaf nodes with boundaries. This will be helpful for closest distance and tangency later in the document. When we are checking for overlap, we use the same findIntersection function above except we only return true on leaf nodes that are flagged as containing boundary points. 
*** Note: edge case for when line intersects when passing directly through a corner or along a boxes edge. Need to make sure the way we count a box as having a boundary would solve for intersection in these cases. 

Pseudocode
```
createBBoxNew(grid, xmin, ymin, xmax, ymax)
loop over grid between mins and maxes
  find new max and min x and y coords for 0's or negative numbers // makes a bounding box tight around the 
                                                                  // shape within the passed in coordinates
  if there is a 0 anywhere in the grid
    mark bbox as a boundary bbox
  return bbox
  

bvhCreatorNew(node, grid, position of bbox, iter)
  bbox = createBBoxNew (size)
  increase iter (determines X or Y split)
  if the leaf nodes would be smaller than what is allowed
    set as leaf Node
  else if iter == even // X split
    set current node as not a leaf node
    call bvhCreator on the right and left, dividing the bbox in half along X axis
  else if iter == odd // Y split
    set current node as not a leaf node
    call bvhCreator on the right (bottom) and left (top), dividing the bbox in half along Y axis


coord* findIntersectionNew (Node a, Node b) //top level function call will return true if any leaf nodes overlap
  if overlaps(a.bbox, b.bbox) AND a and b are boudary containing nodes
    if they are both leaves
          call BLACKBOX, return coords
    if both have children
         if any calls to findIntersectionNew(one's children, other's children) return a list of coord
            append all valid coords together and return list
    if only one has children
        if any calls to findIntersectionNew(one's children, other node) return a list of valid coord
            append all valid coords together and return list
  else no overlap so return []
  
```
## Containment
Here I present three possible solutions. Part 1 is about a method of BVH traversal I came up with for checking containment.
The second and third parts are the polygon containment test and the level set containment tests described in Keenan's blog post. 
## Part 1: Negative space BVH traversal
This is my proposed new solution to the containments problem. It requires an additional data structure, but would not require computing the level set, just having a discrete representation of the shape's boundary. The rationale for this method is that if shape A is NOT contained in shape B, there will be a part of A that overlaps with the negative space of B. 
In this solution:
First I check if bboxes contain each other, if not I force the boxes to move toward one another 
Once outermost shape’s bbox contains the inner shapes’s bbox, I create a BVH surrounding the whitespace of the outermost bounding box. (see pseudocode below)
The same function used is findIntersection for Area Intersection with the exception that the BVH (Node b) that is passed in will be the whitespace tree of B. 
A true return value means part of A is possibly not contained in B, and the BLACKBOX solution will tell us for sure. The good thing about this method is that we can use similar BVH traversal as area and boundary intersection and the black box solution for two overlapping leaf nodes will be similar.

```
enforceBBoxContainment(bbox a, bbox b)
  if !contains(a, b)
      push b toward a's center //in the optimizer this is returned as a distance function which is then minimized

createBBoxNegatives(grid, xmin, ymin, xmax, ymax)
loop over grid between mins and maxes
  find new max and min x and y coords for POSITIVE numbers // makes a bounding box tight around the whitespace around
                                                           // a shape within the passed in coordinates
  return bbox
```

## Part 2: Polygon Containment
Using bezier paths created by the front end, we can plot points that create a polygon containment of a bezier curve. If the polygon containment of one shape has at least one point inside a shape and the polygon's edges never pass across the containing shape's edges, the shape is contained. 
****** This could present a challenge as parsing bezier curves to get an enveloping shape could be tricky. I need to read more into it and see how time consuming this will be. 

Pseudocode
```
constructPolygon(svg file, position){
  points = parseSVG(file); // This could be complicated, *******
                           // we will need to present points in a way that makes it easy to create
                           // all the edges of a polygon like below
  polygon p
  for i in points
    add to p edges( createEdge(points[i], points[i+1]));
   return p;
}

bool ptInsidePolygon(x,y , B){
  rayLine = (x,y) + t*d         // d is a random ray direction
  for e in B.edges
    if intersects(rayLine, e)   // count the amount of intersections 
      count++;
  if (count % 2 == 0)
    return false    // if hit an even number of edges, we are not inside the object
  else
    return true     // else we are contained
}

bool intersect(edge, edge2){
  solve for places where edge == edge2
  if outside of edge's start and end points
    return false
  else return true    // there is an intersection of lines that make up polygon edges on the polygon border
}

bool insidePolygon(A, B){
 if !ptInsidePolygon(A.point, B)
    return false
 for e in A.edges
  for f in B.edges
    if !intersect(e,f)  // check intersection of every edge in the contained polygon against the other polygon edges
}
```

Runtime???

## Part 3: Level Set Containment
We have written about this a lot so I will give a summary and link to [other documents](https://docs.google.com/document/d/11fBrgEjwB_BAE1aItIxaY2eCK6nugDDjY1Cu4eygFVI/edit#heading=h.clnnojafen2w). 
Level sets are discrete representations of the eikonal equation for an implicit surface. The reason to use them is to easily work with this continuous function in a grid structure. The grid shows us how close to the surface we are at every cell in the grid and we can interpolate the cell's values to find the distance to the surface at points between cell centers. The level set also gives us the gradient, so we know not only how close to the surface we are, but also the direction of steepest descent (toward the surface). We can use level sets to tell us whether one shape is inside another (at every boundary or inside cell, aka <= 0 value, these cells overlap with boundary or inside values for the larger shape), and if not, the gradient direction in which to move the area that is not contained:
Pseudocode
```
findBoundary(picture){
  find size of picture, rows, cols
  create 2d int array
  //save 0s positions
  for x in rows
    for y in cols
      if arr[x][y] == black, set arr[x][y] to 0
      else set to 1.
  OR
  //save 0 positons in a 1d array
  for x in rows
    for y in cols
      if arr[x][y] == black, save in next index in arr
}

containsLevelSet(grid, rows, cols, outershape){
  for x in all black pixels
    if !ptInsidePolygon(x coordinates, outershape)
        return false
  return true
}
```
If we want to move a shape inside another shape if part of the first shape's area is not contained, we could use [fast sweeping or fast marching](http://brickisland.net/diagrams/2018/06/17/methods-to-find-distance-between-arbitrary-shapes-in-penrose-a-comparison-of-fast-methods-for-solving-the-eikonal-equation/). 

## Tangency
See [Closest Distance](#closest-distance) for background. To determine tangency, we will need to delve deeper than just a BVH solution, as tangency relies on boundaries touching at one point and the slope of both boundaries being the same at that point. The slope of the lines is something that can't be exactly calculated using BVH's but we have the building blocks to tell if the points are touching. Closest distance gives us a value for how close two objects are regardless of position but if we use these two functions in tandem I can see issues with enforcing tangency. Because closest distance returns more than one bboxes it could be expensive finding the closest point and if you have multiple equidistant points, which direction do you move an object toward? We can also force objects to overlap using [area intersection](#area-intersection) and then move the shapes to only be touching in one spot and rotated so the slopes at the points that are touching are equal. Without the black box method for how we will solve tangency between two leaf nodes I don't know which solution is best. This question needs more time to find an adequate answer.

```
findTangency (a, b)
  if they intersect 
    intersect places = intersects(a.bvh, b.bvh) 
    call black box tangency solver between the overlapping nodes
    
// What to do when there are multiple overlapping leaf nodes? Choose one and find a local minima? 
```


## Closest Distance
FFirst, I'll explain the naive method I came up with. I'm not sure how inefficient it is. It reduces the number of leaf nodes we are looking at to a small subset of closest leaf nodes and then calls the blackbox solution on all valid pairs. In this method, we compare the children of both objects with two possible goals:
- Find the two leaf nodes where boundary containing leafs overlap
BLACKBOX can solve for this case to see if distance == 0 or less, or if boundaries are just very close
- Find the closest boundary containing leaves The rationale here is that the closest few leaf nodes between shapes will contain the closest point and the smallest distance. I don't say only one leaf because the bbox is just an estimation, so the closest box might not have the closest point. 
As we iterate through the trees, if a bbox of a child's minimum possible distance to our shape is greater than the maximum possible distance of another child's bbox, the closest point cannot be in the first child. 
We know the maximum possible distance from a point to a bbox is the distance to the 2nd closest corner: 
We can utilize this to find the maximum distance between two boxes by comparing distance from corners: 
If the minimum distance of one child is greater than the max distance of another child, we ignore the first child because it isn't possible for the closest point to be in that bbox.

We iterate through the tree, trimming
Pseudocode
```
findClosestBBoxes(node a, node b){
  if both not leaves
    compare maxdist(a.right, b.right) to mindist (a.left, b.right)
    
    if any nodes have a max distance LESS than any others minimum distance
      ignore traversing that tree, only focus on the latter child
     
}
```

# Questions before send
- Should I even iterate deeper over bboxes that are ALL area? Picture example. Doesn't make sense to go deeper for space/time/logic reasons, but maybe I'm wrong.
- Time complexity of the searching
- Writing out steps for svg parsing
- Do I include hacky method for local level sets?
