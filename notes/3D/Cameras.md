# WebGL 3D Cameras

We need to effectively move the world in front of the camera. The easiest way to do this is to use an "inverse" matrix. The math to compute an inverse matrix in the general case is complex but conceptually it's easy. The inverse is the value you'd use to negate some other value. For example, the inverse of a matrix that translates in X by 123 is a matrix that translates in X by -123. The inverse of a matrix that scales by 5 is a matrix that scales by 1/5th or 0.2. The inverse of a matrix that rotates 30° around the X axis would be one that rotates -30° around the X axis.

Up until this point we've used translation, rotation and scale to affect the position and orientation of our 'F'. After multiplying all the matrices together we have a single matrix that represents how to move the 'F' from the origin to the place, size and orientation we want it. We can do the same for a camera. Once we have the matrix that tells us how to move and rotate the camera from the origin to where we want it we can compute its inverse which will give us a matrix that tells us how to move and rotate everything else the opposite amount which will effectively make it so the camera is at (0, 0, 0) and we've moved everything in front of it.

First, because we are drawing 5 things and they all use the same projection matrix we'll compute that outside the loop
```
// Compute the projection matrix
var aspect = gl.canvas.clientWidth / gl.canvas.clientHeight;
var zNear = 1;
var zFar = 2000;
var projectionMatrix = m4.perspective(fieldOfViewRadians, aspect, zNear, zFar);
```
Next we'll compute a camera matrix. This matrix represents the position and orientation of the camera in the world. The code below makes a matrix that rotates the camera around the origin `radius * 1.5` distance out and looking at the origin. Then, the camera is moving.
```
var numFs = 5;
var radius = 200;
 
// Compute a matrix for the camera
var cameraMatrix = m4.yRotation(cameraAngleRadians);
cameraMatrix = m4.translate(cameraMatrix, 0, 0, radius * 1.5);
```
We then compute a "view matrix" from the camera matrix. A "view matrix" is the matrix that moves everything the opposite of the camera effectively making everything relative to the camera as though the camera was at the origin (0,0,0). We can do this by using an `inverse` function that computes the inverse matrix (the matrix that does the exact opposite of the supplied matrix). In this case the supplied matrix would move the camera to some position and orientation relative to the origin. The inverse of that is a matrix that will move everything else such that the camera is at the origin.
```
// Make a view matrix from the camera matrix.
var viewMatrix = m4.inverse(cameraMatrix);
```
Now we combine the view and projection matrix into a view projection matrix.
```
// Compute a view projection matrix
var viewProjectionMatrix = m4.multiply(projectionMatrix, viewMatrix);
```
Finally we draw a circle of Fs. For each F we start with the view projection matrix, then rotate and move out radius units.
```
for (var ii = 0; ii < numFs; ++ii) {
  var angle = ii * Math.PI * 2 / numFs;
  var x = Math.cos(angle) * radius;
  var y = Math.sin(angle) * radius
 
  // starting with the view projection matrix
  // compute a matrix for the F
  var matrix = m4.translate(viewProjectionMatrix, x, 0, y);
 
  // Set the matrix.
  gl.uniformMatrix4fv(matrixLocation, false, matrix);
 
  // Draw the geometry.
  var primitiveType = gl.TRIANGLES;
  var offset = 0;
  var count = 16 * 6;
  gl.drawArrays(primitiveType, offset, count);
}
```
To put our camera where we want and to point what object we want. First we call `cameraPosition`, then we need to know the position of the thing we want to look at or aim at. We'll call it the `target`. If we subtract the `target` from the `cameraPosition` we'll have a vector that points in the direction we'd need to go from the camera to get to the target. Let's call it `zAxis`. Since we know the camera points in the `-z` direction we can subtract the other way `cameraPosition - target`. We normalize the results and copy it directly into the `z` part of a matrix.
```
+----+----+----+----+
|    |    |    |    |
+----+----+----+----+
|    |    |    |    |
+----+----+----+----+
| Zx | Zy | Zz |    |
+----+----+----+----+
|    |    |    |    |
+----+----+----+----+
```
This part of a matrix represents the Z axis. In this case the Z-axis of the camera. Normalizing a vector means making it a vector that represents 1.0. In 3D we need unit spheres and a normalized vector represents a point on a unit sphere.

