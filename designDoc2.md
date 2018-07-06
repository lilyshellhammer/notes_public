# Design Doc 2
## Introduction

### Background
Currently our system creates a Level Set using the fast sweeping method, then creates a BVH from the Level Set, and started imlementing these data structures to fulfill objectives (unary or binary relationships the user wants to enforce on shapes in the scene). While implementing this system we were
- Generating a Level Set/Signed Distance Field (abr. SDF) for a grid 2x as big as our starting shape (circle)
- Sweeping through the SDF once (not even the full four times) to generate correct values
- At every position in the SDF, storing the coordinates of the closest point
- Creating a BVH from the SDF and recalculating BOTH at every step

Many of these things we were doing were not only inefficient and should be replace dby another algorithm, they were entirely unnecessary. The changes I am proposing to our syste with this second design document are:
### Changes
- Not calculate the SDF. 
    - The most important thing we got from our version of the SDF was boundary location. If we can create a discrete representation of the boundary and inside/outside cells this will give us enough information to (later) easily calculate:
        - Confirmed overlap or lack there of at [Leaf node bounding boxes](#Overlaps-Function)
        - (In some simplified form yet to be determined) whether one object is outside of anotherß (similar to  `contains` level set version.. final form of `contains` tbd)
- Not recalculate the size of above grid or of the generated BVH. 
   - This was a big oversight on our part. We were stucking thinking about the system on a grid layout, when the grid is just a discrete representation of a continuous function. It makes it possible to store this continuous function in an intelligible, 

## PseudoCode

### Overlaps Function
