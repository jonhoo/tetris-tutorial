The game as discussed so far has a number of limitations compared to what a
"real" Tetris clone should have. In particular, the following three things need
to be fixed:

  - When a piece is locked in place, its blocks should retain their color.
  - If a piece is up against a wall, and a rotation is considered invalid
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

**Work in progress.**
