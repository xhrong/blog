---
title: HTML5截图
tags: [HTML5,js,截图]
grammar_cjkRuby: true
categories: [Javascript]
date: 2019-06-29
---


### 截取DOM可见区域（html2canvas）

The script allows you to take "screenshots" of webpages or parts of it, directly on the users browser. The screenshot is based on the DOM and as such may not be 100% accurate to the real representation as it does not make an actual screenshot, but builds the screenshot based on the information available on the page.

#### Usage

The html2canvas library utilizes `Promise`s and expects them to be available in the global context. If you wish to
support [older browsers](http://caniuse.com/#search=promise) that do not natively support `Promise`s, please include a polyfill such as
[es6-promise](https://github.com/jakearchibald/es6-promise) before including `html2canvas`.

To render an `element` with html2canvas, simply call:
` html2canvas(element[, options]);`

The function returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) containing the `<canvas>` element. Simply add a promise fulfillment handler to the promise using `then`:

    html2canvas(document.body).then(function(canvas) {
        document.body.appendChild(canvas);
    });




#### How does it work
The script renders the current page as a canvas image, by reading the DOM and the different styles applied to the elements.

It does not require any rendering from the server, as the whole image is created on the client's browser. However, as it is heavily dependent on the browser, this library is not suitable to be used in nodejs. It doesn't magically circumvent any browser content policy restrictions either, so rendering cross-origin content will require a proxy to get the content to the same origin.

The script is still in a very experimental state, so I don't recommend using it in a production environment nor start building applications with it yet, as there will be still major changes made.


### 截取DOM完整区域（dom-to-image）

dom-to-image is a library which can turn arbitrary DOM node into a vector (SVG) or raster (PNG or JPEG) image, written in JavaScript. It's based on domvas by Paul Bakaus and has been completely rewritten, with some bugs fixed and some new features (like web font and image support) added.


#### Installation

##### NPM

`npm install dom-to-image`

Then load

```javascript
/* in ES 6 */
import domtoimage from 'dom-to-image';
/* in ES 5 */
var domtoimage = require('dom-to-image');
```

##### Bower

`bower install dom-to-image`

Include either `src/dom-to-image.js` or `dist/dom-to-image.min.js` in your page
and it will make the `domtoimage` variable available in the global scope.

```html
<script src="path/to/dom-to-image.min.js" />
<script>
  domtoimage.toPng(node)
  //...
</script>
```

#### Usage

All the top level functions accept DOM node and rendering options,
and return promises, which are fulfilled with corresponding data URLs.  
Get a PNG image base64-encoded data URL and display right away:

```javascript
var node = document.getElementById('my-node');

domtoimage.toPng(node)
    .then(function (dataUrl) {
        var img = new Image();
        img.src = dataUrl;
        document.body.appendChild(img);
    })
    .catch(function (error) {
        console.error('oops, something went wrong!', error);
    });
```

Get a PNG image blob and download it (using [FileSaver](https://github.com/eligrey/FileSaver.js/),
for example):

```javascript
domtoimage.toBlob(document.getElementById('my-node'))
    .then(function (blob) {
        window.saveAs(blob, 'my-node.png');
    });
```

Save and download a compressed JPEG image:

```javascript
domtoimage.toJpeg(document.getElementById('my-node'), { quality: 0.95 })
    .then(function (dataUrl) {
        var link = document.createElement('a');
        link.download = 'my-image-name.jpeg';
        link.href = dataUrl;
        link.click();
    });
```

Get an SVG data URL, but filter out all the `<i>` elements:

```javascript
function filter (node) {
    return (node.tagName !== 'i');
}

domtoimage.toSvg(document.getElementById('my-node'), {filter: filter})
    .then(function (dataUrl) {
        /* do something */
    });
```

Get the raw pixel data as a [Uint8Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array)
with every 4 array elements representing the RGBA data of a pixel:

```javascript
var node = document.getElementById('my-node');

domtoimage.toPixelData(node)
    .then(function (pixels) {
        for (var y = 0; y < node.scrollHeight; ++y) {
          for (var x = 0; x < node.scrollWidth; ++x) {
            pixelAtXYOffset = (4 * y * node.scrollHeight) + (4 * x);
            /* pixelAtXY is a Uint8Array[4] containing RGBA values of the pixel at (x, y) in the range 0..255 */
            pixelAtXY = pixels.slice(pixelAtXYOffset, pixelAtXYOffset + 4);
          }
        }
    });
```

* * *

_All the functions under `impl` are not public API and are exposed only
for unit testing._

* * *

#### Rendering options

##### filter

A function taking DOM node as argument. Should return true if passed node
should be included in the output (excluding node means excluding it's
children as well). Not called on the root node.

##### bgcolor

A string value for the background color, any valid CSS color value.

##### height, width

Height and width in pixels to be applied to node before rendering.

##### style

An object whose properties to be copied to node's style before rendering.
You might want to check [this reference](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Properties_Reference)
for JavaScript names of CSS properties.

##### quality

A number between 0 and 1 indicating image quality (e.g. 0.92 => 92%) of the
JPEG image. Defaults to 1.0 (100%)

##### cacheBust

Set to true to append the current time as a query string to URL requests to enable cache busting. Defaults to false

##### imagePlaceholder

A data URL for a placeholder image that will be used when fetching an image fails. Defaults to undefined and will throw an error on failed images

#### How it works

There might some day exist (or maybe already exists?) a simple and standard
way of exporting parts of the HTML to image (and then this script can only
serve as an evidence of all the hoops I had to jump through in order to get
such obvious thing done) but I haven't found one so far.  

This library uses a feature of SVG that allows having arbitrary HTML content
inside of the `<foreignObject>` tag. So, in order to render that DOM node
for you, following steps are taken:  

1.  Clone the original DOM node recursively

2.  Compute the style for the node and each sub-node and copy it to
    corresponding clone

    -   and don't forget to recreate pseudo-elements, as they are not
        cloned in any way, of course

3.  Embed web fonts

    -   find all the `@font-face` declarations that might represent web fonts

    -   parse file URLs, download corresponding files

    -   base64-encode and inline content as `data:` URLs

    -   concatenate all the processed CSS rules and put them into one `<style>`
        element, then attach it to the clone

4.  Embed images

    -   embed image URLs in `<img>` elements

    -   inline images used in `background` CSS property, in a fashion similar to
        fonts

5.  Serialize the cloned node to XML

6.  Wrap XML into the `<foreignObject>` tag, then into the SVG, then make it a
    data URL

7.  Optionally, to get PNG content or raw pixel data as a Uint8Array, create an
    Image element with the SVG as a source, and render it on an off-screen
    canvas, that you have also created, then read the content from the canvas

8.  Done!  

#### Things to watch out for

-   if the DOM node you want to render includes a `<canvas>` element with
    something drawn on it, it should be handled fine, unless the canvas is
    [tainted](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_enabled_image) -
    in this case rendering will rather not succeed.  

-   at the time of writing, Firefox has a problem with some external stylesheets
    (see issue #13). In such case, the error will be caught and logged.  

### 截取全页面

#### 利用html2canvas

```javascript
  html2canvas(document.body).then(function (canvas) {
                document.body.appendChild(canvas);
  });
			
```
说明：动态添加的元素会是空白



#### 利用dom-to-image




```javascript
var node = document.body;

domtoimage.toPng(node)
    .then(function (dataUrl) {
        var img = new Image();
        img.src = dataUrl;
        document.body.appendChild(img);
    })
    .catch(function (error) {
        console.error('oops, something went wrong!', error);
    });
```
说明：效果有时也不理想


### 截取Canvas

调用canvas.toDataURL()，获取canvas数据后，保存为图片

有如下`<canvas>`元素

```javascript
<canvas id="canvas" width="5" height="5"></canvas>
```

可以用这样的方式获取一个 data-URL

```javascript
var canvas = document.getElementById("canvas");
var dataURL = canvas.toDataURL();
console.log(dataURL);
// "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNby
// blAAAADElEQVQImWNgoBMAAABpAAFEI8ARAAAAAElFTkSuQmCC"
```

设置jpeg图片的质量节

```javascript
var fullQuality = canvas.toDataURL("image/jpeg", 1.0);
// data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...9oADAMBAAIRAxEAPwD/AD/6AP/Z"
var mediumQuality = canvas.toDataURL("image/jpeg", 0.5);
var lowQuality = canvas.toDataURL("image/jpeg", 0.1);
```


### Chrome截图

Chrome截图方法：
1、ctrl + shift + i
2、ctrl +shift + p
3、搜索"screen"

![enter description here](./images/1561804207895.png)

### 参考资源
1、https://github.com/tsayen/dom-to-image
2、https://blog.csdn.net/u012260672/article/details/79302465
3、https://github.com/niklasvh/html2canvas/
4、http://caibaojian.com/html2canvas.html
5、http://html2canvas.hertzen.com/
6、https://www.jb51.net/article/128554.htm