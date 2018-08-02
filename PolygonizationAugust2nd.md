# Polygonization Distances

This document explains how different max and min, sign and unsigned distances are defined between polygons.

A,B - "solid" polygons (open)
boundaryA, boundaryB - boundaries
| . | : R^2 -> R; (x1, x2) |-> sqrt(x_1^2, x_2^2)


POINT-SHAPE DISTANCE 

d_a(x) = min (y in boundary A) |x-y|
d_a, s_a : R^2 -> R
s_a(x) = d_a(x) { -1, x in A
                  0, x in bondary A
                  1, x not in A or boudnary A}
                  

SHAPE-SHAPE DISTANCES

[little d]
This is the shortest distance between polygons. It is the distance between the point on A that is closest to B and the point on B that is closest to A. This distance is unsigned. It is symmmetrical, meaning d(A,B) is the same as d(B,A). 

PROOF for symmetry

[little s]
This is the minimum signed distance between polygons. It is the most negative distance from a point on A to it's closest point on boundary B. Meaning for all points on A, this is the point whose closest boundary distance is the minimum signed distance. the  It is *not* symmmetrical, meaning d(A,B) is different than d(B,A). This is because the sign of a distance is determined by one shape's boundary. The sign of your distance is deteremined by whether a point on B is contained in A. 

[big d]
This is the maximum unsigned distance between polygons. It is the distance between the point on A that is farthest from the boundary of B. This distance is unsigned.  TODO: Is this symmetrical or asymmetrical?

[big s]
This is the maximum signed distance between polygons. It is the point on A whose distance to the closest boundary point on B is the most positive. When A is contained in B, the maximum signed distnace is the point who is closest to the boundary of B and is a negative number.   TODO: Is this symmetrical or asymmetrical?

[max max]???

Claims
When B is contained in A
  small s = Big d
  big s = small d
  
 When B is outside of A
  small d = small s
  big d = big s
  
 When B intersects with A at least once:
  small d = big D
 
 ... [CONTINUE]
