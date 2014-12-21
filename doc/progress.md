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

**Work in progress.**
