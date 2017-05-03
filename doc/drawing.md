If you open [tetris.js](../tetris.js), you'll see that there's a bunch of
nonsense-looking lines at the top. The ones we're going to discuss here are
repeated below for convenience, followed by a description of what they do.

## Drawing setup

```javascript
var canvas = document.getElementById("board");
var ctx = canvas.getContext("2d");
```

This is a magical incantation you use in JavaScript to get a *2D drawing
context* for a HTML canvas. That's just a fancy way to say a thing you can use
to draw things to screen. Since we know about variables by now, we can tell
that `canvas` and `ctx` are both variables in the above code, but what are
their values? `canvas` is being set to the result of *calling* the
`getElementById` *function* on the variable `document` with the *argument*
`"board"`.

Okay, lots of new terminology there. It's about time we introduce the concept
of *functions*.

### Functions

Based on the code we've been exposed to thus far, programs appear to run
top-to-bottom. Each line is executed, and once it has been (potentially
multiple times if in a loop), the next line is executed. This then continues
until the last line in the file. For all but the most trivial applications,
this would quickly become unmanageable. Some pieces of code need to be repeated
in many different places (for example the code to draw a square on screen), and
copy-pasting it in results in harder-to-maintain code (if you need to change
it, you need to change it everywhere), and is quite inconvenient.

Functions are one of the ways that this problem is mitigated. A function is
essentially a named block of code. Whenever you want those lines of code to
execute, you simply write the name of the function, and the interpreter will
jump to the first of the lines in that block, execute them one after the other,
and at the end of the block, jump back to where you *called* the function from.

Functions also commonly take one or more *arguments*. An argument is additional
info that is passed into the function. For example, a `sum` function might take
two arguments `x` and `y`, and return their sum. Consider the following
function, and how it would be called:

```javascript
function sum(x, y) {
    z = x + y;
    return z;
}

four = sum(1, 3);
```

The `return` statement you see at the end of the function take a value (or the
value contained inside a variable), and makes that be the value of the function
when it is called. In the above code snippet, `four` would thus be assigned the
numerical value `4`.

You'll also note that I said the function is called *on* the variable
`document`. Functions can be "attached" to variables (we'll get to the exact
details of this way later in this tutorial), and when that function is then
called on the variable, the variable itself is passed in as a special, magical
argument referred to simply as `this`. These kinds of functions are generally
called *methods*.

### Back to the code

Where were we? Oh, yes:

```javascript
var canvas = document.getElementById("board");
var ctx = canvas.getContext("2d");
```

