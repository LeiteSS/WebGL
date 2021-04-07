# Saving and Loading Files in a Web Page

https://games.greggman.com/game/saving-and-loading-files-in-a-web-page/

This article is targeted at people who've started learning web programming. They'd made a few web pages with JavaScript. Maybe they've made a paint program using 2d canvas or a 3d scene using three.js. Maybe it's an audio sound maker, maybe it's a tile map editor. At some point they wonder "how do I save files"?

Maybe they have a save button that just puts all the data into a textarea and presents it to the user and says "copy this and paste it into notepad to save".

Well, the way you save in a web page is you save to a web server. OH THE HORROR! I hear you screaming WHAT? A server? Why would I want to install some giant server just to save data?

I'm here to show you a web server is not a giant piece of software. In fact it's tiny. The smallest web server in many languages can be written in a few lines of code.

For example node.js is a version of JavaScript that runs outside the browser. If you've ever used Perl or Python it works exactly the same. You install it. You give it files to run. It runs them. Perl you give perl files, python you give python files, node you give JavaScript files.

So using `node.js` here is the smallest web server
```
const http = require('http');
function handleRequest(request, response){
    response.end('Hello World: Path = ' + request.url);
}
http.createServer(handleRequest).listen(8000, function() { });
```
JUST 5 LINES OF CODE!!!

Now, all these 5 lines do is return "Hello World:: Path = <path>" for every page but really that's the basics of a web server. Looking at the code above without even explaining it you could imagine looking at request.url and deciding to do different things depending on what the url is. One URL might save, one might load, one might login, etc. That's really it.

Let's explain these 5 lines of code
```
const http = require('http');
```
`require` is the equivalent of `import` in **python** or `#include` in **C++** or `using` in **C#** or in **JavaScript** using `<script> src="..."></script>`. It's loading the module `'http'` and referencing it by the variable `http`.
```
function handleRequest(request, response){
    response.end('Hello World: Path = ' + request.url);
}
```
This is a function that will get called when we get a request from a browser. The request holds data about what the browser requested like the `URL` for the request and all the headers sent by the browser including cookies, what language the user's browser is set to, etc...

`response` is an object we can use to send our response back to the browser. As you can see here we're sending a string. We could also load a file and send the contents of that file. Or we would query a database and send back the results. But everything starts here.
```
const server = http.createServer(handleRequest);
server.listen(8000, function() { 
  console.log("Listening at http://localhost:8000");
});
```
The last line I expanded a little. First it calls http.createServer and passes it the function we want to be called for all requests.

Then it calls server.listen which starts it listening for requests from the browser. The 8000 is which port to listen on and the function is a callback to tell us when the server is up and running.

TRY IT OUT!

To run this server install node.js. Don't worry it's not some giant ass program. It's actually rather small. Much smaller than python or perl or any of those other languages.

Now open a terminal on OSX or on windows open a "Node Command Prompt" (node made this when you installed it).

Make a folder somewhere and `cd` to it in your terminal / command prompt

Make a file called `index.js` and copy and paste the 5 lines above into it. Save it.

Now type `node index.js`

In your browser open a new tab/window and go to `http://localhost:8000` It should say Hello World: Path = \. If you type some other URL like http://localhost:8000/foo/bar?this=that you'll see it returns that back to you.

Congratulations, you just wrote a web server!

## Let's add serving files

You can imagine the code to serve files. You'd parse the `URL` to get a path, read the corresponding file, call `response.end(contentsOfFile)`. It's literally that easy. But, just to make it less code (and cover more cases) there's a library that does it for us and it's super easy to use.

Press `Ctrl-C` to stop your server if you haven't already. Then type
```
npm install express
```
It will download a bunch of files and put them in a subfolder called `node_modules`. It will also probably print an error about no `package.json` which you can ignore (google package.json later)

Now let's edit our file again. We're going to replace the entire thing with this
```
"use strict";
const express = require('express');
const baseDir = 'public';
 
let app = express();
app.use(express.static(baseDir));
app.listen(8000, function() {
    console.log("listening at http://localhost:8000");
});
```
Looking at the last 2 lines you see `app.listen(8000...` just like before. That's because express is the same http server we had before. It's just added some structure we'll get to in a bit.

The cool part here is the line
```
app.use(express.static(baseDir));
```
It says "serve all the files from `baseDir`.

So, make a subfolder called `public`. Inside make a file called test.html and inside that file put O'HI You. Save it. Now run your server again with node index.js

Go to `http://localhost:8000/test.html` in your browser. You should see "O'HI You" in your browser.

Congratulations. You have just made a web server that will serve any files you want all in 9 lines of code!

## Let's Save Files

To save files we need to talk about `HTTP` methods. It's another piece of data the browser sends when it makes a request. Above we saw the browser sent the `URL` to the server. It also sends a method. The default method is called `GET`. There's nothing special about it. it's just a word. You can make up any words you want but there are 7 or 8 common ones and `GET` means "Get resource".

If you've ever made an `XMLHttpRequest` (and I hope you have because I'm not going to explain that part), you specify the method. Back on the server we could look at `request.method` to see what you specified and use that as yet another piece of data to decide what to do. If the method is `GET` we do one thing. If the method is BANANAS we do something else.

