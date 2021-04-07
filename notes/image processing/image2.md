# WebGL image processing continued

If you want to learn how to build shader with mental mill, there is a [channel on youtube that provides a tutorial](https://www.youtube.com/channel/UCVTOd9r0bef-CC5xPkvS9jw). This technique is used in games engines:) 

However, in this class we will see how to provides to the users options to manipulate the image. 

A more flexible way is to use 2 more textures and render to each texture in turn, ping ponking back and forth and applying the next effect each time. 
```
Original Image  -> [Blur]           -> Texture 1
Texture 1       -> [Sharpen]        -> Texture 2
Texture 2       -> [Edge Detect]    -> Texture 1
Texture 1       -> [Blur]           -> Texture 2
Texture 2       -> [Normal]         -> Canvas
```
To this we need to create a collection of states (a list of attachments) called `FrameBuffer`. By attaching a texture to a `FrameBuffer` we can frame into that texture.

First, let's do some lines used in the **showingAnImage.html**, **un-blurredImage.html**, **swappingImage.html**, and **processingImage.html**. To create a function.
```
function createAndSetupTexture(gl) {
    var texture = gl.createTexture();
    gl.bindTexture(gl.TEXTURE_2D, texture);
 
    // Set up texture so we can render any size image and so we are
    // working with pixels.
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
 
    return texture;
}
 
// Create a texture and put the image in it.
var originalImageTexture = createAndSetupTexture(gl);
gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);
```
And now let's use that function to make 2 more textures and attach them to 2 `FrameBuffers`
```
// create 2 textures and attach them to framebuffers.
var textures = [];
var framebuffers = [];
for (var ii = 0; ii < 2; ++ii) {
    var texture = createAndSetupTexture(gl);
    textures.push(texture);
 
    // make the texture the same size as the image
    gl.texImage2D(
        gl.TEXTURE_2D, 0, gl.RGBA, image.width, image.height, 0,
        gl.RGBA, gl.UNSIGNED_BYTE, null);
 
    // Create a framebuffer
    var fbo = gl.createFramebuffer();
    framebuffers.push(fbo);
    gl.bindFramebuffer(gl.FRAMEBUFFER, fbo);
 
    // Attach a texture to it.
    gl.framebufferTexture2D(
        gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, texture, 0);
}
```
Now let's make a set of kernels and then a list of them to apply.
```
// Define several convolution kernels
var kernels = {
    normal: [
      0, 0, 0,
      0, 1, 0,
      0, 0, 0
    ],
    gaussianBlur: [
      0.045, 0.122, 0.045,
      0.122, 0.332, 0.122,
      0.045, 0.122, 0.045
    ],
    unsharpen: [
      -1, -1, -1,
      -1,  9, -1,
      -1, -1, -1
    ],
    emboss: [
       -2, -1,  0,
       -1,  1,  1,
        0,  1,  2
    ]
};
 
// List of effects to apply.
var effectsToApply = [
    "gaussianBlur",
    "emboss",
    "gaussianBlur",
    "unsharpen"
];
```
And finally let's apply each one, ping ponging which texture we are rendering too
```
// start with the original image
gl.bindTexture(gl.TEXTURE_2D, originalImageTexture);
 
// don't y flip images while drawing to the textures
 gl.uniform1f(flipYLocation, 1);
 
// loop through each effect we want to apply.
for (var ii = 0; ii < effectsToApply.length; ++ii) {
    // Setup to draw into one of the framebuffers.
    setFramebuffer(framebuffers[ii % 2], image.width, image.height);
 
    drawWithKernel(effectsToApply[ii]);
 
    // for the next draw, use the texture we just rendered to.
    gl.bindTexture(gl.TEXTURE_2D, textures[ii % 2]);
}
 
// finally draw the result to the canvas.
gl.uniform1f(flipYLocation, -1);  // need to y flip for canvas
setFramebuffer(null, canvas.width, canvas.height);
drawWithKernel("normal");
 
function setFramebuffer(fbo, width, height) {
    // make this the framebuffer we are rendering to.
    gl.bindFramebuffer(gl.FRAMEBUFFER, fbo);
 
    // Tell the shader the resolution of the framebuffer.
    gl.uniform2f(resolutionLocation, width, height);
 
    // Tell webgl the viewport setting needed for framebuffer.
    gl.viewport(0, 0, width, height);
}
 
function drawWithKernel(name) {
    // set the kernel
    gl.uniform1fv(kernelLocation, kernels[name]);
 
    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 6);
}
```
Calling `gl.bindFramebuffer` with null tells WebGL you want to render to the canvas instead of to one of your framebuffers.

WebGL has to convert from clip space back into pixels. It does this based on the settings of `gl.viewport`. Since the `FrameBuffers` we are rendering into are a different size than the canvas we need to set the `viewport` appropriately when rendering to the `FrameBuffer` textures and then again when finally rendering to the canvas.

Finally in the original example we flipped the `Y` coordinate when rendering because WebGL displays the canvas with `0,0` being the bottom left corner instead of the more traditional for 2D top left. That's not needed when rendering to a `FrameBuffer`. Because the `FrameBuffer` is never displayed, which part is top and bottom is irrelevant. All that matters is that pixel `0,0` in the `FrameBuffer` corresponds to `0,0` in our calculations. To deal with this I made it possible to set whether to flip or not by adding one more input into the shader.
```
<script id="2d-vertex-shader" type="x-shader/x-vertex">
...
uniform float u_flipY;
...
 
void main() {
   ...
 
   gl_Position = vec4(clipSpace * vec2(1, u_flipY), 0, 1);
 
   ...
}
</script>
```
And then we can set it when we render with
```
...
 
var flipYLocation = gl.getUniformLocation(program, "u_flipY");
 
...
 
// don't flip
gl.uniform1f(flipYLocation, 1);

...
 
// flip
gl.uniform1f(flipYLocation, -1);
```
If you wanted to do full on image processing you'd probably need many GLSL programs. A program for hue, saturation and luminance adjustment. Another for brightness and contrast. One for inverting, another for adjusting levels, etc. You'd need to change the code to switch GLSL programs and update the parameters for that particular program.