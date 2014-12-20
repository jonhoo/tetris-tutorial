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

**Work in progress.**
