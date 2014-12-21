Tetris would not be a particularly interessting game if all blocks could simply
pass through each others, or if they were not stopped by the walls or the
floor. For our game to have some substance, we will need to detect these kinds
of things; a technique often called *collision detection*. We have a couple of
different collisions we need to consider:

  - The falling piece hitting the floor.
  - The falling piece being moved left or right into a wall.
  - The falling piece hitting a block on the board.
  - The falling piece being rotated, and the new rotation hitting a wall or
    block.

The fact that all of these scenarios are related to the falling piece *doing*
something is a big hint as to how we might implement these features. Now that
we have become familiar with functions, and, in particular, functions attached
to variables, the fact that the piece *does* something points to that we might
want to make collision detection a function that can be peformed *on* the
piece.

So, let's assume for a second that we have a function on the piece that we can
call, and that returns true if the piece collides with something given some
change in the piece's position or rotation. What kinds of arguments would it
need to take, and how would we use it? Well, by consulting the list above, we
see that there are three changes to a piece that we might want to check for a
collision for: x-movement (left/right), y-movement (down), and a changed
pattern (rotation). So, let's make those the arguments: `dx` (for delta x; a
common mathematical notation for the change in some variable), `dy`, and `pat`
(the new pattern to use for the piece).

Before we go into the implementation of `piece.collides(dx, dy, pat)`, let us
see how we would use it. We have to modify all of our different piece change
operations so that they check for collisions before applying any movement:

```javascript
Piece.prototype.rotate = function() {
	var nextpat = this.patterns[(this.patterni + 1) % this.patterns.length];
	if (!this._collides(0, 0, nextpat)) {
		this.undraw();
		this.patterni = (this.patterni + 1) % this.patterns.length;
		this.pattern = this.patterns[this.patterni];
		this.draw();
	}
};
Piece.prototype.down = function() {
	if (this._collides(0, 1, this.pattern)) {
		// Piece hits something and should be locked in place
		// A new piece should be spawned
	} else {
		this.undraw();
		this.y++;
		this.draw();
	}
};

Piece.prototype.moveRight = function() {
	if (!this._collides(1, 0, this.pattern)) {
		this.undraw();
		this.x++;
		this.draw();
	}
};

Piece.prototype.moveLeft = function() {
	if (!this._collides(-1, 0, this.pattern)) {
		this.undraw();
		this.x--;
		this.draw();
	}
};
```

Hey, that's actually not too bad! Only one added line per function is something
we can deal with. This really shows why functions are useful! You'll also note
that we now have a (currently empty) block of code to handle the case when a
block, while falling down, hits something, which should lead to it being placed
wherever it is, and a new piece to be spawned.

It's time to tackle the collision function itself. You'll note that it's called
`_collides`, not `collides`. This is simply a convention where functions that
are only used internally (i.e. they're only called on `this`, never on a
variable) are marked as such by prefixing their name with `_`. JavaScript
doesn't care though, so you can leave it off if you want to. Now, without
further ado, here is the collision function.

```javascript
Piece.prototype._collides = function(dx, dy, pat) {
	for (var ix = 0; ix < pat.length; ix++) {
		for (var iy = 0; iy < pat.length; iy++) {
			if (!pat[ix][iy]) {
				continue;
			}

			var x = this.x + ix + dx;
			var y = this.y + iy + dy;
			if (y >= height || x < 0 || x >= width) {
				return true;
			}
			if (y < 0) {
				// Ignore negative space rows
				continue;
			}
			if (board[y][x]) {
				return true;
			}
		}
	}

	return false;
};
```

By now, most of this should seem familiar. We go over all the blocks in the
piece's (potentially new) pattern, skip (`continue` skips one iteration of a
loop) the ones that aren't set. Next, we compute the absolute (that is,
relative to the game board), position of that block after taking `dx` and `dy`
into account, and we make sure that position is not outside the edges of the
board. We also check that there is no block on the game board that collides
with the block in the pattern. The check that `y < 0` is there simply because
trying to use a negative index for `board[y]` would crash our program. We
simply ignore any block collisions above the top of the game board, and call it
a day.

One minor point that I have not mentioned yet, but that is very useful when
writing large programs, is comments. In code, you can usually (some languages
do it differently) insert a comment into your code by prefixing it with `//`.
This indicates to the compiler that it should ignore whatever follows the `//`
when reading you program's code. Comments should generally be used to mention
things in the code that are not obvious why are there. They can also be used to
document what functions do, which parameters they take, what they return, etc.
by using [richer syntax](http://usejsdoc.org/about-getting-started.html) in the
comments. This is generally considered good practice, and for any non-trivial
project, it will make your life easier in the long run.

So, we're starting to get to the state where we have most of the pieces in
place for a working Tetris game (hehe). The missing parts now are user control,
spawning new pieces, row elimination, and some notion of time (pieces should
drop down every so often unless the user makes a move). These will be tackled
in the next section on [game progress](progress.md).
