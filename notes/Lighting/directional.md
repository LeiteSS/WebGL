# WebGL 3D - Directional Lighting

Directional lighting assumes the light is coming uniformly from one direction. The sun on a clear day is often considered a directional light. It's so far way that its rays can be considered to be hitting the surface of an object all in parallel.

If we know what direction the light is traveling and we know what direction the surface of the object is facing we can take the *dot product* of the 2 directions and it will give us the cosine of the angle between the 2 directions.

Knowing what direction the surface of our 3D object is facing and we know the direction the light is shinning the we can just take the dot product of them and it will give us a number 1 if the light is pointing directly at the surface and -1 if they are pointing directly opposite.
Sums up, we can multiply our color by that dot product value and make itself light.

## Introducing Normals

Normals are called that from the Latin word *norma*, a carpenter's square. Just as a carpenter's square has a right angle, normals are at a right angle to a line or surface. In 3D graphics a normal is the word for a unit vector that describes the direction a surface is facing.

A cube has 3 normals at each corner, because we need 3 different normals to represent the way each face of the cube is facing.

Since the 'F' is very boxy and its faces are aligned to the x, y, or z axis it will be pretty easy. Things that are facing forward have the normal `0, 0, 1`. Things that are facing away are `0, 0, -1`. Facing left is `-1, 0, 0`. Facing right is `1, 0, 0`. Up is `0, 1, 0` and down is `0, -1, 0`.
```
function setNormals(gl) {
  var normals = new Float32Array([
          // left column front
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
 
          // top rung front
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
 
          // middle rung front
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
          0, 0, 1,
 
          // left column back
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
 
          // top rung back
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
 
          // middle rung back
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
          0, 0, -1,
 
          // top
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
 
          // top rung right
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
 
          // under top rung
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
 
          // between top rung and middle
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
 
          // top of middle rung
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
          0, 1, 0,
 
          // right of middle rung
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
 
          // bottom of middle rung.
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
 
          // right of bottom
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
          1, 0, 0,
 
          // bottom
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
          0, -1, 0,
 
          // left side
          -1, 0, 0,
          -1, 0, 0,
          -1, 0, 0,
          -1, 0, 0,
          -1, 0, 0,
          -1, 0, 0]);
  gl.bufferData(gl.ARRAY_BUFFER, normals, gl.STATIC_DRAW);
}
```
and set them up. While we're at it let's remove the vertex colors so it's easier to see the lighting.
```
// look up where the vertex data needs to go.
var positionLocation = gl.getAttribLocation(program, "a_position");
var normalLocation = gl.getAttribLocation(program, "a_normal");

...

// Create a buffer to put normals in
var normalBuffer = gl.createBuffer();
// Bind it to ARRAY_BUFFER (think of it as ARRAY_BUFFER = normalBuffer)
gl.bindBuffer(gl.ARRAY_BUFFER, normalBuffer);
// Put normals data into buffer
setNormals(gl);
```
And at render time
```
// Turn on the normal attribute
gl.enableVertexAttribArray(normalLocation);
 
// Bind the normal buffer.
gl.bindBuffer(gl.ARRAY_BUFFER, normalBuffer);
 
// Tell the attribute how to get data out of normalBuffer (ARRAY_BUFFER)
var size = 3;          // 3 components per iteration
var type = gl.FLOAT;   // the data is 32bit floating point values
var normalize = false; // normalize the data (convert from 0-255 to 0-1)
var stride = 0;        // 0 = move forward size * sizeof(type) each iteration to get the next position
var offset = 0;        // start at the beginning of the buffer
gl.vertexAttribPointer(
    normalLocation, size, type, normalize, stride, offset)
```
Now we need to make our shaders use them 

First the vertex shader we just pass the normals through to the fragment shader
```
attribute vec4 a_position;
attribute vec3 a_normal;

uniform mat4 u_matrix;

varying vec3 v_normal;

void main() {
  // Multiply the position by the matrix.
  gl_Position = u_matrix * a_position;
 
  // Pass the normal to the fragment shader
  v_normal = a_normal;
}
```
And the fragment shader we'll do the math using the dot product of the direction of the light and the normal 
```
precision mediump float;
 
// Passed in from the vertex shader.

varying vec3 v_normal;
 
uniform vec3 u_reverseLightDirection;
uniform vec4 u_color;
 
void main() {
   // because v_normal is a varying it's interpolated
   // so it will not be a unit vector. Normalizing it
   // will make it a unit vector again
   vec3 normal = normalize(v_normal);
 
   float light = dot(normal, u_reverseLightDirection);
 
   gl_FragColor = u_color;
 
   // Lets multiply just the color portion (not the alpha)
   // by the light
   gl_FragColor.rgb *= light;
}
```
Then we need to lookup the locations of `u_color` and `u_reverseLightDirection`.
```
  // lookup uniforms
  var matrixLocation = gl.getUniformLocation(program, "u_matrix");
  var colorLocation = gl.getUniformLocation(program, "u_color");
  var reverseLightDirectionLocation =
      gl.getUniformLocation(program, "u_reverseLightDirection");
```
and we need to set them 
```
// Set the matrix.
  gl.uniformMatrix4fv(matrixLocation, false, worldViewProjectionMatrix);
 
  // Set the color to use
  gl.uniform4fv(colorLocation, [0.2, 1, 0.2, 1]); // green
 
  // set the light direction.
  gl.uniform3fv(reverseLightDirectionLocation, m4.normalize([0.5, 0.7, 1]));
```
`normalize`, which we went over before, will make whatever values we put in there into a unit vector. The specific values in the sample are `x = 0.5` which is positive x means the light is on the right pointing left. `y = 0.7` which is positive y means the light is above pointing down. `z = 1` which is positive z means the light is in front pointing into the scene. the relative values means the the direction is mostly pointing into the scene and pointing more down then right.

