The pieces in Tetris should be familiar to anyone who have played the game.
What you may not know though is that they all have names, assigned colors, and
specific [rotation patterns](http://gamedev.stackexchange.com/a/17978/57215).
We want our Tetris clone to be as true to the real game as possible, and so we
will need to implement all these things correctly.

Let us first of all consider the tetrominoes: I, J, L, O, S, T, and Z. Their
names are chosen so as to resemble their shape; I is the 4x1 stick, J is the
3x1 stick with a block at the bottom left, L is similar, but with the block
bottom right, etc. You can see the full set of tetrominoes
[here](http://vignette1.wikia.nocookie.net/tetrisconcept/images/3/3d/SRS-pieces.png/revision/latest?cb=20060626173148),
along with their rotation patterns.

## Building the tetrominoes

After looking at the image linked to above, it should strike you that, if you
ignore the color, each piece is basically just a Boolean matrix. For example, I
in its initial configuration can be considered a 4x4 matrix where all the
values in the second row are set to true, and the remaining cells are set to
false. That seems like a nice, clean way of storing them, so let's try that.

We already know how to store matrices. They're just lists of lists. While we're
at it, we might also want to store all the different rotations of each piece
too. We can do that by storing a list of matrices for each tetromino, and then
rotation will just be moving through that list. This sounds like a great plan!
If you don't want to write it yourself, I've already done it for you in
[tetrominoes.js](../tetrominoes.js). Go have a look -- there shouldn't be
anything particularly surprising in there. I'll wait.

## Making tetrominoes do things

Only one tetromino will be active on the game board at any given time. All the
pieces that previous tetrominoes have left are incorporated into the game
board, and will be drawn using the `drawBoard` function we defined earlier. To
draw the currently active piece, we could simply put a for loop where we end up
calling `drawBoard` from that iterates over all the cells of the active piece's
shape matrix, and calls `drawSquare` for each one that is set to true. While
this would certainly work, consider what would happen if we later on wanted to
draw a piece somewhere else too. For example, Tetris generally shows you which
piece is coming up next in the corner of the game screen, and if we wanted to
do that, we would now have to duplicate the tetronimo drawing code in two
places, which is poor form.

Instead, we will let the tetrominoes know how to draw *themselves*. That way,
we could just do something like `piece.draw()` after we call `drawBoard`, and
we could use the same piece of code if we wanted to show a preview of the next
piece. If you're already guessing that this is related to what we discussed
briefly earlier on in the tutorial about attaching functions to variables, go
pat yourself on the back. You're completely right. If you didn't think of that,
don't worry. All will be clear.

### Object-oriented programming

Object-oriented programming (or OOP) is a complex topic, but we will cover only
the parts that are of immediate relevance to us for our Tetris game here. In
OOP, "things" in your program are modelled as *objects*, and things that are
similar are considered *instances* of a given *class*. For example, you might
have the class `Dog`, and any instance (variable) of that class would be able
to do what a `Dog` can do. In JavaScript, creating an instance of a class is
done by using the `new` keyword like this: `var fido = new Dog();`. Similarly,
we could define the class `Piece`, and have tetrominoes be instances of the
class Piece (e.g. `var piece = new Piece(shape_matrix_for_I);`).

Why is this of any interest to us? Well, a class can define functions and
variables that are attached to each of its instances. For example, the `Piece`
class could have a function called `draw` that would draw the piece at its
current coordinates. It could also have two variables, `x` and `y`, that store
the piece's current location. Naturally, each instance would also need to know
its shape, otherwise drawing it will be hard.

Let's jump in and see what this might look like.

```javascript
function Piece(patterns, color) {
	this.pattern = patterns[0];
	this.patterns = patterns;
	this.patterni = 0;

	this.color = color;

	this.x = 0;
	this.y = -2;
}

Piece.prototype.draw = function() {
	fs = ctx.fillStyle;
	ctx.fillStyle = this.color;
	for (var ix = 0; ix < this.pattern.length; ix++) {
		for (var iy = 0; iy < this.pattern.length; iy++) {
			if (this.pattern[ix][iy]) {
				drawSquare(this.x + ix, this.y + iy);
			}
		}
	}
	ctx.fillStyle = fs;
};
```

There's a lot of weird stuff going on here, so let's go through it piece by
piece (hehe, see what I did there?). First, a class in JavaScript is created
simply by making a function. By convention, class functions are named with the
first letter capitalized. There is nothing special about class functions that
make them classes. What makes them classes is that the developer uses the `new`
keyword when they call it, indicating to the interpreter that they want an
object (instance of the class) instead of just calling the function. This is
somewhat different to how classes work in "proper" OOP languages such as Java,
but they way they're used is fairly similar, so don't worry about it too much.

So, we have our `Piece` class. Notice that is refers a lot to `this`. `this` is
a special keyword that refers to, unsurprisingly, *this* object. If we write
`var fido = Dog();`, then `this` inside `Dog()` will refer to the `fido`
object. `Piece` also takes two arguments: the shape (or rather, shapes, because
we pass it all the different rotations) to use, and the color of the tetromino.
Pattern is used in place of shape in the code, but this is purely a formality.
We store both of these, as well as the currently active pattern inside `this`.
`patterni` is simply the index into `patterns` that tells us which pattern
we're currently using. We start out by using the first one.

`Piece()` also sets up the initial `x` and `y` coordinates of the piece. We
start the piece two rows above the gameboard (`y = -2`) so that it is
invisible, and we start it off all the way to the left (`x = 0`). In reality,
tetrominoes should spawn in the center of the screen, but the code for this is
slightly ugly, so it's been left out here. If you look at
[tetris.js](../tetris.js), you'll see the full expression for centering it.

There's also a line there that mentions `draw`; the function we said we were
going to implement. The contents of it also looks quite familiar -- it's almost
identical to how we wrote `drawBoard`! The only minor differences are that we
are looping over the current piece's pattern (`this.pattern`), and that we are
adding `this.x` and `this.y` to the values we pass to `drawSquare`. The reason
for this is that when we loop over the blocks set to true in the pattern, we're
only getting local coordinates for them (for example, `(1,1)` is true for the
horizontal I). When drawing the piece on the game board, it may have moved
further down the board (say it's at `y = 5`), and then we actually need to draw
`(1,1)` of the piece at `(6,1)`.

Now let's inspect the definition of the `draw` function, because it looks kind
of funny.

```javascript
Piece.prototype.draw = function() {
  ...
};
```

What's this prototype stuff? Well, [it's somewhat
complicated](https://sporto.github.io/blog/2013/02/22/a-plain-english-guide-to-javascript-prototypes/),
so we're going to just ignore it for now. Just think of it as a magical way of
saying "the function `draw` should be present on all instances of the class
`Piece`".

### Moving the tetrominoes

If tetrominoes were stationary, Tetris would not be a particularly interesting
game. We need them to drop down, we need them to be able to move left and
right, and we need them to be rotatable. Now that we know about classes and
methods though, this should be a breeze:

```javascript
Piece.prototype.down = function() {
	this.undraw();
	this.y++;
	this.draw();
};

Piece.prototype.moveRight = function() {
	this.undraw();
	this.x++;
	this.draw();
};

Piece.prototype.moveLeft = function() {
	this.undraw();
	this.x--;
	this.draw();
};

Piece.prototype.rotate = function() {
	this.undraw();
	this.patterni = (this.patterni + 1) % this.patterns.length;
	this.pattern = this.patterns[this.patterni];
	this.draw();
};
```

Rotate might look a bit weird, as it is using the modulo (`%`) operator. This
operator is very useful in programming, as it makes a counter "wrap around"
when it reaches a certain number. For example, if we are counting "modulo 4",
then if we iterate through all integral numbers, we would get 0, 1, 2, 3, 0, 1,
2, 3, 0, etc. When the user rotates a piece, this is precisely what we want.
When the last rotation is being used, rotating again should bring them back to
the first rotation pattern (`[0]`).

But what's this `undraw` function? Well, if we just drew the tetromino in
its new location, and did not draw anything where it used to be, the square we
drew before it moved would still be in place, and the user would be very
confused, as the tetromino would appear to be growing in size! To avoid this,
we first clear all the squares of the tetromino in its current position (this
is the same as filling them with the "none" color; in our case, black). The
code to do so would be identical to that in `draw`, so to make life simpler for
ourselves, we can instead define a helper method, and then make `draw` and
`undraw` just call this helper function. There should be nothing particularly
surprising here:

```javascript
Piece.prototype._fill = function(color) {
	fs = ctx.fillStyle;
	ctx.fillStyle = color;
	var x = this.x;
	var y = this.y;
	for (var ix = 0; ix < this.pattern.length; ix++) {
		for (var iy = 0; iy < this.pattern.length; iy++) {
			if (this.pattern[ix][iy]) {
				drawSquare(x + ix, y + iy);
			}
		}
	}
	ctx.fillStyle = fs;
};

Piece.prototype.undraw = function(ctx) {
	this._fill("black");
};

Piece.prototype.draw = function(ctx) {
	this._fill(this.color);
};
```

But wait a second, I hear you protest. What if we move the piece off the board?
Or into another block? You're completely right; that's something we need to
consider. This is generally referred to as collision detection, and it's so
important it gets its own [section](collisions.md)!
