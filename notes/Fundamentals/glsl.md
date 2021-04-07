# WebGL Shaders and GLSL

Reinforcing and detailing what has already been seen, a vertex shader's job is to generate clip space coordinates. It alway takes the form
```
void main() {
    gl_Position = doMathToMakeClipspaceCoordinates
}
```
Your shader is called once per vertex. Each time it's called you are required to set the special global variable, `gl_Position` to some clip space coordinates.

Vertex shaders need data. They can get data in 3 ways:

-Attributes (data pulled from buffers)
-Uniforms (values that stay the same for all vertices of a single draw call)
-Textures (data from pixels/texels)

<h2>Attributes</h2>

The most common way is through buffers and *attribute*. Creating buffers:
`var buf = gl.createBuffer();`

put data in those buffers
```
gl.bindBuffer(gl.ARRAY_BUFFER, buf);
gl.bufferData(gl.ARRAY_BUFFER, someData, gl.STATIC_DRAW);
```
Then, given a shader program you made you look up the location of its attributes at initialization time 

`var positionLoc = gl.getAttribLocation(someShaderProgram, "a_position");`

and at render time tell WebGL how to pull data out of those buffers and into the attribute
```
// turn on getting data out of a buffer for this attribute
gl.enableVertexAttribArray(positionLoc);

var numComponents = 3; // (x, y, z)
var type = gl.FLOAT;   // 32bit floating point values
var normalize = false; // leave the values as they are
var offset = 0;        // start at the beginning of the buffer
var stride = 0;        // how many bytes to move to the next vertex
                       // 0 = use the correct stride for type and numComponents

gl.vertexAttribPointer(positionLoc, numComponents, type, false, stride, offset);
```
In **fundamental.md** we seen that we can do no math in the shader and just pass the data directly through
```
attribute vec4 a_position;

void main() {
    gl_Position = a_position;
}
```
If we put clip space vertices into our buffers it will work.
Attributes can use `float`, `vec2`, `vec3`, `vec4` ,`mat2`, `mat3`, and `mat4`as types.

<h2>Uniforms</h2>

For a shader uniforms are values passed to the shader that stay the same for all vertices in a draw call. As a very simple example we could add an offset to the vertex shader above
```
attribute vec4 a_position;
uniform vec4 u_offset;

void main() {
    gl_Positionc = a_position + u_offset;
}
```
And now we could offset every vertex by a certain amount. First we'd look up the location of the uniform at initialization time 

`var offsetLoc = gl.getUniformLocation(someProgram, "u_offset");` 

And then before drawing we'd set the uniform 

`gl.uniform4fv(offsetLoc, [1, 0, 0, 0]); // offset it to the right half the screen` 

Note that uniforms belong to individual shader programs. If you have multiple shader programs with uniforms of the same name both uniforms will have their own locations and hold their own values. When calling `gl.uniform???` you're only setting the uniform for the *current program*. The current program is the last program you passed to `gl.useProgram`.

Uniforms can be many types. For each type you have to call the corresponding function to set it.
```
gl.uniform1f (floatUniformLoc, v);                  // for float
gl.uniform1fv(floatUniformLoc, [v]);                // for float or float array
gl.uniform2f (vec2UniformLoc, v0, v1);              // for vec2
gl.uniform2fv(vec2UniformLoc, [v0, v1]);            // for vec2 or vec2 array
gl.uniform3f (vec3UniformLoc, v0, v1, v2);          // for vec3
gl.uniform3fv(vec3UniformLoc, [v0, v1, v2]);        // for vec3 or vec3 array
gl.uniform4f (vec4UniformLoc, v0, v1, v2, v4);      // for vec4
gl.uniform4fv(vec4UniformLoc, [v0, v1, v2, v4]);    // for vec3 or vec3 array

gl.uniformMatrix2fv(mat2UniformLoc, false, [  4x element array ]) // for mat2 or mat2 array
gl.uniformMatrix3fv(mat3UniformLoc, false, [  9x element array ]) // for mat3 or mat3 array
gl.uniformMatrix3fv(mat4UniformLoc, false, [ 16x element array ]) // for mat5 or mat4 array

gl.uniform1i (intUniformLoc, v);                  // for int
gl.uniform1iv(intUniformLoc, [v]);                // for int or int array
gl.uniform2i (ivec2UniformLoc, v0, v1);           // for ivec2
gl.uniform2iv(ivec2UniformLoc, [v0, v1]);         // for ivec2 or ivec2 array
gl.uniform3i (ivec3UniformLoc, v0, v1, v2);       // for ivec3
gl.uniform3iv(ivec3UniformLoc, [v0, v1, v2]);     // for ivec3 or ivec3 array
gl.uniform4i (ivec4UniformLoc, v0, v1, v2, v4);   // for ivec4
gl.uniform4iv(ivec4UniformLoc, [v0, v1, v2, v4]); // for ivec3 or ivec3 array

gl.uniform1i (sampler2DUniformLoc,  v);           // for sampler2D (textures)
gl.uniform1iv(sampler2DUniformLoc,  [v]);          // for sampler2D or sampler2D array

gl.uniform1i (samplerCubeUniformLoc,  v);           // for samplerCube (textures)
gl.uniform1iv(samplerCubeUniformLoc,  [v]);         // for samplerCube or samplerCube array
```
There's also type `bool`, `bvec2`, `bvec3`, and `bvec4`. They use either the `gl.uniform?f?` or `gl.uniform?i?` functions.

