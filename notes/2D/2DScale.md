# WebGL 2D Scale

Using **fAngle.html** and **FCircleUnit.html**, we multiply the position by our desired scale.
```
<script id="vertex-shader-2d" type="x-shader/x-vertex">
attribute vec2 a_position;
 
uniform vec2 u_resolution;
uniform vec2 u_translation;
uniform vec2 u_rotation; 
uniform vec2 u_scale; // new line 
 
void main() {
  // Scale the position
  vec2 scaledPosition = a_position * u_scale; 

  // Rotate the position
  vec2 rotatedPosition = vec2(      
     scaledPosition.x * u_rotation.y + scaledPosition.y * u_rotation.x,
    scaledPosition.y * u_rotation.y - scaledPosition.x * u_rotation.x);
 
  // Add in the translation.
  vec2 position = rotatedPosition + u_translation;  
```
And we add the JavaScript needed to set the scale when we draw.
```
...
 
var scaleLocation = gl.getUniformLocation(program, "u_scale"); 
 
...
 
var scale = [1, 1]; 
 
...
 
// Draw the scene.
function drawScene() {
 
    ...
 
    // Set the translation.
    gl.uniform2fv(translationLocation, translation);
 
    // Set the rotation.                    
    gl.uniform2fv(rotationLocation, rotation);

    // Set the scale.                    
    gl.uniform2fv(scaleLocation, scale);  
 
    // Draw the geometry.
    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    var count = 18;  // 6 triangles in the 'F', 3 points per triangle
    gl.drawArrays(primitiveType, offset, count);
}
```
See **FScale.html**. One thing to notice in this code, is that scaling by a negative value flips our geometry.



