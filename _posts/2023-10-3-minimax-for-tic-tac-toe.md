---
layout: post
title:  "Mastering Tic-Tac-Toe with the Minimax Algorithm"
date:   2023-10-3
categories: jekyll update
permalink: :title
---

<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/latest.js?config=TeX-MML-AM_CHTML' async></script>



## Understanding Tic-Tac-Toe

Before we dive into the Minimax algorithm, let's review the rules of Tic-Tac-Toe:

- The game is played on a 3x3 grid.
- Two players take turns, one using "X" and the other using "O."
- The objective is to get three of your symbols in a row, either horizontally, vertically, or diagonally.
- If the entire grid is filled without a winner, the game ends in a draw.

Tic-Tac-Toe has a relatively small game tree, making it an ideal candidate for demonstrating the Minimax algorithm. Despite its apparent simplicity, Tic-Tac-Toe offers an opportunity to delve into the world of artificial intelligence and algorithms. We will explore the Minimax algorithm and how it can be used to create an unbeatable Tic-Tac-Toe AI.

## The Minimax Algorithm

The Minimax algorithm is a decision-making technique commonly used in two-player games with perfect information, such as Tic-Tac-Toe. It is designed to determine the best move for a player by considering all possible moves and their outcomes.

The algorithm operates on the assumption that both players are playing optimally, i.e., trying to maximize their own chances of winning while minimizing their opponent's chances. The name "Minimax" comes from the two key components of the algorithm:

1. **Minimize**: When it's the opponent's turn, the algorithm aims to minimize their potential gains. It assumes the opponent will make the best move for them.

2. **Maximize**: When it's the AI's turn, the algorithm aims to maximize its potential gains. It selects the move that leads to the highest possible outcome.


Let's attempt to mathematically model our problem. We denote by $$t$$ the number of actions played since the beginning of the game (so $$t\le 9$$), and we denote $$S_t$$ as the state of the game at time $$t$$. We can summarize the state of the game as a sequence of actions:

$$ S_t = [a_1, \dots a_t]$$

The objective is to find the best action $$a_{t+1}$$ to play and transition the game into the state $$S_{t+1} = S_t + [a_{t+1}]$$. For this, we will need to construct a heuristic that assigns a score to possible actions. We will simply choose the action that produces the highest score.

Thus, denoting $$h$$ as the heuristic, we have

$$ a_{t+1} = \underset{a}{\mathrm{argmax}} \: h(S_t+[a],\,0,\,\textbf{min})$$


## The Heuristic

The heuristic is written as $$ h(S,d,\textbf{op}) $$ where $$S$$ is the evaluated state of the game, $$d$$ is the depth of recursion, and $$\textbf{op}$$ is the next score selection policy ($$\textbf{min}$$ or $$\textbf{max}$$).


$$
h(S,\,d,\,\textbf{min}) = \left\{ \begin{array}{cl}
-10 + d & : \text{if} \ S\ \text{is a loosing state} \\
10 - d & : \text{if} \ S\ \text{is a winning state} \\
0 & : \text{if} \ S\ \text{is a draw state} \\
\underset{a}{\textbf{min}}\ h(S+[a],\, d+1,\, \textbf{max}) & : \text{otherwise}
\end{array} \right.
$$

The choice of the heuristic is what constitutes the intelligence behind the minimax algorithm. We assign a score of $$0$$ when the game results in a draw. A negative score when it's a loss with a penalty that increases as the loss becomes imminent. And a positive score when it's a win, with a higher score as the win becomes imminent.

The imminence is determined by the term $$d$$, which represents the number of recursions in the calculation of the heuristic.


And that's all there is to it! The Minimax algorithm is a powerful technique for creating intelligent game-playing agents. By systematically exploring all possible moves and evaluating their outcomes, the AI can make decisions that are difficult to beat. And for the Tic-Tac-Toe, the IA is even unbeatable.



## Application to the Tic-Tac-Toe game


To use the minimax algorithm for tic-tac-toe, you need to define the game state space. You can choose a 3x3 character array where:
- ' ' represents an empty cell,
- 'X' represents a move made by the computer (AI),
- 'O' represents a move made by the player.

First, you'll want to write an evaluation function that assesses the state of the game.

{% highlight C %}

// 2 -> no winner yet
// 1 -> computer won (X)
// -1 -> player won (O)
// 0 -> it's a draw
int evaluateState(char grid[3][3]) {

    // check rows
    for (int i = 0; i < 3; i++) {
        if (grid[i][0] != ' ' && grid[i][1] == grid[i][0] && grid[i][2] == grid[i][0])
            return (grid[i][0] == 'X') ? 1 : -1;
    }

    // check columns
    for (int i = 0; i < 3; i++) {
        if (grid[0][i] != ' ' && grid[1][i] == grid[0][i] && grid[2][i] == grid[0][i])
            return (grid[0][i] == 'X') ? 1 : -1;
    }

    // check diagonals
    if (grid[0][0] != ' ' && grid[1][1] == grid[0][0] && grid[2][2] == grid[0][0])
        return (grid[0][0] == 'X') ? 1 : -1;
    if (grid[0][2] != ' ' && grid[1][1] == grid[0][2] && grid[2][0] == grid[0][2])
        return (grid[0][2] == 'X') ? 1 : -1;
    
    // check draw
    int isDraw = 1;
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (grid[i][j] == ' ') return 0;
        }
    }

    // no winner
    return 2;
}

{% endhighlight %}


Then you need to implement the heuristic function


{% highlight C %}

int minimax(char grid[3][3], int depth, char player) {
    int score = checkWin(grid);

    if (score != 2) {
        // the game is finished, the score must be -1, 1 or 0 here
        return score*10 - score*depth;
    }

    int bestScore;

    if (player == 'X') {
        // X is doing the action
        // we want to maximize the score
        bestScore = -20;

        for (int i=0; i<3; i++) {
            for (int j=0; j<3; j++) {
                if (grid[i][j] != ' ')
                    continue;
                grid[i][j] = 'X';
                score = minimax(grid, depth + 1, 'O');
                grid[i][j] = ' ';
                if (score > bestScore)
                    bestScore = score;
            }
        }
    } else { 
        // O is doing the action
        // we want to minimize the score
        bestScore = 20;

        for (int i=0; i<3; i++) {
            for (int j=0; j<3; j++) {
                if (grid[i][j] != ' ')
                    continue;
                grid[i][j] = 'O';
                score = minimax(grid, depth + 1, 'X');
                grid[i][j] = ' ';
                if (score < bestScore)
                    bestScore = score;
            }
        }
    }
    return bestScore;
}
{% endhighlight %}


And finally the function that use the heuristic to do the action


{% highlight C %}

void computerMove(char board[3][3]) {
    int score;
    int bestScore = -20;
    int bestRow;
    int bestCol;

    for (int i=0; i<3; i++) {
        for (int j=0; j<3; j++) {
            if (board[i][j] == ' ') {
                board[i][j] = 'X';
                score = minimax(board, 0, 'O');
                if (score > bestScore) {
                    bestRow = i;
                    bestCol = j;
                    bestScore = score;
                }
                board[i][j] = ' '; // reset the board for next score computation
            }
        }
    }

    // do the best action
    board[bestRow][bestCol] = 'X';
}

{% endhighlight %}

