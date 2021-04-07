# WebGL Text - Canvas 2D

Instead of using HTML elements for text we can also use another canvas but
with a 2D context.  Without profiling it's just a guess that this would be
faster than using the DOM.  Of course it's also less flexible.  You don't
get all the fancy CSS styling.  But, there's no HTML elements to create
and keep track of.

Similar to the other examples we make a container but this time we put
2 canvases in it.
```html
    <div class="container">
      <canvas id="canvas"></canvas>
      <canvas id="text"></canvas>
    </div>
```
Next setup the CSS so the the canvas and the HTML overlap
```css
    .container {
        position: relative;
    }

    #text {
        position: absolute;
        left: 0px;
        top: 0px;
        z-index: 10;
    }
```
Now look up the text canvas at init time and create a 2D context for it.
```js
    // look up the text canvas.
    var textCanvas = document.querySelector("#text");

    // make a 2D context for it
    var ctx = textCanvas.getContext("2d");
```
When drawing, just like WebGL, we need to clear the 2D canvas each frame.
```js
    function drawScene() {
        ...

        // Clear the 2D canvas
        ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
```
And then we just call `fillText` to draw text
```js
        ctx.fillText(someMsg, pixelX, pixelY);
```
And here's that example

{{{example url="../canvas2d.html" }}}

Why is the text smaller? Because that's the default size for Canvas 2D.
If you want other sizes [check out the Canvas 2D API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Drawing_text).

Another reason to use Canvas 2D is it's easy to draw other things.  For
example let's add an arrow
```js
    // draw an arrow and text.

    // save all the canvas settings
    ctx.save();

    // translate the canvas origin so 0, 0 is at
    // the top front right corner of our F
    ctx.translate(pixelX, pixelY);

    // draw an arrow
    ctx.beginPath();
    ctx.moveTo(10, 5);
    ctx.lineTo(0, 0);
    ctx.lineTo(5, 10);
    ctx.moveTo(0, 0);
    ctx.lineTo(15, 15);
    ctx.stroke();

    // draw the text.
    ctx.fillText(someMessage, 20, 20);

    // restore the canvas to its old settings.
    ctx.restore();
```
Here we're taking advantage of the Canvas 2D [matrix
stack](MatrixStack.md) translate function so we don't have to
do any extra math when drawing our arrow.  We just pretend to draw at the
origin and translate takes care of moving that origin to the corner of our
F.

{{{example url="../canvas2d-arrows.html" }}}

I think that covers using Canvas 2D.  [Check out the Canvas 2D
API](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D)
for more ideas. 