Note that for an array you can set all the uniforms of the array at once. For example
```
// in shader
uniform vec2 u_someVec2[3];

// in JavaScript at init time
var someVec2Loc = gl.getUniformLocation(someProgram, "u_someVec2");

// at render time
gl.uniform2fv(someVec2Loc, [1, 2, 3, 4, 5, 6]); // set the entire array of u_someVec2
```
But if you want to set individual elements of the array you must look up the location of each element individually.
```
// in JavaScript at init time
var someVec2Element0Loc = gl.getUniformLocation(someProgram, "u_someVec2[0]");
var someVec2Element1Loc = gl.getUniformLocation(someProgram, "u_someVec2[1]");
var someVec2Element2Loc = gl.getUniformLocation(someProgram, "u_someVec2[2]");

// at render time
gl.uniform2fv(someVec2Element0Loc, [1, 2]); // set element 0
gl.uniform2fv(someVec2Element1Loc, [3, 4]); // set element 1
gl.uniform2fv(someVec2Element2Loc, [5, 6]); // set element 2
```
Similarly if you create a struct
```
struct SomeStruct {
    bool active;
    vec2 someVec2;
};
uniform SomeStruct u_someThing;
```
you have to look up each field individually
```
var someThingActiveLoc = gl.getUniformLocation(someProgram, "u_someThing.active");
var someThingSomeVec2Loc = gl.getUniformLocation(someProgram, "u_someThing.someVec2");
```

<h1>Fragment Shader</h1>

A Fragment Shader's job is to provide a color for the current pixel being rasterized. It always takes the form
```
precision mediump float;

void main() {
    gl_FragColor = doMathToMakeColor;
}
```
Your fragment shader is called once per pixel. Each time it's called you are required to set the special global variable, `gl_FragColor` to some color.

Fragment shaders need data. They can get data in 3 ways

1. Uniforms (values that stay the same for every pixel of a single draw call)
2. Textures (data from pixels/texels)
3. Varyings (data passed from the vertex shader and interpolated)

<h3>Uniforms in Shaders</h3>

See **uniforms_in_shaders.md**.

<h3>Textures in Fragment Shaders</h3>

Getting a value from a texture in a shader we create a `sampler2D` uniform and use the GLSL function `texture2D` to extract a value from it.
```
precision mediump float;

uniform sampler2D u_texture;

void main() {
    vec2 texcoord = vec2(0.5, 0.5) // get a value from the middle of the texture
    gl_FragColor = texture2D(u_texture, texcoord);
}
```
What data comes out of the texture is [dependet on many settings](textures.md). At a minimum we need to create and put data in the texture, for example
```
var tex = gl.createTexure();
gl.bindTexture(gl.TEXTURE_2D, tex);
var level = 0;
var width = 2;
var = height = 1;
var data = new Uint8Array([
    255, 0, 0, 255,     // a red pixel
    0, 255, 0, 255,     // a green pixel
]);
gl.texImage2D(gl.TEXTURE_2D, level, gl.RGBA, width, height, 0, gl.RGBA, gl.UNSIGNED_BYTE, data);
```
At initialization time look up the uniform location in the shader program

`var someSamplerLoc = gl.getUniformLocation(someProgram, "u_texture");`

