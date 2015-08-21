# And so it begins.

## Boilerplate

The field of computer science is littered with technical jargon (similar to
most other fields), that can be quite inscrutable to newcomers. The term
"boilerplate" is one of the more familiar ones, and refers to "magical
incantations"; code that has to be written to get a basic application up and
running, and which doesn't really change from time to time. Many libraries
exist purely to reduce the amount of boilerplate needed for building
applications that follow common patterns (like single-page web apps).

In our case, we are using the browser to render our game, and so we need to get
it to draw a basic web page, and then give us a blank canvas in which to draw.
The magical incantation for this is:

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
	</head>
	<body>
		<canvas id="board">
		</canvas>
		<script src="tetris.js"></script>
	</body>
</html>
```

This tells the browser that we do not want backwards compatability (`<!DOCTYPE
html>`), that we want to use a character encoding (go Google it) called
"utf-8", that we want to have a blank canvas called "board", and that we want
it to execute the code that is contained in the file `tetris.js`. If you look
at the [final .html file](../index.html), you'll see that there's some extra
stuff in there. Most of it is code in a language called CSS, that styles the
various items on the web page, some of it is for showing the user's score, and
there's an additional `<script>` tag there to avoid having to put all the game
code in a single large file. None of these are necessary for the game to run
(assuming you concatenate all the game code into a single file).

## Storing the board

The primary item of interest in Tetris is the game board. The board consists of
200 tiles arranged in 20 rows of 10 columns. Our first decision when building
the game will be how we want to store the current state of the board, and how
we choose to do that will influence much of the remaining code, as many things
will have to interact with it.

### Keeping state

To store something temporary (the user is unlikely to run the game forever) in
a computer program, we usually put things in the computer's *memory*. The
memory stores sequences 0s and 1s, and lets developers access them by giving an
offset into the memory. The memory an application can see is, in most cases,
private to the application. In JavaScript, and many other interpreted
languages, the contents of memory are initialized to all 0s; other
languages, often lower-level ones like C, require memory to be
explicitly initialized before it can be used. Since the values in an
application's memory represent everything it knows, we sometimes refer
to it as the program's *state*. A program's state also includes certain
other things such as open files and network connections that the
application learns about while executing.

An offset into memory (a memory *address*) is usually given in *bytes* (eight
bits), as in "give me the 63rd byte" or "write this data to the 185th byte". A
byte is usually the smallest unit you can use when reading from or writing to
memory. If you want to change a single bit, you would read a byte, change the
bit in the byte, and then write the byte containing that bit back.

To avoid having to put lots of memory addresses into our programs, programming
languages use the concept of *variables*. A *variable* has different semantics
in different languages, but it is commonly just a reference to a location in
memory. For example, when you write `x = 0`, your computer will find an unused
part of memory, write `0` to it, and then, if you refer to `x` later in your
program, it will know that you really meant "the place where I stored 0
previously". Consider the following code:

```javascript
x = 0
y = x + 1
x = y
```

Here, the computer will first find room for `x`, and write 0 to it. Then, it
will find room for `y`, go read the value in the memory location it found for
`x`, add one to it, and write it to the location it found for `y`. Finally, it
will change the value in the memory location it found for `x` with the value in
the memory location for `y`. For convenience, we usually simply refer to
variables as "having" a value, even though technically they are only
human-readable names for memory addresses that then hold values.

### Types

Computers generally only know about bits; 0s and 1s. The notion of numbers,
letters, and lists are completely foreign to it. When you write a program in a
programming language, there will more often than not be another program that
takes your code as input, and then outputs the low-level machine instructions
operating on 0s and 1s that you can actually run. This program is usually
called the "compiler" or the "interpreter" depending on what language you are
using, and is what really does the variable to memory address conversion we saw
above. For simplicity, we will be referring to these interchangably in this
tutorial, although the two are actually different.

Part of the role of the compiler is to provide the *illusion* that the computer
knows about these things; it fakes them for you. This process of hiding "lower
level" details so that "higher level" code can ignore then is called
*abstraction*, and it is a very powerful concept in computing that we will see
several examples of while developing our Tetris clone.

The way the compiler hides the 0s and 1s from you is by providing a notion of
*types*. All values have a *type*. This type can be determined implicitly (e.g.
3.1 is obviously a real number), or it can be explicitly stated by the
programmer, usually by annotating the variable with a *type declaration*. How
this is done varies from language to language, and some languages do not
support explicit typing at all (JavaScript for example). Just so that you've
seen how type annotations look, here's how you would declare two variables with
the same logical value (the number 5), but with different *types*, in the C
langauge:

```c
int x = 5;
float y = 5.0;
```

The different types mean that the computer will be storing these values
differently in memory. `x` will simply be stored using the binary form of the
number 5, using four bytes, whereas `y` will be stored using the floating point
number format as defined in the IEEE 754 standard, but also in four bytes.
Because all variables have a single type in C (that cannot be changed), C is
considered a "statically typed" language. JavaScript on the other hand, because
it allows any variable to hold any value (and for the type of that value to
change), it is considered a "dynamically typed" language.

The reason types are important is that you might find that they sometimes
produce strange results. To show why, we first have to introduce you to a
couple of common types in programming.

First, most languages have a *character* type, which holds, well, a single
character. Seems simple enough at first, but exactly what a character **is** is
not always so easy to
[pin](http://www.joelonsoftware.com/articles/Unicode.html)
[down](http://blog.golang.org/strings#TOC_5.). Let's ignore the details for
now, and simply assume that such a type exists, and that it holds something
like an 'e'.

Next, we check out another primitive usually provided by most programming
languages: lists. A list is, innocently enough, an ordered sequence of things.
Lists are extremely flexible, and some programming languages such as LISP use
lists for _everything_. Some languages require all the elements of a list to be
of the same type (sometimes called *typed lists*), but JavaScript does not.

Now, let us consider if you wanted to store this sentence in a computer
program. We could introduce a "sentence" type, but most programming languages
instead make do by combining more primitive types. A sentence is really just a
*list* of *character*s, so why not store it as one? This abstraction is called
a *string*, and is used in pratically every program ever written. We can use
*string*s to store other text too, such as words, paragraphs, or even entire
documents. You can usually recognize a string by seeing text enclosed in double
quotes ("").

So, back to the problem I was telling you about types sometimes making weird
things happen. Consider the *string* "1234", and the *number* 1234. The former
is a list of four characters, '1', '2', '3', and '4'. The latter is the number
that follows 1233. Now, consider what happens if we do this:

```javascript
x = "1234"
y = 1234
z = x + y
```

What do you expect `z` to contain after those three lines have been run? More
to the point, what *type* would you expect it to have? If you can't say, how
can you expect a computer to? As it turns out, there are two common approaches
to handling situations like this. The first is for the compiler to simply crash
with a message saying "hey, those two things are different, I don't know what
to do", and then force you to fix your program. Programming langauges that tend
towards doing this are commonly referred to as "strongly typed". The other
approach is for the compiler to guess what the developer meant. Languages that
do this are considered "weakly typed".

How would the computer guess though? In JavaScript, the interpreter attempts to
make every value in an expression the same type (`x + y` is an expression) by
using *implicit type conversion*. In this case, it sees a string, and so it
will attempt to convert the number contained in `y` to a string (resulting in
"1234"). Thus, `z` will contain the *string* "12341234"!

### What about the board?!

Oh, yes, that's right. That's what we were doing. So, the board in Tetris
consists of 200 tiles, all of which are either occupied or not. We could use a
number for each tile, and set it to `1` if the tile is occupied by a piece, or
`0` if it is vacant, but that doesn't quite feel right. What if we
unintentionally set it to `2`? What would our application do then? Luckily, we
have another type available for precisely this scenario: *boolean* values.
"Booleans" can contain only one of two values: *true* or *false*, and are very
frequently used in computer programs both for readability and for semantic
correctness (i.e. it "feels right" to use a Boolean where you can).

Okay, so 200 Booleans it is. Well, we already know about this thing called
lists, so how about we make a list of length 200, and then make each of the 200
elements Booleans? We could do that, but it would be a pain to find, say, the
tile corresponding to the 3rd row, 5th column. If only there was a better way.

Turns out, there is. Who would have thought? See, lists can contain other
lists, making "multi-dimensional" lists. A list of lists is commonly referred
to as a 2D list. If all the nested lists are the same length, this is
essentially a matrix (as we all know and love from maths), whereas if they
aren't, it is usually referred to as a "jagged" 2D list. Let's instead make our
board be a list of 20 lists (one for each row), where each of the nested lists
contain 10 Booleans (one for each tile in that row). Initially, all the tiles
are empty, so let's make sure they're all set to false too. Here's how to do
this in JavaScript:

```javascript
var board = [];
for (var row = 0; row < 20; row++) {
	board[row] = [];
	for (var tile = 0; tile < 10; tile++) {
		board[row][tile] = false;
	}
}
```

Lot's of new stuff here, so let's go over it step by step.

## Baby steps

The code fragment above may look somewhat daunting, but you might be able to
see roughly what's going on by glancing at it for a while. When you've made
your guesses, continue reading.

First, let's get some of the syntax out of the way.

  - In JavaScript, lists are dynamically sized. This means that you do not
    specify their size before using them. `[]` is a value that signifies an
    empty list.
  - `var` is a keyword that indicates that what follows it is a new variable,
    and is not related to any other variable, even if it carries the same name.
    You can leave off `var`, but that means that if someone defines `board`,
    `row`, or `tile` further up in your code, you'll be overwriting their
    values!
  - `;` is the *statement termination character* in JavaScript. Putting in a
    `;` tells the interpreter that it can execute whatever code lead up to the
    `;`, and that whatever follows is a completely separate instruction. `;`s
    are usually optional in JavaScript, but leaving them out can lead to some
    strange and hard to diagnose bugs.
  - `++`, when used as a prefix or a suffix to a variable means "take this
    variable's value and increment it by one". It is called the *unary
    increment operator*, and is available in most langauges. There is also its
    sibling `--` which decrements a variable by one.
  - `somevariable[1]` refers to the 1st element of the list in `somevariable`.
    It is important to note that lists in **most** programming languages are
    *zero-indexed*, so the **first** element in a list of length `n` is `[0]`,
    and the **last** is `[n-1]`! If `somevariable[1]` is itself a list, the
    `[]` operator can be used again to get an element of that nested list as
    `somevariable[1][1]`.

Now, we are ready to consider what was probably the most confusing bit in the
code fragment above; the `for` lines.

### Loops

In applications, it is frequently necessary to do something multiple times.
Imagine going over a list of user and printing out the name of each one, or
count the number of friends in a user's friend list. This is accomplished by
using *loops*. A loop is a statement that *iterates* over a number of values,
executing a piece of code for each *iteration*.

The most common loop is the `for`-loop, and some langauges even [make do
without](https://golang.org/doc/effective_go.html#for) all other loop varaints.
A `for`-loop is started with the following incantation:

```javascript
for (var row = 0; row < 20; row++) {
```

Let's go over this piece by piece. First and foremost, notice that inside the
parentheses, there are two semicolons, separating the text into three
statements. This pattern is the most common style of for-loop, and the three
segments correspond to an *initialization statement*, a *loop condition*, and
an *progress statement*. The initialization statement says what should be done
right before the code inside the loop (what is contained inside `{` and `}`) is
first executed. The loop condition is checked before executing the code in each
iteration of the loop, and if the condition it contains is no longer true, the
code is not executed, and the loop ends. In the code above, the loop would
terminate once the `row` variable has a value that is greater than or equal to
20. Finally, the progress statement indicates what should be done **after**
each time the loop code is executed. In this case, we increment `row` so that
we will consider the next row in the next iteration.

If we "unroll" the original loop (that is, we enumerate all the operations that
are really performed), it would look like this:

```javascript
var board = [];
board[0] = [];
board[0][0] = false;
board[0][1] = false;
...
board[0][8] = false;
board[0][9] = false;
board[1] = [];
board[1][0] = false;
board[1][1] = false;
...
board[19][8] = false;
board[19][9] = false;
```

It should be fairly clear now that this will construct the full 20x10 board we
wanted to build.

Great! Now how do we draw the board on screen? Have a look at
[drawing](drawing.md).
