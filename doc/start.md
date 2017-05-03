# Metainformation

Today (and potentially over the course of many days), we'll be building the
game "Tetris". If you're not familiar with it, I suggest you go Google it now,
and play one of the many variants that exist in the wild. It's a fairly simple,
but great game!

## Programming languages

When building any kind of application, the first choice you need to make is
what *programming language* you want to use. Different programming languages
represent different ways of telling a computer what you want it to do.
Programming languages come in many forms, and they all have large or small
differences that make them unique, but some differences are more noteworthy
than others. I'll cover the common categories pretty quickly in this section.

A programming language generally favors a particular programming idiom; a style
of representing and building programs. Note that very few languages adhere
strictly to one idiom or another, and they often provide features from multiple
categories. The major groups here are:

  - Imperative languages: This is the largest class of languages. In these
    languages, you give the machine a sequence of actions that it should
    execute in order, much like a cooking recipe, and the computer blindly does
    so. Languages like C and JavaScript fall into this category.
  - Functional languages: In these languages, your programs resemble
    chained mathematical expressions. You start with some input, and you tell
    the machine that it should apply various transformations to it to produce a
    new value. This value can then be fed into other transformations, etc.
    Famous languages in this category are LISP and Haskell.
  - Object-oriented languages: These languages model everything as "objects";
    entities that have knowledge about themselves and their related objects,
    and are able to do computations based on that information. Java and Python
    are major players here.
  - Declarative languages: These are not programming languages per se, but
    instead, they *declare* various properties that are then used by some other
    application to make decisions. HTML, XML, and CSS are the most common
    examples of declarative languages.

There are major simplifications above, and most of these languages span
multiple groups (Java and Python are also imperative for example). There are
also a number of other categories such as prototyped languages (JavaScript is
also one of these), and logical languages (Prolog is an excellent example). If
you want to learn more about this, go ahead, but I'm not going to spend any
more time on it.

For this tutorial, we will be using JavaScript. JavaScript initially started
out as a language that was meant to run in the browser, and was used primarily
on websites to provide interactive functionality such as notifications, mouse
location detection, image animation, etc. Recently, developers have moved to
building entire applications using only JavaScript, making the server a dumb
machine that just stores data. Many now also build servers using JavaScript,
decoupling it entirely from the browser. I chose JavaScript because it is quite
easy to get started with (all you need is a browser), because the *syntax* (the
way you write code in the language) is fairly straightforward, and because it
is extremely widely used. You should go explore other languages once you've
finished this tutorial!

## Libraries and frameworks

Programming languages on their own only provide you with primitive building
blocks, and while you are theoretically able to do anything by building it from
scratch, it is often much more efficient (and enjoyable) to use components
other people have built where possible. Since we won't be doing anything
extremely complicated (the entire program is ~250 lines), we are not going to
need any external libraries (apart from the browser) for our game.

To be technically correct, we will be relying on the HTML "framework" in the
sense that the browser provides a lot of functionality to us by being able to
render things on screen very easily. We will also be using the "canvas" feature
supported in modern browsers, that allows us to draw arbitary things, not just
build website-like things.

Let's start digging into some code. Head over to [code](code.md) when you're
ready.