At render time bind the texture to a texture unit
```
var unit = 5;  // Pick some texture uit
gl.activeTexture(gl.TEXTURE0 + unit);
gl.bindTexture(gl.TEXTURE_2D, tex);
```
And tell the shader which unit you bound the texture to

`gl.uniform1i(someSamplerLoc, unit);`

<h3>Varyings</h3>

A varying is a way to pass a value from a vertex shader to a fragment shader which we covered in **how_it_works.md**.

To use varying we need to declare matching varyings in both a vertex and fragment shader. We set the varying in the vertex shader with some value per vertex. When WebGL draws pixels it will interpolate between those values and pass them to the corresponding varying in the fragment shader

Vertex shader
```
atribute vec4 a_position;

uniform vec4 u_offset;

varying vec4 v_positionWithOffset;

void main() {
    gl_Position = a_position + u_offset;
    v_positionWithOffset = a_position + u_offset;
}
```
Fragment shader
```
precision mediump float;

varying vec4 v_positionWithOffset;

void main() {
    // convert from clip space (-1 <-> +1) to color space (0 -> 1).
    vec4 color = v_positionWithOffset * 0.5 + 0.5;
    gl_FragColor = color;
}
```
The example above is a mostly nonsense example. It doesn't generally makes sense to directly copy the clip space values to the fragment shader and use them as color. Nevertheless it will work and produce colors.

<h1>GLSL</h1>

GLSL stands for Graphics Library Shader Language. It's the language shaders are written in. It has some special semi unique features that are certainly not common in JavaScript. It's designed to do the math that is commonly needed to compute things for rasterizing graphics. So for example it has built in types like `vec2`, `vec3`, and `vec4` which represent 2 values, 3 values, and 4 values respectively. Similarly it has `mat2`, `mat3` and `mat4` which represent 2x2, 3x3, and 4x4 matrices. You can do things like multiply a `vec` by a scalar.
```
vec4 a = vec4(1, 2, 3, 4);
vec4 b = a * 2.0;
// b is now vec4(2, 4, 6, 8);
```
Similarly it can do matrix multiplication and vector to matrix multiplication
```
mat4 a = ???
mat4 b = ???
mat4 c = a * b;

vec4 v = ???
vec4 y = c * v;
```
It also has varipus selectors for the parts of a vec. For a vec4

`vec4 v;`

- `v.x` is the same as `v.s` and `v.r` and `v[0]`
- `v.y` is the same as `v.t` and `v.g` and `v[1]`
- `v.z` is the same as `v.p` and `v.b` and `v[2]`
- `v.w` is the same as `v.q` and `v.a` and `v[3]`

It is able to *swizzle* vec components which means you can swap or repeat components.

`v.yyyy`

is the same as

`vec4(v.y, v.y, v.y, v.y)`

Similarly

`v.bgra`

is the same as

`vec4(v.b, v.g, v.r, v.a)`

When constructing a vec or mat you can supply multiple parts at once. So for example

`vec4(v.rgb, 1)`

is the same as

`vec4(v.r, v.g, v.b, 1)`

Also

`vec4(1)`

Is the same as

`vec4(1, 1, 1, 1)`

One thing you'll likely get caught up on is that GLSL is very type strict.

`float f = 1; // ERROR 1 is an int. You can't assign an int to a float`

The correct way is one of these
```
float f = 1.0;      // use float
float f = float(1)  // cast an integer to a float
```
The example above of `vec4(v.rgb, 1)` doesn't complain about the `1` because `vec4` is casting the things inside just like `float(1)`.
GLSL has a bunch of built in functions. Many of them operate on multiple components at once. So for example

`T sin(T angle)`

Means T can be `float`, `vec2`, `vec3` or `vec4`. If you pass in `vec4`you get `vec4` back which the sine of each of the components. In other words if `v`is a `vec4` then

`vec4 s = sin(v);`

is the same as

`vec4 s = vec4(sin(v.x), sin(v.y), sin(v.z), sin(v.w));`

Sometimes one argument is a float and the rest is `T`. That means that float will be applied to all the components. For example if `v1` and `v2` ara `vec4` and `f` is a float then 

`vec4 m = mix(v1, v2, f);`

is the same as

```
vec4 m = vec4(
    mix(v1.x, v2.x, f),
    mix(v1.y, v2.y, f),
    mix(v1.z, v2.z, f),
    mix(v1.w, v2.w, f)
);
```





