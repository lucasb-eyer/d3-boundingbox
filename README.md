D3.js Bounding Box
==================

This is a [reusable component](http://bost.ocks.org/mike/chart/)
for [D3.js](http://d3js.org) which can give any element having `x`, `y`,
`width` and `height` attributes an interactive behaviour, similarly to
[D3's brush](https://github.com/mbostock/d3/wiki/SVG-Controls#brush)
and [interact.js](http://interactjs.io).

Available behaviours are:

- Movable: Dragging the element around,
- Resizable: Resizing the element by dragging the borders and/or corners.

Features of the library are:

- **Doesn't create elements**. Most approaches rely on creating child or sibling pseudo-elements; this one doesn't.
- Constraints on the allowed movements.
- Configurable behaviour: you choose what resize directions, if any, are allowed as well as whether dragging is allowed.
- Custom cursors for every action.
- Configurable "handle" size.
- Callbacks for all possible situations.
- Rock-solid behaviour: no "sticking", "losing" or "disappearing" problems.
    - Check out [test.html](http://lucasb.eyer.be/lab/d3-boundingbox.html)

The following are intentional non-features:

- No support for showing "handles".
    - This is due to the **don't create elements** design-decision.
    - Use CSS border and SVG stroke for that!
- Not well tested on mobile (yet), pull-requests welcome!
- Only targets modern browsers. I didn't care about ancient IE versions, though pull-requests are welcome.
- No taking care of your grandparents/children.

This library is part of a larger collection of D3.js utilities for supporting
the quick creation of browser-based interactive labeling tools.

Installation
============

You can install d3-boundingbox using npm by installing from this git repo:
```
npm install https://github.com/lucasb-eyer/d3-boundingbox.git
```

Usage
=====

Just like any [D3.js reusable component](http://bost.ocks.org/mike/chart/),
create the behaviour and call it on a collection:

```js
var bb = d3lb.bbox()
d3.selectAll("rect").call(bb)
// Alternatively
d3lb.bbox().infect(d3.selectAll("rect"))
```

That's it. Since a svg `rect` element supports the `x`, `y`, `width` and `height`
attributes, it has now become movable and resizable.

The return value of `call` is the D3.js selection it's been called on, so that
you can keep chaining calls to selection methods.
The return value of `infect` is the boundingbox itself, so that you can keep
chaining calls to the boundingbox's methods.

Note that the [test.html](http://lucasb.eyer.be/lab/d3-boundingbox.html)
file covers almost all possible usages, so you can use it as an example.

Also note that there's a `disinfect` function which can be used to remove
all registered event handlers from the element.

In the following, all setter methods can also be used as getters by not passing
in any value parameter.

Imposing a Constraint
---------------------

You can impose constraints, i.e. min/max x- and y-coordinates within which
the element should stay. You can either pass a pair of constant values:

```js
var bb = d3lb.bbox()
    .xextent([minx, maxx])
    .yextent([10, +Infinity])
d3.selectAll("rect.head").call(bb)
// Alternatively
d3lb.bbox().infect(d3.selectAll("rect.head"))
    .xextent([minx, maxx])
    .yextent([10, +Infinity])
```

or a function which will be called with the element's datum and index as
arguments and `this` bound to the element whenever a resize is attempted
and should return two values: the minimum and the maximum.
For example, you can restrict a head's bounding-box to stay within the
corresonding full-body bounding-box:

```js
d3lb.bbox().infect(d3.selectAll("rect.head"))
    .xextent(function(d, i) { return [d.body.left, d.body.right]; }
    .yextent(function(d, i) { return [d.body.top, d.body.bottom]; }
```

To get back to not having any constraints, call `{x,y}extent(false)`.

Also notice how you can use `Infinity` to stay unbounded in some direction.

Selecting Interactions
----------------------

All possible interactions have a name, and you can enable or disable each
individually. Here are their names:

- `x`, `y`: move the element horizontally or vertically by dragging it.
- `w`, `n`, `e`, `s`: resize the element by dragging its left, top, right, or
    bottom border, respectively.
- `nw`, `ne`, `se`, `sw`: resize the element by dragging its top-left,
    top-right, bottom-right or bottom-left corner, respectively.

Using the `directions` function, you can restrict the enabled interactions:

```js
var bb = d3lb.bbox().directions(['e', 'w', 'x'])
d3.selectAll("rect").call(bb)
// Alternatively
d3lb.bbox().infect(d3.selectAll("rect"))
    .directions(['e', 'w', 'x'])
```

The above will only allow the user interact with the rectangles horizontally.

To get back to the default setting, call `directions(true)`.

Custom Cursors
--------------

Since every design is different, you can customize which cursors should be shown
before and during any kind of interaction through the `cursors` function.

It is easy to disable any cursor modification by just calling `cursors(false)`.
the customization is done by passing an object with properties having the
interaction names mentioned above, and their values being the cursors to use:

```js
var bb = d3lb.bbox().infect(d3.selectAll("rect"))
    .directions(['e', 'w', 'x'])
    .cursors({
        x: 'url(dragx.cur), col-resize',
        e: 'url(szleft.cur), e-resize',
        w: 'url(szright.cur), w-resize'
    })
```

There is one additional name, `M`, which is used when dragging in both `x` and
`y` are enabled. This allows you to specify all three of `M`, `x` and `y`, at
initialization and enable/disable at will at runtime.

To get back to the default cursors, call `cursors(true)`.

Callbacks
---------

By registering callbacks through the `on` function, you can get notified of the following events:

- `dragstart`: when the mouse-button is being pressed before dragging starts.
- `dragmove`: when the mouse moves while dragging the element.
- `dragend`: when the mouse-button is released after having dragged the element.
- `resizestart`: when the mouse-button is being pressed on the border of the element.
- `resizemove`: when the mouse moves while dragging a border, i.e. resizing the element.
- `resizeend`: when the mouse-button is released after having resized the element.

All of these functions are passed D3.js' usual `d` and `i`, i.e. data and index
parameters of the element being affected, and set `this` to the element itself.

During any of the callbacks, you can access D3.js' global [`d3.event`](https://github.com/mbostock/d3/wiki/Selections#d3_event)
object and, for example, get the mouse coordinates via [`d3.mouse`](https://github.com/mbostock/d3/wiki/Selections#d3_mouse).

**Note:** If the `dragmove` and/or `resizemove` callbacks return exactly `false`
(not just a falsy value), the move will not happen. You can use this for even
more fine-grained control than you could achieve with the [constraints](#imposing-a-constraint).

This is how you'd add the `drag` class to an element during drag:

```js
var bb = d3lb.bbox().infect(d3.selectAll(".ninjas"))
    .on("dragstart", function(d, i) { this.classList.add("drag") })
    .on("dragend", function(d, i) { this.classList.remove("drag") })
```

Handle Size
-----------

Finally, you can choose how large, in pixels, the resize-handle should be,
either isometrically: `handlesize(5)`, or for each side individually:

```js
var bb = d3lb.bbox()
    .handlesize({
        'w': 3, 'e': 3,
        'n': 6, 's': 6
    })
```

Feedback
--------

It's always nice to hear what cool things people do with my libraries, so don't
hesitate to [let me know](http://lb.eyer.be) what you used it for!

If you have any suggestions, use-cases I missed, or anything, make sure to file
an issue or, better yet, make a pull-request!

License: MIT
------------

Copyright (c) 2015 Lucas Beyer

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