So, `canvas` will be assigned whatever value calling the `getElementById`
method on `document` with the argument `"board"` *return*s. This particular
function is a part of the programming *interface* made available to
applications by the browser, and is a part of the [Document Object
Model](https://developer.mozilla.org/fi/docs/DOM). This interface is
**massive**, and covering it in any kind of detail is outside the scope of this
tutorial, so we'll just pretend that this call will return the element called
"board" from our `.html` file, which happens to be the HTML canvas that we want
to draw in. The standard further dictates that, in order to draw on an HTML
canvas, we need to have a drawing "context". This context will hold information
about which color we're drawing with, how wide the strokes are, what font we're
using, etc. We'll put this abstract drawing context thingy in the `ctx`
variable so we can refer to it later.

Let's continue down the file.

```javascript
var width = 10;
var height = 20;
var tilesz = 24;
```

These are just plain ol' variables with numbers in them. Their names should be
pretty self explanatory; `width` is the width of the game board, `height` is
the height of the game board, and `tilesz` is the number of *pixels* each
tetris square is in each direction. A *pixel* is, [roughly
speaking](http://inamidst.com/stuff/notes/csspx), the smallest visual element
on a computer screen. It's one "dot" if you will. We're saying that each of our
squares should be 24x24 pixels large. Try changing it and see what happens!

```javascript
canvas.width = width * tilesz;
canvas.height = height * tilesz;
```

If we're going to draw a 20x10 grid of tiles, each 24x24 pixels in size, we
need to make sure the canves we're drawing on is large enough. These lines
change the width and height of the HTML canvas to be the appropriate size. Note
that I said the width *of* the canvas. Similar to how we can assign functions
to variables, we can also assign variables to variables. This is the basis of
Object-Oriented Programming, something we'll be exposed to later in this
tutorial.

```javascript
function drawSquare(x, y) {
	ctx.fillRect(x * tilesz, y * tilesz, tilesz, tilesz);
	ss = ctx.strokeStyle;
	ctx.strokeStyle = "#555";
	ctx.strokeRect(x * tilesz, y * tilesz, tilesz, tilesz);
	ctx.strokeStyle = "#888";
	ctx.strokeRect(x * tilesz + 3*tilesz/8, y * tilesz + 3*tilesz/8, tilesz/4, tilesz/4);
	ctx.strokeStyle = ss;
}
```

Ah, a real function! How exciting.
This function draws every single tile you see on the game board when you play
the game. Each tile is drawn using three squares: a filled square covering the
entire tile with a color, a hollow (stroked) gray square tracing the outer edge
of the tile, and a smaller gray stroked square in the center of the tile. Let's
see how these are drawing using this function.

First, we notice that the function is passed two arguments, `x` and `y`, the
tile position we want to draw squares in. To draw a filled square, we call the
`fillRect` method on `ctx` that we defined above. This method takes four
arguments, `x`, `y`, `width`, and `height`. Since the arguments we got were
coordinates, and not pixels, we multiply by the `tilesz` to get the pixel
coordinates to pass to `fillRect`. Since we want to fill the entire tile, we
pass the tile size in pixels as both the width and height of the rectangle to
draw.

For the fill color, we assume that whomever calls `drawSquare` will already
have set the correct fill color. The stroke color on the other hand, we always
want to be gray. This can be controlled by changing the value of `strokeStyle`
on `ctx`. However, we also want to be nice to the code that called us, and make
sure that when we return to the calling code, `strokeStyle` will be set to
whatever it was before we were called. We do this by first storing the old
value to a variable (`ss = ctx.strokeStyle`), and then resetting it before the
function finishes (`ctx.strokeStyle = ss`).

To draw the hollow squares, we use the `strokeRect` function, which takes the
same arguments as `fillRect`. The call to draw the edges is pretty
straightforward, and the second is only somewhat complicated by the fact that
it only draws in the center fourth of the tile. The colors we use (`#555` and
`#888`) are [hexadecimal
representations](https://en.wikipedia.org/wiki/Web_colors#Hex_triplet) of the
amount of Red, Green and Blue (RGB colors) in the color. The higher the number,
the more of that color. When `R = G = B`, we get gray, and the higher the
number, the lighter the gray (in RGB, adding all colors together at full
intensity gives white).

## Drawing the board

To draw the board itself, we define another function, `drawBoard`, that uses
the `board` matrix we defined in the previous part of the tutorial.

```javascript
function drawBoard() {
	fs = ctx.fillStyle;
	for (var y = 0; y < height; y++) {
		for (var x = 0; x < width; x++) {
			ctx.fillStyle = board[y][x] ? 'red' : 'white';
			drawSquare(x, y, tilesz, tilesz);
		}
	}
	ctx.fillStyle = fs;
}
```

Notice that we're doing the same kind of resetting for `fillStyle` here as we
did in `drawSquare`. The nested loop is also almost identical to the one we
used when constructing the board. This makes sense, as we're doing the same
thing; looping over all the tiles. The only difference is that instead of
hard-coding the number of rows and columns, we use the `height` and `width`
variables we defined above. Doing so allows us to easily change the size of the
board without needing to change any of the other code.

What we do for each tile in the loop is quite different from what we did when
setting up the board initially though. However, by now, you may be able to
guess what it does. The only new concept here is that of conditionals, which
it's about time I told you about, as they are essential to any real-world
programming.

### Flow control

Very often in computer programs, you want the program to do different things
depending on input it gets from the user, the current time, what file it is
given, etc. For example, you may want to show an error if the user gives you an
invalid email address, or show the current year somewhere. Programming
constructs that let you do this are called *flow control statements*, because
they direct the *flow* of the application. The most basic kind of flow control
is binary branching, often simply called *if-statements*. Let's see an example
based on the code above.

```javascript
if (board[y][x] == true) {
  ctx.fillStyle = 'red';
} else {
  ctx.fillStyle = 'white';
}
```

When the interpreter reaches an `if` statement, it will look at the expression
inside the parentheses, and if it turns out to be true, the lines in the first
`{}` block will be executed, otherwise the lines in the second `{}` block will
be executed. The `else` keyword is there to say that the second `{}` should
always be executed if the condition is false. This is necessary due to a
variation on the simple binary `if` statement:

```javascript
if (a) {
  // Do something when a is true
} else if (b) {
  // Do something when a is false and b is true
} else {
  // Do something when a is false and b is false
}
```

If you don't want to do anything if the condition is false, you can simply
leave out the `else { ... }` part.

The conditions themselves warrant some explanation. These use something called
Boolean logic, that lets you construct arbitrarily complex constraints on
whether a particular block should be executed (this is called *taking a
branch*). A simply equality constraint was show above; `board[y][x] == true`.
This will evaluate to true if, and only if, `boad[y][x]` has been set to
`true`. If it is set to `false`, the condition will be false, and the else
branch, if it exists, will be taken. Program execution will then continue
normally below the if. Conditions can also be combined by using `&&` (which
means *and*; the expression is true if this is true *and* that is true) and
`||` (which means *or*; the expression is true if this is true *or* that is
true). A number of constraints are available: `==` for equality, `!=` for
inequality, `<` and `<=` for less-than and less-than-or-equal, `>` and `>=` for
greater-than and greater-than-or-equal, and a bunch of others. We'll tackle
them as we go along.

"That is all very well, but what is this `= ? :` business?", I hear you cry.
Relax, I'll answer that right now. `?:`, as it is often referred to as, is the
*ternary conditional assignment operator*. That's a mouthful, but it is
actually quite descriptive. *Ternary* means that it takes three arguments,
*conditional* means that it does different things depending on its arguments,
*assignment* is used because it returns a value that is usually *assigned* to a
variable, and finally, *operator* just means that it does something.
Specifically, `?:` will return its second argument if the first is true, and
the third otherwise. It's effectively a condensed version of an `if-else`
statement where both blocks assign some value to a variable. For example, the
if from above can be succinctly written as

```javascript
ctx.fillStyle = board[y][x] == true ? 'red' : 'white';
```

Furthermore, `board[y][x] == true` can be simplified to just `board[y][x]`,
since comparing something with `true` will
only<sup>[*](https://developer.mozilla.org/en/docs/Web/JavaScript/Guide/Sameness)</sup>
ever be considered true if the expression being checked is true, we can just
leave the comparison out. This simplified variant is what is used in
`drawBoard` above.

So now we've completely covered `drawBoard`, and that completes our discussion
of the drawing code for the game. Next, we'll move on to [working with
pieces](pieces.md).