Just a single vector gives us a point on a unit sphere but which orientation from that point to orient things? We need to fill out the other parts of the matrix. Specifically the X axis and Y axis parts. We know that in general these 3 parts are perpendicular to each other. We also know that "in general" we don't point the camera straight up. Given that, if we know which way is up, in this case (0,1,0), We can use that and something called a "cross product" to compute the X axis and Y axis for the matrix.

If you have 2 unit vectors and you compute the cross product of them you'll get a vector that is perpendicular to those 2 vectors. In other words, if you have a vector pointing south east, and a vector pointing up, and you compute the cross product you'll get a vector pointing either south west or north east since those are the 2 vectors that are perpendicular to south east and up. Depending on which order you compute the cross product in, you'll get the opposite answer.

In any case if we compute the cross product of our `zAxis` and up we'll get the `xAxis` for the camera.
```
up cross zAxis = xAxis
```
And now that we have the `xAxis` we can cross the `zAxis` and the `xAxis` which will give us the camera's `yAxis`
```
zAxis cross xAxis = yAxis
```
Now all we have to do is plug the 3 axes into a matrix. That gives us a matrix that will orient something that points at the target from the `cameraPosition`. We just need to add in the position
```
+----+----+----+----+
| Xx | Xy | Xz |  0 |  <- x axis
+----+----+----+----+
| Yx | Yy | Yz |  0 |  <- y axis
+----+----+----+----+
| Zx | Zy | Zz |  0 |  <- z axis
+----+----+----+----+
| Tx | Ty | Tz |  1 |  <- camera position
+----+----+----+----+
```
Here's the code to compute the cross product of 2 vectors.
```
function cross(a, b) {
  return [a[1] * b[2] - a[2] * b[1],
          a[2] * b[0] - a[0] * b[2],
          a[0] * b[1] - a[1] * b[0]];
}
``` 
Here's the code to subtract two vectors.
```
function subtractVectors(a, b) {
  return [a[0] - b[0], a[1] - b[1], a[2] - b[2]];
}
```
Here's the code to normalize a vector (make it into a unit vector).
```
function normalize(v) {
  var length = Math.sqrt(v[0] * v[0] + v[1] * v[1] + v[2] * v[2]);
  // make sure we don't divide by 0.
  if (length > 0.00001) {
    return [v[0] / length, v[1] / length, v[2] / length];
  } else {
    return [0, 0, 0];
  }
}
```
Here's the code to compute a `lookAt` matrix.
```
var m4 = {
  lookAt: function(cameraPosition, target, up) {
    var zAxis = normalize(
        subtractVectors(cameraPosition, target));
    var xAxis = normalize(cross(up, zAxis));
    var yAxis = normalize(cross(zAxis, xAxis));
 
    return [
       xAxis[0], xAxis[1], xAxis[2], 0,
       yAxis[0], yAxis[1], yAxis[2], 0,
       zAxis[0], zAxis[1], zAxis[2], 0,
       cameraPosition[0],
       cameraPosition[1],
       cameraPosition[2],
       1,
    ];
  }
```
And here is how we might use it to make the camera point at a specific 'F' as we move it.
```
...
 
  // Compute the position of the first F
  var fPosition = [radius, 0, 0];
 
  // Use matrix math to compute a position on a circle where
  // the camera is
  var cameraMatrix = m4.yRotation(cameraAngleRadians);
  cameraMatrix = m4.translate(cameraMatrix, 0, 0, radius * 1.5);
 
  // Get the camera's position from the matrix we computed
  var cameraPosition = [
    cameraMatrix[12],
    cameraMatrix[13],
    cameraMatrix[14],
  ];
 
  var up = [0, 1, 0];
 
  // Compute the camera's matrix using look at.
  var cameraMatrix = m4.lookAt(cameraPosition, fPosition, up);
 
  // Make a view matrix from the camera matrix.
  var viewMatrix = m4.inverse(cameraMatrix);
 
  ...
```

## `LookAt` standards

It makes a matrix that moves everything else in front of the camera rather than a matrix that moves the camera itself. 


