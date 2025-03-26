gametheory
============
- Category: Learning
- Tags:
- Created: 2025-03-25T18:44:58-08:00

## CHAPTER 1 - INTRODUCTION
- Game theory allows us to study interactions between two parties
	- Cooperative... and otherwise
		- We mostly focus on noncooperative games here
	- A "game" really means an engagement
- There are certain requisite components to make a complete game.
	- A list of players
	- A complete list of what the players can do
	- A description of what players *know* when they act
	- A specification of how player's actions lead to outcomes
	- A specification of player's preferences regarding outcomes

## CHAPTER 2 - EXTENSIVE FORM
- Two common forms exist in which noncooperative games are played
	- Extensive form
	- Normal (strategic) form
- Let us explore an example of noncooperative games and extensive form: the Katzenberg-Eisner game
	- Extensive form takes on the visual appearance of a tree
		- Nodes are decision points
		- Branches are each a respective possible decision
		- Nodes and branches make up a proper extensive-form representation
	- We start at the initial node - the initial decision taking place
		- Every extensive-form representation has exactly one of these
		- Here, Katzenberg decides to stay at or leave Disney
			- If he stays, we assume the game is over. No decision nodes follow.
	- Other decisions have to be made if he leaves
		- Eisner chooses to produce *A Bug's Life* or not
		- Katzenberg decides to produce *Antz* or not
			- This decision happens independently and regardless of if Eisner produces or not. 
				- Therefore we have the same decision branches eminating from the two nodes following Eisner's choice
	- With the extensive form, we can represent the player's information by describing whether they know where they are in the tree as the game progresses
		- Ex. Katzenberg moves first always, so he always knows that he's at node a when he's really there
			- Eisner observes Katzenberg leaving, so he knows that he's at node b
			- Katzenberg does **not** know if he's at node c or d, and **neither** player knows if the other decides to produce
				- We represent this with a dotted line between the two indistinguishable nodes, and describe them as being in the same **information set**. More on this later.
	- We traverse through the extensive form tree by making decisions at decision nodes and following branches.
		- We end up at *terminal nodes*, which represent outcomes of games.
		- In extensive form, there is a 1-to-1 relationship between unique paths and terminal nodes
	- As mentioned, we use the term *information set* to describe a player's knowledge at a point in the game.
		- Ex. the information set for node a is {a} because the node is distinguishable from others
		- Ex. the information set for nodes c and d are {c,d} because the nodes are indistinguishable
		- **Each information set corresponds with a decision**. 
			- Despite having multiple decision nodes, Katzenberg only makes one decision at that information set - to produce or not, blind to Eisner
	- How do we quantify "preference"? We can assign *utility* or *payoff* values to each outcome for each player
		- In Katzenberg-Eisner, the payoff is monetary, so it's easier to put a value to it.
	- Then, put a player to each node, a unique action to each branch, and a payoff vector to each terminal node

## CHAPTER 3 - STRATEGIES AND THE NORMAL FORM
- A **strategy** is a complete contingent plan for a player in a game
	- "Complete contingent" means that the plan specifies each move of the player at each of their information sets
	- Ex. for Katzenberg-Eisner, Katzenberg's strategies must specify the action to take at each information set: {a}, {c,d}, {e}
	- Ex. for 2.7(a), there are four strategies for player 2
		- HH'
			- "In the top information set, I go high. In the bottom information set, I go high."
		- HL'
			- "In the top information set, I go high. In the bottom information set, I go low."
		- LH'
			- "In the top information set, I go low. In the bottom information set, I go high."
		- LL'
			- "In the top information set, I go low. In the bottom information set, I go low."
	- Ex. for 2.7(b), there are two strategies for player 2
		- H
			- "I'm blind to player 1, so I will go high regardless"
		- L
			- "I'm blind to player 1, so I will go low regardless"
	- Note the usefulness of denoting differences between similar movements by the same player at different information sets. Avoids ambiguity.
