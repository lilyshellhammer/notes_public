# Question:
Write pseudocode for an (efficient and correct) algorithm that answers each of the following questions for a pair of BVHs A and B. (You are allowed to assume that the query is available for any kind of leaf nodes.)  Give the time complexity of each algorithm.  For full points, you will need to write legibly in complete sentences.
1. [Do A and B intersect, and where?](#area-intersection)
2. [Do the boundaries of A and B intersect, and where?](#boundary-intersection)
3. [Does A contain B?](#containment)
4. How close are A and B to being tangent, and at what pair of points?
5. What is point on B closest to any point on A?

## Overview
In this document, I summarize methods for checking area intersection, boundary intersection, containment, tangency, and closest distance using (mainly) BVHs. The two methods I write up that do not deal specifically with BVH's are the polygon containment test and the level set containment test. 
Previously, we calculated a discrete representation for the boundary at the beginning of our optimization and recalculated the level set and the BVH at every step. The new methods (other than the level set containment test) don't require a level set (the black box solution to the overlapping/containing/distanceTo nodes *may*, but not BVH traversal) and don't need recalculation of the boundary or the BVH unless the object is rotated. The BVH can be scaled and rotated.
In every example, the main BVH's are calculated as such:
Pseudocode
```
bvhCreator (node, size of bbox)
  create bbox
  increase iter (determines X or Y split)
  if the leaf nodes would be smaller than what is allowed
    set as leaf Node
  else if iter == even (X split)
    set current node as not a leaf node
    call bvhCreator on the right and left, dividing the bbox in half along X axis
  else if iter == odd (Y split)
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

### Area Intersection
If A and B intersect, leaf nodes of their bounding volume hierarchy intersect. We iterate through the trees as follows:
Compare A and B's bounding boxes. If they overlap, compare every permutation of pairs of their childrens bounding boxes. If two children don't overlap, ignore the rest of that those children's children nodes. If the children do overlap continue to iterate until two leaf nodes that overlap are found. If no leaf nodes overlap, the funciton will return false. If any overlapping nodes are found, the function returns true. 
Pseudocode
```
findIntersection (Node a, Node b) //top level function call will return true if any leaf nodes overlap
  if they are both leaves
      check overlap, if it works return true
  if nodes overlap
    if check if both have children
         // return true if any of the children findOverlap calls returns true:
         return findOverlap(one's children, other's children)
    if only one has children
        // return true if any of the children findOverlap calls returns true:
       return findOverlap(one's children, other node)
  if dont overlap, return false
```
Close-to-real code
```C++
bool findIntersection(Node a, Node b){
  if(isLeaf(a) && isLeaf(b)){
    if (overlaps(a.bbox, b.bbox)) return true  //this is where we would use the checkSmallestBBoxOverlap blackbox
    else return false;
  }
  if(overlaps(a.bbox, b.bbox){
    if(isLeaf(a), isLeaf(b))
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
Cases for return value:
  If two levels of the tree don't overlap, return false for those two nodes comparison. Upon returning true, the parent call will evaluate these levels along with any other children nodes the parent had. If any of the children have returned true, the parent will return true.
  If the two levels overlap and they have children, recurse through the children. Return true if any of teh child calls return true.
  If two leafs overlap, true will be returned.

Runtime
  
### Boundary Intersection
If the boundaries of A and B intersect, the leaf nodes of their bounding volume hierachies which contain the boundary within them intersect. The bounding boxes with only "inside" area don't tell us anything about boundary behavior. My proposed solution to this is to "flag" leaf nodes with boundaries. When we are checking for overlap, we use the same findIntersection function above except we only return true on leaf nodes where they contain boundaries.
```C++
findIntersection(Node a, Node b){
  if(isLeaf(a) && isLeaf(b)){
    if (isBoundaryNode(a) && isBoundaryNode(b) && overlaps(a.bbox, b.bbox)) return true  //this is where we would use the                                                                                           checkSmallestBBoxOverlap blackbox
         else return false;
  }
  if(overlaps(a.bbox, b.bbox){
    if(hasTwoChildren(a), hasTwoChildren(b))
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

### Containment
Here I present three possible solutions. Part 1 is about a method of BVH traversal I came up with for checking containment.
The second and third parts are the polygon containment test and the level set containment tests described in Keenan's blog post. 
### Part 1: Negative space BVH traversal
This is my proposed new solution to the containments problem. It requires an additional data structure, but would not require computing the level set, just having a discrete representation of the shape's boundary. 
### Part 2: Polygon Containment
### Part 3: Level Set Containment
