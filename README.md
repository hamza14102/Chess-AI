# Chess AI

This folder has the following content:

-   `main.py`. This file plays the game (python3 main.py).
-   `submitted.py`: Implementations of the project.
-   `mp06_notebook.ipynb`: This is a <a href="https://anaconda.org/anaconda/jupyter">Jupyter</a> notebook to help you interact with the project. You can completely ignore it if you want, although you might find that it gives you useful instructions.
-   `chess/`, `res/`, `tools/`. These directories contain code and resources from PyChess that are necessary to run the assignment.
-   `requirements.txt`: This tells you which python packages you need to have installed, in order to run. You can install all of those packages by typing `pip install -r requirements.txt` or `pip3 install -r requirements.txt`.

This file (`mp06_notebook.ipynb`) will walk you through the whole MP, giving you instructions and debugging tips as you go.

### Table of Contents

1. <a href="#section1">Getting Started</a>
1. <a href="#section2">the PyChess API</a>
1. <a href="#section3">Assignment</a>
1. <a href="#section4">Extra Credit</a>
1. <a href="#section5">Submitted to Gradescope</a>

<a id='section1'></a>

## I. Getting Started

The `main.py` file will be the primary entry point for this project. Let’s start by running it as follows:

```python
!python3 main.py --help

```

This will list the available options. You will see that both player0 and player1 can be human, or one of four types of AI: random, minimax, alphabeta, or stochastic. The default is `--player0 human --player1 random`, because the random player is the only one already implemented. In order to play against an "AI" that makes moves at random, type

```python
!python3 main.py

```

<!-- #region -->

You should see a chess board pop up. When you click on any white piece (you may need to double-click), you should see bright neon green dots centered in all of the squares to which that piece can legally move, like this:

<p align="center">
    <img width="400" src="images/assignment5_human.png"/> 
</p>

When you click (or double-click) on one of those green dots, your piece will move there. Then the computer will move one of the black pieces, and it will be your turn again.

If you have trouble using the mouse to play, you can debug your code by watching the computer play against itself. For example,

```bash
python3 main.py --player0 random --player1 random
```

If you want to start from one of the stored game positions, you can load them as, for example:

```bash
python3 main.py --loadgame game1.txt
```

<!-- #endregion -->

<a id='section2'></a>

## II. the PyChess API

The chess-playing interface that we're using is based on [**PyChess**](https://github.com/pychess/pychess). All the components of PyChess that you need are included in the assignment5.zip file, but if you want to learn more about PyChess, you are welcome to download and install it. The standard distribution of PyChess includes a game-playing AI using the alphabeta search algorithm. You are welcome to read their implementation to get hints for how to write your own, but note that we have changed the function signature so that if you simply cut and paste their code into your own, it will not work.

You do not need to know how to play chess in order to do this assignment. You need to know that chess is a game between two players, one with white pieces, one with black pieces. White goes first. Players alternate making moves until white wins, black wins, or there is a tie. You don't need to know anything else about chess to do the assignment, though you may have more fun if you learn just a little (e.g., by playing against the computer).

Though you don't need to know anything about chess, you do need to understand a few key concepts, and a few key functions, from the PyChess API. The most important concepts are:

1. **player**. There are two players: Player 0, and Player 1.
   Player 0 plays white pieces, Player 1 black. Player 0 goes first. - **side**. PyChess keeps track of whose turn it is by using a boolean called **side**:
   `side==False` if Player 0 should play next.
2. **move**. A move is a 3-list: `move==[fro,to,promote]`.
   `fro` is a 2-list: `fro==[from_x,from_y]`, where `from_x` and `from_y` are each numbers between 1 and 8, specifying the starting x and y positions. `to` is also a 2-list: `to==[to_x,to_y]`.
   `promote` is either `None` or `"q"`, where `q` means that you are trying to promote your piece to a queen.
3. **board**. A board is a 2-tuple of lists of pieces: `board==([white_piece0, white_piece1, ...], [black_piece0, black_piece1, ...])`.
   Each piece is a 3-list: `piece=[x,y,type]`.
   `x` is the x position of the piece (left-to-right, 1 to 8).
   `y` is the y position of the piece (top-to-bottom, 1 to 8).
   `type` is a letter indicating the type of piece, which can be (`p`=pawn, `r`=rook, `n`=knight, `b`=bishop, `q`=queen, or `k`=king).

To get some better understanding, let's look at the way the board is initialized at the start of the game. The function `chess.lib.convertMoves` starts with an initialized board, then runs forward through a series of specified moves, and gives us the resulting board. If the series of specified moves is the empty string, then `chess.lib.convertMoves` gives us the opening board position:

```python
import chess.lib.utils, pprint

side, board, flags = chess.lib.convertMoves("")

print("Should we start with the White player?", side)

print("")

print("The starting board position is:")

pprint.PrettyPrinter(compact=True,width=40).pprint(board)

```

### II.A evaluate

