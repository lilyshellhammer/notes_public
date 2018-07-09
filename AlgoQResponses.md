# Question:
Write pseudocode for an (efficient and correct) algorithm that answers each of the following questions for a pair of BVHs A and B. (You are allowed to assume that the query is available for any kind of leaf nodes.)  Give the time complexity of each algorithm.  For full points, you will need to write legibly in complete sentences.
1. [Do A and B intersect, and where?](#area-intersection)
2. Do the boundaries of A and B intersect, and where?
3. Does A contain B?
4. How close are A and B to being tangent, and at what pair of points?
5. What is point on B closest to any point on A?

### Area Intersection
If A and B intersect, leaf nodes of their bounding volume hierarchy intersect. We iterate through the trees as follows:
Compare A and B's bounding boxes. If they overlap, compare their childrens bounding boxes. If two children don't overlap, ignore the rest of that tree. If the children do overlap continue to iterate until two leaf nodes that overlap are found. If no leaf nodes overlap, the funciton will return false. If any overlappign nodes are found, the tree return true. 
```
findIntersection(Node a, Node b){
  if(isLeaf(a) && isLeaf(b)){
    if (overlaps(a.bbox, b.bbox)) return true  //this is where we would use the checkSmallestBBoxOverlap blackbox
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
Cases for return value:
  if two levels of the tree don't overlap, return false for those two nodes comparison. Upon returning true, the parent call will evaluate these levels along with any other children nodes the parent had.

### Boundary Intersection
If the boundaries of A and B intersect, the leaf nodes of their bounding volume hierachies which contain the boundary within them intersect. The bounding boxes with only "inside" area don't tell us anything about boundary behavior. My proposed solution to this is to "flag" leaf nodes with boundaries. When we are checking for overlap, we use the same findIntersection function above except we only return true on leaf nodes where they contain boundaries.
```
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

### 
