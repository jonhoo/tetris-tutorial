The game as discussed so far has a number of limitations compared to what a
"real" Tetris clone should have. In particular, the following three things need
to be fixed:

  - When a piece is locked in place, its blocks should retain their color.
  - If a piece is up against a wall, and a rotation is considered invalid only
    because it would place a block ourside the edges of the board, the block
    should be "kicked" to the side to make room for it. This is referred to as
    [wall kicking](https://developer.chrome.com/devtools/docs/console) in
    Tetris.
  - It's really inconvenient for the user that their score is displayed in the
    developer console. Instead, we want the score to be shown directly on the
    page holding the game.

This section will give solutions for these problems, suggest some other
improvements you may want to solve yourself to hone your skills, and
congratulate you on completing your own Tetris game.

## Board colors

Making the board retain colors when pieces are locked in place is actually
surprisingly easy. What we do, is that we change the values of our `board`
matrix to be strings rather than Booleans. If a tile contains the empty string,
`== ""`, then we consider that the same as if it were set to false, and if it
is set to anything else (`!= ""`), we consider it the same as if it were set to
true. Now, when we lock a piece (in `Piece.prototype.lock`), instead of doing

```javascript
board[this.y + iy][this.x + ix] = true;
```

We do

```javascript
board[this.y + iy][this.x + ix] = this.color;
```

And in `drawBoard()`, we replace

```javascript
ctx.fillStyle = board[y][x] ? 'red' : 'white';
```

Which just uses red or white depending on whether a tile is occupied, with 

```javascript
ctx.fillStyle = board[y][x] || "white";
```

Which will use the color set for a tile in `board`, or, if that is set to a
"falsy" value like the empty string, the OR will kick in, and the drawing color
will be set to "white". That's it! Pretty neat, huh?

## Wall kicking

Wall kicking is also fairly straightforward. Remember, our current rotate
function looks like this:

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
```

To add wall kicking, all we need to do is try to "nudge" the piece one block
to the left or right if rotating it in place causes a collision. If nudging
causes it not to collide, then we both move *and* rotate the piece. It is
probably easier to illustrate with code:

```javascript
Piece.prototype.rotate = function() {
	var nextpat = this.patterns[(this.patterni + 1) % this.patterns.length];
	var nudge = 0;

	if (this._collides(0, 0, nextpat)) {
		// Check kickback
		nudge = this.x > width / 2 ? -1 : 1;
	}

	if (!this._collides(nudge, 0, nextpat)) {
		this.undraw();
		this.x += nudge;
		this.patterni = (this.patterni + 1) % this.patterns.length;
		this.pattern = this.patterns[this.patterni];
		this.draw();
	}
};
```

Notice how much of the code remains quite similar. What changed is that we now
do **two** collision tests, one without the nudge, and one with the nudge. The
way `nudge` is set may seem kind of strange, but all it does is to nudge by
`-1` if the piece is to the right of the center (we're kicking away from the
right wall), and by `+1` if the piece is to the left of the center (kicking
from the left).

Trying this feature out is pretty cool too. With the old version in place, if
you put, for example, a T block with its long side against a wall, and then try
to rotate it, nothing will happen. With the kickback code in place, it will
simply shift over by one block, and rotate correctly, exactly as in regular
Tetris!

## Score counter

If you remember back to the previous section when we build
`Piece.prototype.lock` which locks a piece into place, you may recall that in
it, we placed a line saying

```javascript
console.log(lines);
```

Which prints the player's new score to the developer console. We now want to
replace that with something that shows the score somewhere the player will
actually see it. Above the game board seems like as good a place as any.

To accomplish this, we're going to be using some more web tech. We're going to
create a new HTML *element* (our canvas is also an element) that simply holds
text, and that we will use to put our score inside. We do so by modifying
[index.html](../index.html), and adding the following element *tag* above the
game's canvas tag:

```html
<span id="lines">Lines: 0</span>
```

We can now get a handle to this element similar to how we found our canvas
element to retrieve its drawing context; by using `document.getElementById`.

```javascript
var linecount = document.getElementById('lines');
```

We've already seen one method that is available on some HTML elements,
`getContext`, which we used to get a drawing context for our canvas. There are
**a lot** of such methods and variables, but the one of particular interest to
us right now is `.textContent` which lets us access, and modify, the text
contained within an element. Perfect! So, we put the above line somewhere at
the top of tetris.js, and then in our `lock` function, we replace the
`console.log` line with

```javascript
linecount.textContent = "Lines: " + lines;
```

Sweet, we have a live score tracker!

## Congratulations

Well how about that! You've just completed your first computer game, and have
hopefully learnt a bunch of stuff in the process. We have touched on a lot of
very complex topics, and have skipped a significant amount of the underlying
theory and concepts for the purposes of clarity, but it's a start.

If you enjoyed this, your next step should be to dive deeper into some of the
concepts introduced in this tutorial. Whether it's picking up a new language,
write another game, or extending this one, doesn't really matter. Just keep
programming, and look up things you do not know or understand, and you will
learn very quickly. Remember that searching is one of a programmer's best
allies. Websites like [Stack Overflow](http://stackoverflow.com/) contain vast
amounts of Q&A that will likely be relevant to any problem you encounter on
your journey as an aspiring programmer.

Good luck, and have fun!

## Further improvements

**Work in progress.**