express has wrapped that http object from our first example and it adds a few major things.

1. it does more parsing of `request.url` for us so we don't have to do it manually.

2. it routes. Routing means we can tell it for any of various paths what function to call. For example we could say if the path starts with `/fruit` call the function `HandleFruit` and if the path starts with `/purchase/item/:itemnumber` then call `HandleItemPurchase` etc.. In our case we're going to just say we want all routes to call our function.

3. it can route based on method. That way we don't have to check if the method was `GET` or `PUT` or `DELETE` or `BANANAS`. We can just tell it to only call our handler if the path is `XYZ` and the method is `ABC`.

So let's update the code. `Ctrl-C` your server if you haven't already and edit `index.js` and update it to this
```
"use strict";
const express = require('express');
const path = require('path');
const fs = require('fs');
const baseDir = 'public';
 
let app = express();
app.put('*', function(req, res) {
    console.log("saving:", req.path);
    let body = '';
    req.on('data', function(data) { body += data; });
    req.on('end', function() {
        fs.writeFileSync(path.join(baseDir, req.path), body);
        res.send('saved');
    });
});
app.use(express.static(baseDir));
app.listen(8000, function() {
    console.log("listening at http://localhost:8000");
});
```
The first 2 added lines just reference more built in node libraries. path is a library for manipulating file paths. fs stands for "file system" and is a library for dealing with files.

Next we call `app.put` which takes 2 arguments. The first is the route and '*' just means "all routes". Then it takes a function to call for this route. app.put only routes `PUT` method requests so this line effectively says "Call our function for every route when the method is `PUT`".

The function adds a tiny event handler to the data event that reads the data the browser is sending by adding it to a string called body. It adds another tiny event handler to the end event that then writes out the data to a file and sends back the message 'saved'.

And that's it! We'd made a server that saves and loads files. It's very insecure because it can save and load any file but if we're only using it for local stuff it's a great start.

## Loading And Saving From the Browser

The final thing to do is to test it out by writing the browser side. I'm going to assume if you've already made some web pages and you're at the point where you want to load and save that you probably have some idea of what `XMLHttpRequest` is and how to make forms and check for users clicking on buttons etc. So with that in mind here's the new `test.html`
```
<html>
<head>
    <style>
    textarea {
        display: block;
    }
    </style>
</head>
<body>
 
<h1>Saving</h1>
<label for="savefilename">filename:</label>
<input id="savefilename" type="text" value="myfile.txt" />
<textarea id="savedata">
this is some test data
</textarea>
<button id="save">Save</button>
 
<h1>Loading</h1>
<label for="loadfilename">filename:</label>
<input id="loadfilename" type="text" value="myfile.txt" />
<textarea id="loaddata">
</textarea>
<button id="load">Load</button>
 
</body>
<script>
// make $ a shortcut for document.querySelector
const $ = document.querySelector.bind(document);
 
// when the user clicks 'save'
$("#save").addEventListener('click', function() {
 
    // get the filename and data
    const filename = $("#savefilename").value;
    const data = $("#savedata").value;
 
    // save
    saveFile(filename, data, function(err) {
        if (err) {
            alert("failed to save: " + filename + "\n" + err);
        } else {
            alert("saved: " + filename);
        }
    });
});
 
// when the user clicks load
$("#load").addEventListener('click', function() {
 
    // get the filename
    const filename = $("#loadfilename").value;
 
    // load 
    loadFile(filename, function(err, data) {
        if (err) {
            alert("failed to load: " + filename + "\n" + err);
        } else {
            $("#loaddata").value = data;
            alert("loaded: " + filename);
        }
    });
});
 
function saveFile(filename, data, callback) {
    doXhr(filename, 'PUT', data, callback);
}
 
function loadFile(filename, callback) {
    doXhr(filename, 'GET', '', callback);
}
 
function doXhr(url, method, data, callback) {
  const xhr = new XMLHttpRequest();
  xhr.open(method, url);
  xhr.onload = function() {
      if (xhr.status === 200) {
          callback(null, xhr.responseText);
      }  else {
          callback('Request failed.  Returned status of ' + xhr.status);
      }
  };
  xhr.send(data);
}
</script>
</html>
```
If you now save that and run your server then go to `http://localhost:8000/test.html` you should be able to type some text in the savedata area and click "save". Afterwords click "load" and you'll see it got loaded. Check your hard drive and you'll see a file has been created.

Now of course again this is insecure. For example if you type in `text.html` in the save filename and pick "save" it's going to overwrite your `text.html` file. Maybe you should pick a different route instead of "*" in the `app.put('*',` ... line. Maybe you want to add a check to see if the file exists with another kind of method and only update if the user is really sure.

The point of this article was not to make a working server. It was to show you how easy making a server is. A server like this that saves local files is probably only useful for things like an internal tool you happened to write in **JavaScript** that you and/or your team needs. They could run a small server like this on their personal machines and have your tool load, save, get folder listings, etc.

But, seeing how easy it is also hopefully demystifies servers a little. You can start here and then graduate to whole frameworks that let users login and share files remotely.

I should also mention you can load and save files to things like Google Drive or Dropbox. Now you know what they're basically doing behind the scenes.