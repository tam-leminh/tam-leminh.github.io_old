---
bg: "draughts.jpg"
layout: post
title:  "Draughts AI"
crawlertitle: "Draughts AI - Tâm Le Minh"
summary: "Game of Draughts with a basic AI bot"
date:   2016-06-28 23:09:47 +0700
categories: posts
author: Tâm Le Minh
type: "project"
---

Strategic games have been a subject of AI research since decades. This is a fascinating domain, not only because 
people like playing games, but their respective ecosystems allow the algorithm to be easily benchmarked, by simply 
matching them against players of various level. Thus, methods for solving different games such as Chess, Checkers or 
Poker have considerably developed. In 1997, the chess reigning World Champion Garry Kasparov is world ranked No. 1 
since 1984. In what is considered a milestone in the history of AI, the supercomputer Deep Blue just beat him in a 
regular 6-game match. Though the match was close, this was the first time AI topped the world champion in Chess, a 
game considered to be unsolvable.

Indeed, different algorithms and techniques exist for different games. In Tic-tac-toe, all the possibilities can be 
easily explored by sheer force: we can simply compute all the possible configurations and sequences and order them in 
a tree. Then it can be showed that an optimal player 2 can force the draw, whatever player 1 does. Therefore any game 
between two perfect players (playing with an optimal strategy) would result in a draw. In Chess, from the initial 
position, 10^46 different configurations are reachable (Shannon). Taking into account the possible sequences to arrive 
to these positions, Allis estimated the game-tree complexity to be around 10^123. For such an amount of possibities, 
computation by sheer force is not possible. 

Both Tic-tac-toe and Chess are games with complete information. Both players have the same full information on the state 
of the game. This is different from Poker or Bridge, where players don't know the order of the cards in the deck or they 
don't know the cards in their opponents hands. Luck is also involved, so a player could play perfectly and still lose. 
The usual approach is similar, but the algorithms also take probabilities and likelihood calculations into account.

Also, DeepMind's AlphaGo became very famous after beating increasingly strong Go players and finally the World Champion 
Ke Jie in 2017. AlphaGo's approach is different from classic techniques. It incorporates techniques of deep learning and 
reinforcement learning that we will not discuss in this post. Yet, AlphaGo and its successor AlphaZero proved to be 
extremely performant, having also bested Stockfish 8, one of the top Chess bots, in 2017.

For Checkers, there are different variants of the game. In American Checkers (also called English Draughts) variant has 
been weakly solved by Schaeffer with the bot Chinook in 2007. Although there are 10^20 possible configurations in total, 
Schaeffer proved that any opening would result in an endgame configuration involving less than 10 pieces. Then he explored 
all these possible endgame configurations. After 18 years of computation, the Chinook finally learned the 10^14 endgame 
possibilities and how to solve them.

Now if we consider the International variant of Checkers, the rules are slightly different than the American version:
- the board size is 10x10 with 40 starting pieces, instead of 8x8 and 24 in the American version,
- the men (uncrowned pieces) can capture backwards,
- the kings (crowned pieces) can "fly", i.e. they have unlimited range over unoccupied squares,
- a move must capture the maximum number of pieces.
Not only there are even more possibilities, but also the gameplay is different from the American Version. Kings are 
significantly stronger pieces, so promoting men becomes more important. With the capture constraint, one can set traps by 
"offering" a piece to the opponent. The opponent is forced to capture the piece, even if it results in an eventually worse 
position for him. 

I designed and developed this International Checkers game with Matthieu Drouard, another ISAE-SUPAERO student. It was part 
of our 1st-year research project, supervised by J.-M. Alliot. The application is programmed in C. There are three main 
components:
- the move generator, i.e. the engine of the game, including the rules. For each position, it computes the possible moves. 
For each moves, it computes the next position,
- the GUI, made with SDL,
- the AI algorithm, which evaluates the best move for a player.
The user can choose to play without AI (e.g. against another player), to play against an AI bot or to make two AI bots play 
against each other.
