No game is particularly interesting if it does not take input from the user,
whether it be through a mouse pointer, touch, the keyboard, or other sensors.
For our tetris clone, we're going to keep things simple, and let the entire
game be controlled through the arrow keys. Up will rotate, left and right will
move the piece left and right, and down will move the piece down.

## User input

Getting user input is very language dependent, and is possibly one of the
things that varies most between languages. Most languages also have several way
input can be read. For web pages, input is handled through *events*. The
browser fires of an event whenever something happens, and attaches related
information to it. A browser application can *listen* for such events, and then
optionally do something when they happen.

For our game, we're only interested in *keyboard events*. In particular, we
care about *key press* events (which occurs when a letter would be typed in a
text box). There are others such as *key down*, *key up*, but we will be
ignoring them for now. Along with a key press event, we get the key code of the
key that was pressed. These key codes are mostly the same across browsers, but
[not always](http://www.javascripter.net/faq/keycodes.htm). We care about the
keys 37, 38, 39, and 40, which correspond to left, up, right, and down
respectively.

When registering interest in an event, we also need to tell the browser *where*
we care about an event happening. Since our game is the only thing on the page,
we want to hear about keyboard events regardless of where they happen, and we
thus use the `body` element of the `document`, which refers to "the whole
page". The full code for our key press event listener looks like this:

```javascript
var dropStart = Date.now();
document.body.addEventListener("keypress", function (e) {
	if (e.keyCode == 38) { // Player pressed up
		piece.rotate();
		dropStart = Date.now();
	}
	if (e.keyCode == 40) { // Player holding down
		piece.down();
	}
	if (e.keyCode == 37) { // Player holding left
		piece.moveLeft();
		dropStart = Date.now();
	}
	if (e.keyCode == 39) { // Player holding right
		piece.moveRight();
		dropStart = Date.now();
	}
}, false);
```

Hopefully, this should start to look quite familiar by now. `piece` refers to
the currently falling piece, and `dropStart` is a variable we use to keep track
of the last time the user made a move. The reason for this is that, in Tetris,
a piece **will not drop** while a user is making a move. How `dropStart` is
used will soon become clear.

## The Game Loop

In most games, there is a *central game loop*. This is the (never-ending) loop
that drives the game forward, either by acting on user input, or having things
change over time. Since we handle inputs when they happen, all that is really
left for the main loop is to make the falling piece drop down once every so
often. This is handled by the `main()` function.

```javascript
var done = false;
function main() {
	var now = Date.now();
	var delta = now - dropStart;

	if (delta > 1000) {
		piece.down();
		dropStart = now;
	}

	if (!done) {
		requestAnimationFrame(main);
	}
}
main();
```

We calculate how much time has passed since the last time a piece dropped, or
the last time the user made a move, and if that is more than 1000 milliseconds
ago (= 1 second), then we move the piece down and reset our timer. Not too
complicated. We also call this strange `requestAnimationFrame` function. This
is a magical function in JavaScript that will call the function you pass it
again the next time the browser wants to re-render the page to make animations
look smooth. The exact details are outside the scope of this tutorial, but just
think about it as though we're simply calling main() from main(), thus giving
us an infinite loop.

The `done` condition around the call to main is so that we can end the game
when the user loses. We will see how that happens later, but we now know that
all we have to do in order for it to end is to set `done = true`.

## Locking pieces in place

When a piece tries to move down, but doing so would make any block in the piece
overlap with a block on the board, the piece should instead be placed where it
currently is, and a new piece should be spawned. Furthermore, if putting the
new piece in place caused any row on the game board to be filled, that row
should be eliminated, and all rows higher up on the board should be moved down.

If you remember back to the chapter on [collision detection](collisions.md),
you might remember that our `Piece.down()` function already detects this
scenario, so all we need to do is put whatever we want to happen when a piece
should be locked in place there. Because we like functions, we'll create a new
function on `Piece`  called `lock`, and then just add `piece.lock()` to the
collison branch in `down()`. We also want to spawn a new piece after locking
the previous one, but for now, we'll leave that out.

Okay, so `lock()` needs to do a couple of things. First, it needs to take all
blocks in the current pattern of the piece, and store them in the game board.
Then, it needs to check every row of the game board whether it is full, and if
so, move all rows that are higher up down (this effectively removes the row in
question). Finally, we need to redraw the board, and potentially show the user
their score. Let's see the pieces of the `lock()` code corresponding to each of
these.

```javascript
var lines = 0;
Piece.prototype.lock = function() {
	for (var ix = 0; ix < this.pattern.length; ix++) {
		for (var iy = 0; iy < this.pattern.length; iy++) {
			if (!this.pattern[ix][iy]) {
				continue;
			}

			if (this.y + iy < 0) {
				// Game ends!
				alert("You're done!");
				done = true;
				return;
			}
			board[this.y + iy][this.x + ix] = true;
		}
	}
```

Function declaration, loops, check whether the block in the pattern is set.
Those are all well-known by now. The next check might look weird, but it is
actually pretty straight forward. If a piece locks in place, but any one of its
blocks are above the top of the game board, then the player has failed, and the
game is over. Remember from before that all we had to do in this case to get
the game to halt was to set `done = true`, and so we do that. We also call the
browser-provided function `alert`, which displays a message to the user in a
pop-up, and lets them click OK to dismiss it. Finally, we set the block slot in
the board corresponding to where the piece's block is, to true, to mark it as
occupied.

Now, we need to eliminate full rows:

```javascript
	var nlines = 0;
	for (var y = 0; y < height; y++) {
		var line = true;
		for (var x = 0; x < width; x++) {
			line = line && !board[y][x];
		}
		if (line) {
			for (var y2 = y; y2 > 1; y2--) {
				for (var x = 0; x < width; x++) {
					board[y2][x] = board[y2-1][x];
				}
			}
			for (var x = 0; x < width; x++) {
				board[0][x] = false;
			}
			nlines++;
		}
	}
```

There's some nasty-looking code here, so let's step through it one piece at the
time. First, notice that the outer loop is simply looping over all the rows on
the board. We need to check them all, so no magic there. For each row, we
define a Boolean variable, `line`, which should only be set to true if every
tile in a row is occupied. We do this by initializing it to true, and then
doing a logical AND (only true when both the left and right expression are
true) with the values of every tile. If any tile is not occupied, it will be
set to false, and the AND will become false, making `line` false.

If the row was indeed fully occupied, we need to move down all rows above it.
The two nested for loops after the `if` do this for all but the topmost row (it
is handled specially below, because it has no row above it). To "move" a row
down, we loop over all the tiles of the row that is going to be overwritten,
and set each tile's value to be the value of the corresponding tile in the row
above. `board[y2][x] = board[y2-1][x]` accomplishes this.

Finally, we increment a counter to keep track of the number of lines that were
eliminated by this one piece locking in place. This will be used to update the
player's score, using the following code:

```javascript
	if (nlines > 0) {
		lines += nlines;
		drawBoard();
		console.log(lines);
	}
};
```

If any rows were eliminated, we add to the total score, redraw the board (since
some rows have changed), and finally we output the user's score. `console.log`
will print whatever is passed to it to the browser's [developer
console](https://developer.chrome.com/devtools/docs/console). The final version
does something marginally more fancy by showing the score on the page, but this
is a minor detail.

## Spawning new pieces

One final piece stands between us and a working Tetris game: spawning new
pieces. Everything else is in place. There are two places where we need to
create a new piece: when the game starts, and when a piece is locked in place.
So, we're going to define a new function, `newPiece`, that we'll call there,
that will return a random new piece. We can then assign that to the variable
`piece` that we use everywhere to refer to the currently falling piece. First
though, we need to tell our program about the different pieces that are
available. This is pretty straightforward:

```javascript
var pieces = [
	[I, "cyan"],
	[J, "blue"],
	[L, "orange"],
	[O, "yellow"],
	[S, "green"],
	[T, "purple"],
	[Z, "red"]
];
```

We now have a list of pattern-color pairs that can be used to construct new
`Piece` objects. Our `newPiece` function will simply pick one of these pairs at
random, construct a new object, and then return it.

```javascript
function newPiece() {
	var p = pieces[parseInt(Math.random() * pieces.length, 10)];
	return new Piece(p[0], p[1]);
}
```

`Math.random()` is a JavaScript function that returns a random number between 0
and 1. By multiplying that with the number of pieces we have, we get a random
index into the list that we can use to construct a random piece. However, this
number is a real number (it could, for example, be 3.14), and so it might not
be a valid list index. We use the `parseInt` function (the `10` argument is to
say that we want a base-10 number) to cut off the decimal values and get a
whole number. After picking this pair, we construct a new object using the
pattern and color from the pair, and return this!

## Almost done

The game is now in working condition, and fully playable. There are some odd
quirks though, that we will address in this [bonus section](improvements.md).
