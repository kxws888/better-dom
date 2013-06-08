better-dom [![Build Status](https://api.travis-ci.org/chemerisuk/better-dom.png?branch=master)](http://travis-ci.org/chemerisuk/better-dom)
==========
Modern javascript library for working with DOM.
Making DOM to be nice

JSDoc - http://chemerisuk.github.io/better-dom/

Overview
--------
Everybody who manipulated DOM via vanilla javascript knows that it is an awful API. Current specification has bugs, browser behavior incosistences etc. The library tries to fix that: it introduces it's own more friednly types for document nodes with developer-fiendly APIs.

Important to note that it covers only DOM so, for instance, there are no methods for working with AJAX.

Installation
------------
Use [bower](http://bower.io/) to download the library with its dependencies:

    bower install better-dom

This will clone the latest version of the better-dom into the `components` directory at the root of your project.

Then include the library on your page with the script below:

```html
<!DOCTYPE html>
<html>
<head>
    <!-- force IE to use the best version of the browser engine -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    ...    
</head>
<body>
    ...
    <script>
    (function(D,O,M){M=D.documentElement;M.addBehavior?(M.setAttribute("data-htc",
    O=O.substr(0,O.length-2)+"htc"),M.addBehavior(O)):(M=D.createElement("script"),
    M.async=!1,M.src=O,D.head.appendChild(M))})(document,"/components/better-dom/better-dom.js");
    </script>  
</body>
</html>
```

Compatablity
------------
The library introduces it's own objects for working with DOM. It doesn't modify any native prototypes. `DOM` is actually the only one global variable.

Moreover you can use DOM extentions with any other AJAX-library because they are...

Ajax-friendly
-------------
The idea is to write DOM plugins in declaratively. `DOM.extend` is used to implement a new extension and the coolest thing is that any matched HTML element is automatically captured by the library even in dynamic content. As a result it's much simpler to write [polyfills](#quick-example-placeholder-polyfill) or to create your own web components for a single-page website.

Event handling
--------------
Events handling is a big part of writing code for DOM. And there are some features included to the library that force developers to use best practicises.

> Get rid of the event object
 
Event handlers don't own an event object now and this thing improves testability of your code:

```js
// NOTICE: handler don't have e as the first argument
DOM.find("#link").on("click", function() {...});
// NOTICE: options argument
DOM.find("#link").on("keydown", function(keyCode, altKey) {...}, {args: ["keyCode", "altKey"]});
```

> Call preventDefault() or stopPropagation() before logic

It's a common situation to work with unsafe code that can throw an exception. If preventDefault() or stopPropagation() are called at the end of logic than program may start to work unexpected.

```js
// NOTICE: preventDefault is always called before the handler
DOM.find("#link").on("click", handler, {cancel: true});
// NOTICE: stopPropagation os always called before the handler
DOM.find("#link").on("click", handler, {stop: true});
```

> Callback systems are brittle

If any of the callback functions throw an error then the subsequent callbacks are not executed. In reality, this means that a poorly written plugin can prevent other plugins from initialising (read  http://dean.edwards.name/weblog/2009/03/callbacks-vs-events/ for additional details).

```js
DOM.ready(function() { throw Error("exception in a bad code"); });
// NOTICE: you'll always see the message in console
DOM.ready(function() { console.log("Nothing can break your code") });
```

Performance
-----------
DOM is usually the main bottleneck of javascript applications. Therefore performance question should be on the top for any library that works with it.

Quick example: placeholder polyfill
-----------------------------------
It's pretty simple to write polyfills, because you do not need to worry about monitoring if an appropriate element found in DOM:

```js
DOM.supports("placeholder", "input") || DOM.extend("[placeholder]", {
    holder: "<input type='text' style='box-sizing: border-box; position: absolute; color: graytext; background: none no-repeat 0 0; border-color: transparent'/>"
}, {
    constructor: function(tpl) {
        var offset = this.offset(),
            holder = tpl.holder;

        holder
            .set(this.get("placeholder"))
            .setStyle("width", offset.right - offset.left)
            .on("click", this.fire, ["focus"], this);

        this.on("focus", holder.hide, [], holder);
        this.on("blur", this._showPlaceholder, [holder]);

        if (this.get() || this.isFocused()) holder.hide();

        this.before(holder);
    },
    _showPlaceholder: function(holder) {
        if (!this.get()) holder.show();
    }
});
```

See it in action: http://chemerisuk.github.io/better-placeholder-polyfill/ (open in IE < 10)

More complex example: elastic textarea
--------------------------------------
This is a textarea extension which autoresizes textarea to contain all text:

```js
DOM.extend("textarea.elastic", {
    wrapper: "div[style=position:relative]>pre[style=visibility:hidden;margin:0;border-style:solid]>span[style=display:inline-block;white-space:pre-wrap]"
}, {
    constructor: function(tpl) {
        var wrapper = tpl.wrapper,
            holder = wrapper.child(0),
            span = holder.child(0);

        holder.setStyle({
            font: this.getStyle("font"),
            padding: this.getStyle("padding"),
            "border-width": this.getStyle("border-width")
        });

        this.on("input", this._syncWithHolder, [span]);
        this._syncWithHolder(span);

        this.parent("form").on("reset", this._syncWithHolder, [span, true], this);

        wrapper.append(this.after(wrapper));
    },
    _syncWithHolder: function(span, defaultValue) {
        value = this.get(defaultValue ? "defaultValue" : "value");

        // use &nbsp; to fix issue with adding a new line
        if (value[value.length - 1] === "\n") value += "&nbsp;";
        
        // IE doesn't respect newlines so use <br> instead
        span.set(value.split("\n").join("<br>"));
    }
});

DOM.importStyles("textarea.elastic", {
    position: "absolute",
    left: 0,
    top: 0,
    width: "100%",
    height: "100%",
    overflow: "hidden",
    resize: "none",
    "box-sizing": "border-box"
});
```
See it in action: http://chemerisuk.github.io/better-elastic-textarea/

Another example: dateinput polyfill
-----------------------------------
See the extension repository at https://github.com/chemerisuk/better-dateinput-polyfill.

Specs examples (using jasmine) are included.

Browser support
---------------
* Chrome
* Safari
* Firefox
* Opera
* IE8+
