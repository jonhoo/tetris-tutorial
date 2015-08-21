The game as discussed so far has a number of limitations compared to what a
"real" Tetris clone should have. In particular, the following three things need
to be fixed:

  - When a piece is locked in place, its blocks should retain their color.
  - If a piece is up against a wall, and a rotation is considered invalid only
    because it would place a block ourside the edges of the board, the block
    should be "kicked" to the side to make room for it. This is referred to as
    [wall kicking](http://tetris.wikia.com/wiki/Wall_kick) in Tetris.
  - It's really inconvenient for the user that their score is displayed in the
    developer console. Instead, we want the score to be shown directly on the
    page holding the game.
  - Using `keypress` to handle key input has a couple of downsides, namely that
    it doesn't work for arrow keys in Chrome, and the speed of repetition when
    a key is held down can't be configured. We should fix that.

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

## Key bonanza

If you remember back to the progress section, you'll recognize the following
code for handling key presses.

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

This code has one major issue: `keypress` doesn't work in the Chrome browser
for non-printable characters such as arrow keys. For more info of the madness,
have a look at [this page](http://unixpapa.com/js/key.html). Instead, we need
to use the `keydown` event, but this comes with its own problem. If a user
holds a key down, the `keypress` event will be triggered multiple times, just
like holding a letter down in a text editor will repeat the character multiple
times. Unfortunately, this does not happen for `keydown`, because technically,
the key is only physically pushed down once.

The way to overcome this is to start a timer on `keydown`, and keep handling
the key that was pressed each time the timer fires. When we get a `keyup` event
for the corresponding key, we cancel the timer, and thus stop handling the key.

Let's see the code to do this first:

```javascript
var downI = {};
document.body.addEventListener("keydown", function (e) {
	if (downI[e.keyCode] !== null) {
		clearInterval(downI[e.keyCode]);
	}
	key(e.keyCode);
	downI[e.keyCode] = setInterval(key.bind(this, e.keyCode), 200);
}, false);
document.body.addEventListener("keyup", function (e) {
	if (downI[e.keyCode] !== null) {
		clearInterval(downI[e.keyCode]);
	}
	downI[e.keyCode] = null;
}, false);

function key(k) {
	if (k == 38) { // Player pressed up
	// ... (the rest of the ifs from the original code)
}
```

There are a couple of new concepts being introduced here. In particular, we
have `{}`, `setInterval`/`clearInterval`, and `bind`. We'll go through them one
by one.

`{}` in JavaScript denotes an empty *dictionary* or *map*. Dictionaries are
available in many programming languages, and are very similar to lists in that
they hold many elements. The difference lies in how these elements are
accessed. In a list, the first element is element 0, the second is element 1,
and so on. In a dictionary, the indices can be anything; a string, a number, a
character, etc. In the code above, we use the dictionary to store a mapping
from each key that is being held down to its timer. We need to keep this
mapping so that we can cancel the timer when a key is released, as you can see
from the code.

`setInterval` and `clearInterval` are fairly self-explanatory. `setInterval` is
a function that takes two arguments, a function to call, and the number of
milliseconds between each invocation of that function. In the code above, we
set the interval to be 200ms, so if a key is held down, the corresponding
action will be performed five times a second. Try tweaking this and see what
happens when you keep a key pressed. To stop a timer created by `setInterval`,
you need to call `clearInterval`, and give it the identifier for the interval
you want to cancel. The identifier to use is the one that is returned by
`setInterval`, and this is why we need to store it in the `downI` map.

It is also worth noting that we are potentially keeping multiple interval
timers at the same time. This is intentional, as a player might press and hold
the down arrow, and then slightly later press and hold the left arrow. We want
each to only be triggered five times a second, and we don't want, say, the left
arrow action to stop just because the player releases the arrow down key. You
might find it useful to try implementing the above with only one timer, and see
what problems you run into.

`bind` is the final new concept introduced here, and it is a somewhat confusing
beast. To understand what it does, let us start with why it is useful for us in
this particular case. The function that is given to `setInterval` is called
*without any arguments*. This is not particularly useful for us, because we
need to be able to tell which key to act on in `function key`. We could store
what key is being pressed in some variable, but then we lose support for
pressing multiple keys at the same time! This is where `bind` comes in. `bind`
can be called on a function, and returns a new function that calls the first
one with some of the arguments already set to a predetermined value. For
example, imagine you have a function `hello` that takes a single argument
`name`, and suppose you want that to be called by `setInterval` with `name` set
to `John`. By passing `hello.bind(this, "John")` instead of just `hello` to
`setInterval`, this is exactly what will happen. The `name` argument will be
*bound* to `"John"`.

Now, you may wonder, what is this `this` thing that is given as the first
argument to `bind`? Well,
[this](http://javascriptissexy.com/understand-javascripts-this-with-clarity-and-master-it/)
is a topic that many find confusing, and covering it in any detail is beyond
the scope of this tutorial. To give you a very vague answer with respect to
`bind`, the first argument that is given to `bind` will be what the magical
`this` variable is set to inside the function `hello` when it is called. What
we're saying by passing `this` as the first argument is that `this` inside
`hello` will refer to the same `this` as where we're calling `setInterval`
from. Since we're not using `this` inside `hello` (or inside `key`), it
actually doesn't matter what we give as the first argument to `bind` here, so
we use `this` as a matter of convention if nothing else.

The rest of the code fragment above should be fairly straightforward. When a
key is pressed, we clear any previous timer for that key (there probably
shouldn't be any, since we clear on `keyup`) just in case, we handle the key
press once immediately (wouldn't want the first rotate to happen 200ms after
pressing the button!), and then start a timer, storing the timer name in our
map. When a key is released, we just stop the timer and clear its name from the
map. Easy peasy!

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