To fix when rotating the F the lighting isn't changing, we need to re-orient the normals as the object is re-oriented. Like we did for positions we can multiply the normals by some matrix. The most obvious matrix would be the `world` matrix. As it is right now we're only passing in 1 matrix called `u_matrix`. Let's change it to pass in 2 matrices. One called `u_world` which will be the world matrix. Another called `u_worldViewProjection` which will be what we're currently passing in as `u_matrix`
```
attribute vec4 a_position;
attribute vec3 a_normal;
 
uniform mat4 u_worldViewProjection;
uniform mat4 u_world;
 
varying vec3 v_normal;
 
void main() {
  // Multiply the position by the matrix.
  gl_Position = u_worldViewProjection * a_position;
 
  // orient the normals and pass to the fragment shader
  v_normal = mat3(u_world) * a_normal;
}
```
Notice we are multiplying `a_normal` by `mat3(u_world)`. That's because normals are a direction so we don't care about translation. The orientation portion of the matrix is only in the top 3x3 area of the matrix.

Now we have to look those uniforms up
```
 // lookup uniforms
  var worldViewProjectionLocation =
      gl.getUniformLocation(program, "u_worldViewProjection");
  var worldLocation = gl.getUniformLocation(program, "u_world");
```
And we have to change the code that updates them
```
// Set the matrices
gl.uniformMatrix4fv(
    worldViewProjectionLocation, false,
    worldViewProjectionMatrix);
gl.uniformMatrix4fv(worldLocation, false, worldMatrix);
```
To solve a third problem[*], first we'll update the shader. Technically we could just update the value of `u_world` but it's best if we rename things so they're named what they actually are otherwise it will get confusing.

[*] We're multiplying the `normal` by the `u_world` matrix to re-orient the normals. What happens if we scale the world matrix? It turns out we get the wrong normals.
```
attribute vec4 a_position;
attribute vec3 a_normal;
 
uniform mat4 u_worldViewProjection;
uniform mat4 u_worldInverseTranspose;
 
varying vec3 v_normal;
 
void main() {
  // Multiply the position by the matrix.
  gl_Position = u_worldViewProjection * a_position;
 
  // orient the normals and pass to the fragment shader
  v_normal = mat3(u_worldInverseTranspose) * a_normal;
}
```
The we need to look that up
```
 var worldInverseTransposeLocation =
      gl.getUniformLocation(program, "u_worldInverseTranspose");
```
And we need to compute and set it
```
var worldViewProjectionMatrix = m4.multiply(viewProjectionMatrix, worldMatrix);
var worldInverseMatrix = m4.inverse(worldMatrix);
var worldInverseTransposeMatrix = m4.transpose(worldInverseMatrix);
 
// Set the matrices
gl.uniformMatrix4fv(
    worldViewProjectionLocation, false,
    worldViewProjectionMatrix);

gl.uniformMatrix4fv(
    worldInverseTransposeLocation, false,
    worldInverseTransposeMatrix);
```
and here's the code to transpose a matrix
```
var m4 = {
  transpose: function(m) {
    return [
      m[0], m[4], m[8], m[12],
      m[1], m[5], m[9], m[13],
      m[2], m[6], m[10], m[14],
      m[3], m[7], m[11], m[15],
    ];
  },
 
  ...
```
The effect is subtle and we aren't scaling anything, so there's no noticeable difference, but at least now we're prepared.

## Alternatives to `mat3(u_worldInverseTranspose) * a_normal`

In our shader in ***Directional.html** there's a line like this
```
v_normal = mat3(u_worldInverseTranspose) * a_normal;
```
We could have done this
```
 v_normal = (u_worldInverseTranspose * vec4(a_normal, 0)).xyz;
```
Because we set `w` to 0 before multiplying that would end up multiplying the translation from the matrix by 0 effectively removing it. 

Yet another solution would be to make `u_worldInverseTranspose` a `mat3`. There are 2 reasons not to do that. One is we might have other needs for the full `u_worldInverseTranspose` so passing the entire `mat4` means we can use with for those other needs. Another is that all of our matrix functions in JavaScript make 4x4 matrices. Making a whole other set for 3x3 matrices or even converting from 4x4 to 3x3 is work we'd rather not do unless there was a more compelling reason.
