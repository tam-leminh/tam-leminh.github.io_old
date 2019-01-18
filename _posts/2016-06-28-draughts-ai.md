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

## AI in board games

Indeed, different algorithms and techniques exist for different games. In Tic-tac-toe, all the possibilities can be 
easily explored by sheer force: we can simply compute all the possible configurations and sequences and order them in 
a tree. Then it can be showed that an optimal player 2 can force the draw, whatever player 1 does. Therefore any game 
between two perfect players (playing with an optimal strategy) would result in a draw. In Chess, from the initial 
position, $$10^{46}$$ different configurations are reachable (Shannon). Taking into account the possible sequences to arrive 
to these positions, Allis estimated the game-tree complexity to be around $$10^{123}$$. For such an amount of possibities, 
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
been weakly solved by Schaeffer with the bot Chinook in 2007. Although there are $$10^20$$ possible configurations in total, 
Schaeffer proved that any opening would result in an endgame configuration involving less than 10 pieces. Then he explored 
all these possible endgame configurations. After 18 years of computation, the Chinook finally learned the $$10^14$$ endgame 
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

## AI principles

At each turn, the algorithm should determine the best move to play for the selected player. It relies on a game 
tree where the nodes are game states $$s$$. In Checkers or Chess, a state can be represented by a configuration of pieces, 
but also the next player to play. At one particular state of the game, each possible $$a \in A_s$$ move leads to a new state 
$$s' \in S_s$$ (the subscripts denote that these are the set of actions and sets accessible from the state $$s$$), i.e. it's 
an edge leading to a new node. Let $$a(s,s')$$ be such a move. The root of the tree is the node representing the current 
state. Therefore, each layer in the tree represent one move. The root is the layer 0. For example, one can list all the nodes 
in the layer 2 to know all the possible configurations after 2 moves. Let $$S_i$$ be the set of all possible states after $$i$$ 
moves, or all the states of the $$i$$-th layer. Note that within one layer, there could be several nodes representing the same state 
as different Also, because the players alternate turns and there is only one move allowed per turn, each layer represent a turn. 
Thus, one layer could represent player 1's turn, then the next layer would represent player 2's, etc. A branch of the tree is 
ended only when the game ends, i.e. the leafs are checkmate or stalemate states. 

Let's consider that the algorithm plays as player 1. Then, each decision/move should be associated with a value $$Q(a)$$, measure of its 
quality. Now, Checkers are a zero-sum game. That means for one player, a good situation for is equally good for them than bad 
for their opponent. For player 1, for one particular move $$a_t$$, the value $$Q_1(a_t)$$ gets as high as their opponent's $$Q_2(a_t)$$ 
gets low. Each player looks to maximize its own values $$Q$$: I hope to play the best move and I hope my opponent plays as poorly as 
possible. This is equivalent to saying that player 1 wants to maximize $$Q$$ whereas player 2 wants to minimize it. The minimax 
algorithm is based on this idea. Now for simplicity, instead of reasoning in actions and states, we can take advantage of the tree and 
use nodes and layers. Each node $$X$$ corresponds to a state $$s$$. It has $$n_X$$ children nodes $$X_i \in (X_i)_{i \in [1,n_X]}$$, each 
associated to a state $$s_i$$ by an action $$a(s,s_i) \in A_{s}$$. Let $$V_l$$ be a score value for the nodes of the $$l$$-th 
layer, such as if $$X_i$$ is in the $$l$$-th layer, $$V_l(X_i) = Q(a(s,s_i))$$.

How can we calculate $$V_l$$? 

### Minimax algorithm

First, let's suppose that we can evaluate the quality of a state $$s$$. This is modelled with an evaluation function $$f$$. A higher 
$$f(s)$$ means the more favourable to us. Inversely, it is lower when the state is worse. Because each node $$X$$ is associated with a 
state $$s$$, we can write $$f(X) = f(s)$$.

At the root node $$X_{root}$$, player 1 plays. They can select the move that leads to the node $$X_{max}$$ where the value of $$f$$ 
is the highest, i.e. $$X_{max} = \arg \max_{i \in [1,n_{Xroot}]}$$ $$ V_1(X_i) = f(X_i)$$. However this is a short-sighted strategy. 
Indeed, in this case, only the next move is evaluated. Maybe this can allow player 2 to react with an even better move, that 
will eventually put player 1 in an awful situation. Perhaps there are poor moves in the short-term, sacrificing pieces for example, 
but would result in a better position several turns ahead. For an analogy in Chess, a good move when considering only one turn 
ahead but bad when considering 2 turns, could be to capture a defended pawn with a rook, because then the rook would be captured 
itself in the next turn. So a following this approach, we should also check the opponent's possibilities. Hence, we cannot just 
straightforwardly use the function f. We would rather define 
$$X_{max}$$ such as $$X_{max} = \arg \max_{i \in [1,n_{Xroot}]} V_1(X_i)$$ where $$V_1$$ is now 
$$V_1(X) = \min_{i \in [1,n_X]} V_2(X_i) = f(X_i)$$. That means, we consider the strongest reaction the opponent can throw after 
each of our possible move, and we choose the move where this optimal reaction is the weakest. In other words, we are preventing 
them as much as possible to play the best moves.

Layer after layer, we can continue to look forward and plan more moves. In an opening or midgame scenario, it's impossible to 
to build the complete tree due to the overwhelming large number of possibilities. Therefore, we must define a tree depth corresponding 
to the number of moves we want to plan. An larger tree depth means better moves, but the computational effort required also increases 
exponentially. The previous problem can be generalized for $$L$$ layers:
Define $$V_L(X) = f(X)$$
For $$l \in [1,L-1]$$:
	If $$l$$ is even:$$
		\begin{equation}
			V_l(X) = 
			\begin{cases}
				- \infty, & \text{if} X has no children \\
				\max{i \in [1,n_X]} V_{l+1}(X_i), & \text{if} X has children 
			\end{cases}
		\end{equation}$$
	if $$l$$ is odd:$$
		\begin{equation}
			V_l(X) = 
			\begin{cases}
				+ \infty, & \text{if} X has no children \\
				\min{i \in [1,n_X]} V_{l+1}(X_i), & \text{if} X has children 
			\end{cases}
		\end{equation}$$
	
Pick $$X_{max} = \arg \max_{i \in [1,n_{Xroot}]} V_1(X_i)$$

We observe that there are two kinds of layers. The ones where the player is playing (layers indexed by even numbers), so the 
algorithm tries to maximize $$V$$, and the ones where the opponent is playing (layers indexed by odd numbers), so the algorithm 
assume they want to minimize $$V$$. So the levels alternate between maximizing and minimizing steps.

Also, before, we assumed that all the branches can be developed beyond the $$L$$-th layer. In practice, this is often true, 
especially in start or mid game situations. However, in the case where there are winning or losing positions, the tree does not 
develop further in the direction of the corresponding nodes. We can assume that when it's the player's turn and they have 
possibilities, that means they have lost. So we can affect a $$- \infty$$ value to this node. If this is the opponent who cannot 
play, the player has won. So we can affect $$+ \infty$$.

### Minimax implementation

For a fixed depth L, one way to solve the minimax problem is to evaluate all the nodes of the $$L$$-th layer, so calculate 
$$V_L(X) = f(X)$$. Next, these values can be propagated upwards to $$V_{L-1}$$, $$V_{L-2}$$, etc. maximizing or minimizing 
the relevant values, until reaching $$V_1$$. However, in practice, this method is not efficient. The tree must be entirely 
built and kept in the memory before starting to calculate the $$V$$ values. 

Instead, a recursive function can be used, taking advantage the minimax algorithm. In this case, it computes and propagates 
the $$V$$ scores while exploring the tree.

function Minimax($$X$$):  
&nbsp;&nbsp;&nbsp;&nbsp;if $$X$$ is in $$L$$-th layer:  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;return $$f(X)$$  
&nbsp;&nbsp;&nbsp;&nbsp;elif $$X$$ is in a maximizing layer ($$l$$ is even):  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;result := $$- \infty$$  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;for each child $$X_i$$:  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;result := $$\max$$(result, Minimax($$X_i$$))  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;return result  
&nbsp;&nbsp;&nbsp;&nbsp;else:  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;result := $$+ \infty$$  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;for each child $$X_i$$:  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;result := $$\min$$(result, Minimax($$X_i$$))  
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;return result  

This is a depth-first exploration: One branch is developed until the leaf, then the value of its predecessor is found evaluating 
all its children. Then the value of this predecessor will be used to find the value of its own predecessor, etc.

### Alpha-beta pruning

The alpha-beta pruning can be used to optimize the minimax algorithm. Its purpose is to skip the branches of which we know will not 
influence the decision. For this, during the tree search, two variables $$\alpha$$ and $$\beta$$ are used to store the minimum score 
the player is currently assured to have and the maximum score the opponent is assured to have. It makes sense as the player wants 
to maximize the value, they will not play any move scoring below $$\alpha$$ and as the opponent wants to minimize the value, 
they will not play any move scoring above $$\beta$$. That means that when $$\alpha \geq \beta$$, it's not worth exploring the rest 
of the branch anymore.

function Minimax($$X$$, $$\alpha$$, $$\beta$$):  
&nbsp;&nbsp;&nbsp;&nbsp;if $$X$$ is in $$L$$-th layer:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return $$f(X)$$  
&nbsp;&nbsp;&nbsp;&nbsp;elif $$X$$ is in a maximizing layer ($$l$$ is even):  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result := $$- \infty$$  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for each child $$X_i$$:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result := $$\max$$(result, Minimax($$X_i$$, $$\alpha$$, $$\beta$$))  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$\alpha$$ := $$\max$$($$\alpha$$, result)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if $$\alpha \geq \beta$$:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return $$\alpha$$  
&nbsp;&nbsp;&nbsp;&nbsp;else:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result := $$+ \infty$$  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for each child $$X_i$$:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result := $$\min$$(result, Minimax($$X_i$$, $$\alpha$$, $$\beta$$))  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$\beta$$ := min($$\beta$$, result)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if $$\alpha \geq \beta$$:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return $$\beta$$  

### Horizon effect

However, there is a major flaw in the algorithm. Because of the fixed maximum depth of the tree, the algorithm can only plan 
a certain number of moves forward. In games such as Checkers (or Chess, Go, etc.), there are moves that can drastically change 
the situation (e.g. capture of a significant number of pieces). If a move of this kind appear to be possible immediately after 
the maximum depth, the algorithm is not able to plan them. The algorithm would not be able to detect a catastrophic situation 
that could happen right after, which makes it inefficient. This is called the horizon effect.

This can be worked around by evaluating the quietness of a position. For example, we can suppose the position to 
be noisy if there are possible captures, quiet otherwise. Then, on the last layer, the algorithm can continue to develop the tree 
after the maximum depth, but only for the noisy positions, until all the leaves are quiet. Thus, for a small cost, the algorithm 
can avoid obvious hidden traps that were beyond its vision (or horizon).

