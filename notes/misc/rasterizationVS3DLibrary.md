# WebGL - Rasterization vs 3D libraries

This markdown explain why the author called WebGL a Rasterization API. Noting that everything is a matter of perspective. 

WebGL does not draw 3D graphics out of the box. You can write a library that will draw 3D graphics with WebGL but by itself it
does not do 3D graphics.

In 3D libraries they take care of the 3D. You give them a camera position and field of view, a couple of lights, and a cube. They deal with all the rest (You supply 3D data and the library take care of calculating clip space points from 3D). On the other hand you need to know matrix math, normalized coordinates, frustums, cross products, dot products, varying interpolation, lighting specular calculations and all kinds of the other stuff that often take months or years to fully understand.

OpenGL is a library that handle the 3D for you. Otherwise, OpenGL ES 2.0 does not. And **three.min.js** handles the 3D too, see **cube.html** and **webGLCube.html** to understand. 
And realize that if you inspect the code **canvasCube.html** and **webGLCube.html**, you'll see there's not a whole lot of difference in therms of the amount of knowledge or for that matter even the code. Ultimately the Canvas version loops over the vertices, does the math we supplied and draws some line in 2D. The WebGL version does the same thing except the we supplied is in GLSL and executed by the GPU.

