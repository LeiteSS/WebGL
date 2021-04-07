# WebGL Boilerplate

One thing that makes WebGL seem complicated is that you have these 2 tiny functions, a vertex shader and a fragment shader. Those two functions usually run on your GPU which is where all the speed comes from. That's also why they are written in a custom language, a language that matches what a GPU can do. Those 2 functions need to be compiled and linked. That process is, 99% of the time, the same in every WebGL program.

Here's the boilerplate code for compiling a shader.
```
/**
 * Creates and compiles a shader.
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * @param {string} shaderSource The GLSL source code for the shader.
 * @param {number} shaderType The type of shader, VERTEX_SHADER or FRAGMENT_SHADER.
 * @return {!WebGLShader} The Shader.
 */
function compileShader(gl, shaderSource, shaderType) {
    // Create the shader object
    var shader = gl.createShader(shaderType);

    // Set the shader source code.
    gl.shaderSource(shader, shaderSource);

    // Compile the shader
    gl.compileShader(shader);

    // Check if it compiled
    var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
    if (!success) {
        // Something went wrong during compilation; get the error
        throw "could not compile shader:" + gl.getShaderInfoLog(shader);
    }

    return shader;
};
```
And the boilerplate code for linking 2 shaders into a program
```
/**
 * Creates a program from 2 shaders.
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * @param {!WebGLShader} vertexShader A vertex shader.
 * @param {!WebGLShader} fragmentShader A fragment shader.
 * @return {!WebGLProgram} A program.
 */
function createProgram(gl, vertexShader, fragmentShader) {
    // Create a program
    var program = gl.createProgram();

    // attach the shader.
    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);

    // link the program.
    gl.linkProgram(program);

    // Check if it linked
    var success = gl.getProgramParameter(program, gl.LINK_STATUS);
    if (!success) {
        // Something went wrong during compilation; get the error
        throw "program failed to link:" + gl.getProgramInfoLog(program);
    }

    return program;
};
```
Of course how you decide to handle errors might be different. Throwing exceptions might not be the best way to handle things. Still, those few lines of code are pretty much the same in nearly every WebGL program.

Storing the shaders in non javascript `<script>`tags make them easy to edit.
```
/**
 * Creates a shader from the content of a script tag.
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * @param {string} scriptId The id of the script tag.
 * @param {string} opt_shaderType The type of shader to create.
 *       If not passed in will use the type attribute from the
 *       script tag.
 * @return {!WebGLShader} A Shader.
 */
function createShaderFromScript(gl, scriptId, opt_shaderType) {
    // look up the script tag by id.
    var shaderScript = document.getElementById(scriptId);
    if (!shaderScript) {
        throw("*** Error: unknown script element" + scriptId);
    }

    var shaderSource = shaderScript.text;

    // If we didn't pass in a type, use the 'type' from 
    // the script tag.
    if (!opt_shaderType) {
        if (shaderScript.type == "x-shader/x-vertex") {
            opt_shaderType = gl.VERTX_SHADER;
        } else if (shaderScript.type == "x-shader/x-fragment") {
            opt_shaderType = gl.FRAGMENT_SHADER;
        } else if (!opt_shaderType) {
            throw("*** Error: shader type not set");
        }
    }
    
    return compileShader(gl, shaderSource, opt_shaderType);
};
```
Now to compile a shader I can just do

`var shader = compileShaderFromScript(gl, "someScriptTagId");`

Attach them to a program and like them.
```
/**
 * Creates a program from 2 script tags.
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * @param {string[]} shaderScriptIds The array of ids of the script tags for the shader.
 *      The first is assumed to be the vertex shader, the second the fragment shader.
 * @return {!WebGLProgram} A Program.
 */
 function createProgramFromScripts (gl, shaderScriptIds) {
     var vertexShader = createProgramFromScripts(gl, shaderScriptIds[0], gl.VERTEX_SHADER);
     var fragmentShader = createProgramFromScripts(gl, shaderScriptIds[1], gl.FRAGMENT_SHADER);

     return createProgram(gl, vertexShader, fragmentShader);
 };
```
 The two code below are to resize the canvas
 ```
 <!-- HTML Body -->
 <canvas id="c"></canvas>
 <script src="author's code/webgl-utils.js"></script>
 ```

 then 
 ```
 // JavaScript <script> tag
 var program = webglUtils.createProgramFromScripts(
     gl, [idOfVertexShaderScript, idOfFragmentShaderScript]);

...
canvas = document.getElementById("c");
webglUtils.resizeCanvasToMatchDisplaySize(canvas);
```

<h1>Codes that we will use</h1>

- `webgl-lessons-ui.js`: This provides code to setup sliders that have a visible value that updates when you drag the slider.
- `lessons-helper.js`: It helps print error messages to the screen when used inside the live editor among other things.
- `m3.js`: This is a bunch of 2d math functions. They get created started with the first article about matrix math and as they are created they are inline but eventually they're just too much clutter so after few example they are used by including this script.
- `m4.js`: This is a bunch of 3d math functions. They get created started with the first article about 3d and as they are created they are inline but eventually they're just too much clutter so after the 2nd article on 3d folder they are used by including this script