- Let's introduce some notation
	- Given a game, let $S_i$ denote the *strategy space* (or *strategy set*) of player *i*
		- Meaning that $S_i$ represents all possible strategies of player *i*
	- Therefore, $s_i$ denotes a single strategy of player *i* in $S_i$. Insert "is in" character here.
		- Ex. $s_1$ could be {H}, $s_2$ could be {H,L'} for 2.7(a)
	- A *strategy profile* is a vector of strategies, one for each player.
		- Ex. a vector s = {$s_1$, $s_2$, $s_3$, ..., $s_i$}
	- Let *S* denote the set of all strategy profiles {$S_1$,$S_2$,$S_3$, ..., $S_i$}
		- Mathematically, think of this as S = $S_1$ * $S_2$ * $S_3$ * ... * $S_i$
	- Let $s_-i$ be a strategy profile for all players **except** for player *i*
		- $s_{-i} = (s_1, s_2, s_3, ..., s_{i-1}, s_{i+1}, ..., s_n)$
		- When splitting up a strategy profile *s*, it helps to denote $s = (s_i, s_{-i})$
		- We usually refer to the "$-i$" players as "opponents" even in cooperative situations
	- Ex. 3.1(b)
		-```
		 1  I   2  I   1  A
		 o ---> o ---> o ---> 4,2 
		 |		|	   |
		 | O    | O	   | B
		 |      |	   |
		 v		v	   v
		 2,2    1,3    3,4
		```
		- Player 1: $S_1 = {IA, IB, OA, OB}$
		- Player 2: $S_2 = {I, O}$
			- Note that we detail strategies even when they don't logically occur, like OA and OB.
	- How do we go about representing strategy sets as the number gets larger?
		- Ex. charitable project problem
			- Two players can contribute to a fund. 
			- If they collectively contribute at least $900, the project is successful and they each get 800 - (contribution)
			- If they are unsuccessful, they get no money back, so their payoff is -(contribution)
			- Each player has $600 to work with. Each player knows that the other player has this much.
			- Player 1 goes, player 2 observes then goes.
				- Describe $s_2$ as a function: $s_2: [0,600] -> [0,600]$
		- Note that Player 1 has one knowledge set and infinite choices, whereas Player 2 has infinite knowledge sets and infinite choices.
			- Functional representations help here
	- We can also represent payoffs in this way.
		- $u_i: S -> \mathbb{R}$
			- "Any strategy profile in the set of strategy profiles S goes in, any real number comes out (representing payoff)"
			- "The payoff for player i given a certain strategy profile describing all players"
		- Ex. $u_i(s)$ for 3.1(b) could be $u_1(IB,I) = 3$, $u_2(IB,I) = 4$, $u_1(IB,O) = 1$, etc.
		- When the strategies are seperate, we represent payoff vectors in matrices where each cell corresponds to a vector of payoffs for all n players
			- Formal def.: A game in *normal form* (aka *strategic form*) consists of:
				- A set of players {1, 2, ..., n}
				- Their corresponding strategy spaces {$S_1$, $S_2$, ..., $S_n$}
				- A payoff function for each unique combination of strategies {$u_1$, $u_2$, ..., $u_n$}
			- With two players, we can visualize this in a matrix. Ergo, normal form games are sometimes called "matrix games"
	- Examples of normal form games
		- Matching Pennies: Two players reveal their coins. If they choose the same, p1 gets both coins. Otherwise, p2 gets both coins.
			- ```
			       H      T
			H (1,-1) (-1,1)
			T (-1,1) (1,-1)
			```
		- Prisoner's Dilemma: Two prisoners (players) can cooperate or defect against each other. 
			- ```
			      C     D
			C (2,2) (0,3)
			D (3,0) (1,1)
			```
		- Battle of the Sexes: Guy and gal want to go on a date, but cannot communicate (move simultaneously and independently.) Guy prefers movies, gal prefers opera, but both would rather be together.
			- ```
			      O     M
			O (2,1) (0,0)
			M (0,0) (1,2)
			```
		- Hawk/dove AKA Chicken: Two people are driving towards each other. Both can swerve and save face. One can swerve and be "beaten". Neither can swerve, disfiguring both.
			- ```
			      H	    D
			H (0,0) (3,1)
			D (1,3) (2,2)
			```
		- Pigs: There's a dominant pig and a submissive pig by a bowl in their pen. On the other side of the pen is a button to dispense food. 
			- If neither pushes, nobody gets food. If S pig pushes, D pig eats all the food first. If D pig pushes, S pig can have some before D pig comes. If both push, D pig bullies S pig.
			- ```
			      P      D
			P (4,2)  (2,3)
			D (6,-1) (0,0)
			```
		- Coordination game: both players are rewarded if they choose the same strategy and are not if they don't
			- ```
			      A     B
			A (1,1) (0,0)
			B (0,0) (1,1)
			```
		- Pareto coordination game: Same as coordination, but now there is incentive to choose strategy A over B
	- It's not always easy to convert between extensive form and normal form! 
		- Consider asynchronous decisions, like the KE game. In this scenario, extensive -> normal works, but not the other way around.
		- One-shot (simultaneous, independent) games are well modeled by normal form, though.

## CHAPTER 4 - BELIEF, MIXED STRATEGIES, AND EXPECTED PAYOFFS

- When we play a game, we also think about other player's choices. This is a *belief.*
	- Formally, we represent "belief" as a probability.
	- $p$ likelihood that p2 chooses strategy C. $p$ is a probability where: 
		- $p = 1$ means p1 is certain that p2 will choose C
		- $p = 0$ means p1 is certain that p2 will **NOT** choose C 
		- Anything in between means that there's a belief that either *could* potentially be chosen.
	- Formally:
		- A belief of player $i$ is a probabilty distribution over the strategies of the other players.
			- Denote this as $\theta_{-i}$.
			- $\theta_{-i} \in \Delta S_{-i}$, where $\Delta_{-i}$ is the set of probability distributions regarding the strategy sets of all the players except $i$
				- Meaning that this probability is one of all possible distributions of $S_{-i}$
		- When player $i$ thinks about player $j$, for each strategy $s_j \in S_j$ of player $j$:
			- $\theta_j(s_j)$ is the probability that player $i$ thinks player $j$ will play $s_j$
			- Remember properties of a probability distribution - positive probability for each strategy, and all strategy probabilities add to 1.
	- If we expect a strategy to be selected based on a probability distribution, we call this a "mixed strategy." Some strategies may be assigned probability of 0.
		- A "belief" specifies the perceived probability of each other player. This belief can be that the opponent is playing a **mixed strategy** (selected off a distribution) or **pure strategy** (playing one strategy)
		- Denote a mixed strategy of player $i$ as $\sigma_i \in \Delta S_i$
			- **Pure strategy** can be thought of as a mixed strategy in which a player has a 100% chance of picking a certain strategy.
				- The set of mixed strategies therefore contains the set of pure strategies
		- If expecting a mixed strategy, we can't be sure of a certain payoff, so we calculate *expected value* based on the likelihood of each payoff for each $s_i$
			- When player $i$ has a belief $\theta_i$ about the strategies of the others and plans to select strategy $s_i$, their expected payoff is the "weighted payoff" they would get if they played $s_i$ and the others played $s_i$ 
			- Formally: $u_i(s_i, \theta_i) = \sum_{s_{-i} \in S_{-i}} \theta_{-i}(s_{-i})u_i(s_i,s_{-i})$
		- Ex. if my opponent has the available strategies *L*, *M*, and *R*, and I believe in a 50/25/25 chance of choosing each respective strategy, decide the expected payoff of one of my strategies by adding all the payoff * probability for each opposing strategy
		- What if we change two dominated strategies when believing in mixed strategy? Individual preference doesn't change, but expected value does.
			- There's more to payoff numbers than order! 

