The function `value=evaluate(board)` returns the heuristic value of the board for the white player (thus, in the textbook's terminology, the white player is Max, the black player is Min).

For example, you can find the numerical value of a board by typing:

```python
import chess.lib.heuristics

# Try the default board
value = chess.lib.heuristics.evaluate(board)
print("The value of the default board is",value)

# Try a board where Black is missing a rook
board2 = [ board[0], board[1][:-1] ]
value = chess.lib.heuristics.evaluate(board2)
print("If we eliminate one of black's rooks, the value is ",value)

# Try a board where White is missing a rook
board3 = [ board[0][:-1], board[1] ]
value = chess.lib.heuristics.evaluate(board3)
print("If we eliminate one of white's rooks, the value is ",value)

# Eliminate one piece from each player
board4 = [ board[0][:-1], board[1][:-1] ]
value = chess.lib.heuristics.evaluate(board4)
print("If the players are each missing a rook, the value is ",value)

```

### II.B encode and decode

Lists cannot be used as keys in a dict, therefore, in order to give your `moveTree` to the autograder, you will need some way to encode the moves. `encoded=encode(*move)` converts a `move` into a string representing its standard chess encoding. The **decode** function reverses the processing of `encode`. For example:

```python
from chess.lib.utils import encode, decode

# This statement evaluates to True
move1 = encode([7,2],[7,4],None)
print("The move [7,2]->[7,4] encodes as",move1)

# This statement also evaluates to True
move2=encode([5,7],[5,8],"q")
print("The move [5,7]->[5,8] with promotion to queen is encoded as",move2)

# This statement evaluates to True
move3 = decode("g7g5")
print("The move g7g5 is decoded to",move3)
```

### II.C generateMoves, convertMoves, makeMove

The function **generateMoves** is a [generator](https://docs.python.org/3/glossary.html#term-generator) that generates all moves that are legal on the current board. The function **convertMoves** generates a starting board. The function **makeMove** implements a move, and returns the resulting board (and side and flags). For example, the following code prints all of the moves that white can legally make, starting from the beginning board:

```python
import submitted, importlib
import chess.lib

# Create an initial board
side, board, flags = chess.lib.convertMoves("")

# Iterate over all moves that are legal from the current  board position.
# print(type(submitted.generateMoves(side, board, flags)))
for move in submitted.generateMoves(side,board,flags):
    newside, newboard, newflags = chess.lib.makeMove(side, board, move[0], move[1], flags, move[2])
    print("This move is legal now:",move, newflags)

```

The **flags** and **newflags** variables specify whether or not it has become legal for black to make certain specialized types of moves. For more information, see `chess/docs.txt`.

### II.D random

In order to help you understand the API, the file `submitted.py` contains a function from which you can copy any useful code. The function `moveList, moveTree, value  = random(side, board, flags, chooser)` takes the same input as the functions you will write, and generates the same type of output, but instead of choosing a smart move, it chooses a move at random.

Here, the input parameter `chooser` is set to `chooser=`<a href="https://docs.python.org/3/library/random.html#functions-for-sequences">random.choice</a> during normal game play, but during grading, it will be set to some other function that selects a move in a non-random fashion. Use this function as if it were equivalent to `random.choice`.

```python
import submitted, importlib
importlib.reload(submitted)
help(submitted.random)
```

<a id='section3'></a>

## III. Move Search

For this, we will use three functions: `minimax` and `alphabeta`. The content of these functions is described in the sections that follow.

### III.A minimax search

```python
importlib.reload(submitted)
help(submitted.minimax)
```

As you can see, the function accepts `side`, `board`, and `flags` variables, and a non-negative integer, `depth`. It should perform minimax search over all possible move sequences of length `depth`, and return the complete tree of evaluated moves as `moveTree`. If `side==True`, you should choose a path through this tree that minimizes the heuristic value of the final board, knowing that your opponent will be trying to maximize value; conversely if `side==False`. Return the resulting optimal list of moves (including moves by both white and black) as `moveList`, and the numerical value of the final board as `value`.

A note about `depth`: The `depth` parameter specifies the total number of moves, including moves by both white and black. If `depth==1` and `side==False`, then you should just find one move, from the current board, that maximizes the value of the resulting board. If `depth==2` and `side==False`, then you should find a white move, and the immediate following black move. If `depth==3` and `side==False`, then you should find a white, black, white sequence of moves. For example, see [wikipedia's page on minimax](https://en.wikipedia.org/wiki/Minimax#Minimax_algorithm_with_alternate_moves) for examples and pseudo-code.

You can test it by playing against it. If you have pygame installed, the following line should pop up a board on which you can play against your own minimax player.

```python
!python3 main.py --player1 minimax
```

If you want to watch a minimax agent win against a random-move agent, you can type

```python
!python3 main.py --player0 minimax --player1 random

```

### III.B alphabeta search

```python
import submitted
help(submitted.alphabeta)
```

For any given input board, this function should return exactly the same value and moveList as `minimax`; the only difference between the two functions will be the returned `moveTree`. The tree returned by `alphabeta` should have fewer leaf nodes than the one returned by `minimax`, because alphabeta pruning should make it unnecessary to evaluate some of the leaf nodes.

You can test this using:

```python
!python3 main.py --player0 random --player1 alphabeta

```

<a id='section4'></a>
