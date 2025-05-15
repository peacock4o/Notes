multiagent_systems
==================
- Category: Learning
- Tags: 
- Created: 2025-04-26T14:48:25-07:00

## CHAPTER 1 - DISTRIBUTED CONSTRAINT SATISFACTION

### 1.1 - Defining distributed CSP
- Multiagent systems are common. Here's a good example: distributed constraint satisfaction
	- Given a set of variables and a set of constraints between variables, how do we assign these variables to satisfy the constraints?
- Useful way to visualize this - graph coloring problem.
	- Three nodes connected to one another have the constraints that they cannot be colored the same as other nodes.
		- Each can be red, blue, or green
- How do we formally define a CSP?
	- A CSP consists of:
		- Finite set of variables $X = {X_1, X_2, ..., X_n}$
			- Ex. $X = {X_1, X_2, X_3}$
		- Domain $D_i$ for each variable $X_i$
			- Ex. $D_1 = {red, blue, green}, D_2 = ...$
		- Set of constraints ${C_1, C_2, ..., C_m}$
			- Constraints are predicated on some subset of the variables $X$
			- Contraints restrict the participating variables
			- Ex. `!(C_1 = red && C_2 = red)`
	- *Instantiation* of variable subset $S \in X$ = assignment of a unique domain value for each variable in $S$
		- It's a *legal instantiation* if all constraints which only mention variables in $S$ are satisfied
		- A *solution* to a network is a legal instantiation of all variables
- In distributed CSP:
	- Each variable is controlled by an independent agent
	- Agents can (generally) assign their own variable
	- Agents can (generally) only communicate with neighbors in the constraint graph
	- Agents follow a distributed algorithm - local computation + communication with other agents = organization!
		- Good algorithms find (or disprove) a solution quickly
		- We examine two types of algorithms:
			- Least-commitment approach
			- Heuristic approach
	- Assume for all problems that messages arrive on time and in order

### 1.2 - Domain pruning algorithms
- Consider: domain-pruning algorithms
	- We start with the Revise function
		- "For every value $v_i$ in my domain $D_i$, if my neighbor has no value $v_j$ in $D_j$ with which our constraint is satisfied, remove $v_i$ from $D_i$"
		- Also known as "arc consistency"
	- If this function terminates with one solution per node, we have a solution
	- If it terminates with more than one solution per node, we *might* have a solution - inconclusive
	- If it terminates when a node has no values in domain, there is no solution
	- (!) The algorithm is sound (always correct) and guaranteed to terminate, but is not complete (may fail to produce a verdict)
		- See default red/green/blue algorithm - this algorithm can be used for preprocessing, but is very weak
	- Based on *unit resolution* from propositional logic
		- "If $A_1$ (is true) and $\neg(A_1 \land A_2 \land ... \land A_n)$, then $\neg(A_2 \land A_3 \land ... \land A_n)$"
		- Translate this to the situation:
		- "If they can only be red and we can't both be red, then I can't be red."
			- A *Nogood* is a rule detailing a forbidden value combination.
			- We write $\neg(x_1 = red \land x_2 = red)$ as the Nogood ${x_1, x_2}$
	- This is a weak rule, though. We need something bigger: *hyper-resolution*
		- Essentially the same as resolution, but for all possibilities of the leading variable at once.
		- See paper for examples, but here's an example flow:
			- Node $i$ can be $v_1 \lor v_2 \lor ... \lor v_n$
			- We also have a set of Nogoods, one for each $v_n$
			- We hyperresolve these combinations to generate a new Nogood (like in unit resolution,) if it doesn't already exist in the node's set
- Using hyper-resolution, consider the following algorithm: ReviseHR
	- Following flow:
		- "I am agent $i$ and I have a set of Nogoods $NG_i$. I also just got a set of Nogoods from agent $j$ called $NG^*_j"
		- "I'll add any Nogoods from $NG^*_j$ to $NG_i$ if I didn't already have them"
		- "Now, I'll hyper-resolve and try to generate any new Nogoods from my new set. Collect these in $NG^*_i$"
		- "If my set **is nonempty**, then add $NG^*_i$ to $NG_i$ and send it out to my neighbors."
			- If I produce no Nogoods, I didn't receive any new information. This is fine, but since $NG_i$ presumably changed, keep working.
			- Producing nothing is **NOT** the same as producing an empty set. Empty set means there is no logical solution to satisfy the problem.
		- "If I find the empty set in my generated set of Nogoods, then there's no logically no solution and I should stop working."
			- I *want* to send the empty set, but stop myself from continuing after.
		- "Repeat this flow until there's no more changes in my set of Nogoods $NG_i$"
	- See red/blue exclusive nodes example - generate Nogoods sequentially until there are no more options, then deduce an empty set
- This algorithm isn't the most practical - we have to send massive amounts of Nogoods. Neither was the Revise function. Why?
	- Because they're *least commitment* - they are restricted to removing only provably impossible value combinations.
	- Instead, we can use heuristics. Try out values, and backtrack when they don't work.

### 1.3 - Heuristic search algorithms
- Name of the game - try out certain values, communicate between nodes and backtrack when needed.
- Consider a centralized heuristic algorithm
	- Central controller orders nodes, tries out values in a row, backtrack earlier in the row when stuff doesn't work.
	- This is nice, but not distributed.
- Consider a (naive) distributed heuristic algorithm
	- Each node tries out a value and notifies neighbors of value. If not consistent with constraints, switch and notify.
	- This is alright, but it might cycle forever, even with a solution
- Let's combine these two (flawed) algorithms into something nicer. Specifically, let's include:
	- The ordered structure of nodes from alg. 1
	- The messaging algorithm from alg. 2
- Result: **asynchronous backtracking algorithm (ABT)**
	- Key notes:
		- Each node communicates its value updates to *lower* priority nodes ONLY
		- When a node receives a value from a higher node, it checks for its own congruency.
			- If it doesn't need to move, it doesn't move
			- If it needs to move and is able to find a new spot, it moves and sends another update to lower nodes
			- If it needs to move and is unable to find a new spot, it deletes the value of the next highest node from its agent view, moves, updates lower nodes, and sends a Nogood with the (pruned) agent view to the next highest node (value previously deleted from view.)
		- When a node receives a Nogood from a lower node, it records it and attempts to find a new spot. See above steps.
	- Example: $n$ queens problem. Try running through it!
	- ABT can be optimized for sure. Mostly revolves around Nogoods being simplified.
		- Finding an absolutely minimal Nogood is NP-hard. Use various heuristics to cut down size without being wrong.

## CHAPTER 2 - DISTRIBUTED OPTIMIZATION

### 2.1 - Distributed dynamic programming for path planning

#### 2.1.1 - Asynchronous dynamic programming
- To discuss this, let's first consider the path planning problem
	- The path planning problem consists of:
		- A set of nodes $N$, comprising of $n$ nodes.
		- A set of directed links between nodes $L$
		- A weight function $w : L \rightarrow \mathbb{R}^+$
			- Meaning each link has a weight
		- Two nodes $s,t \in N$
		- The goal is to find the path from $s$ to $t$ with the lowest total weight
	- Also think about it like this:
		- Consider a set of goal nodes $T \subset N$. We are interested in the shortest path from $s$ to any node $t \in T$
	- If we have a node $x$ on the shortest path between $s$ and $t$, then the section between $s$ and $x$ must also be the shortest path for them.
		- Allows us to execute *dynamic programming*: "divide-and-conquer" style
- Represent the shortest distance from node $i$ to goal node $t$ as $h^*(i)$
	- $h^*(i)$ = value function of actual shortest path to goal $t$
	- $h(i)$ = value function of node $i$'s current estimate of shortest path to goal $t$
- Consider: AsyncDP alg
	- Per node $i$, the shortest distance from $i$ to $t$ can be determined by :
		- For each neighbor $j$:
			- Add their current $h(j)$ and our weight $w(i,j)$. Set my output value for $f(j)$ to this.
		- $h(i)$ is $min_jf(j)$
	- This doesn't scale well - since it's worst case O(n) time, having huge search spaces doesn't really work.
		- Time to use heuristics!

#### 2.1.2 - Learning real-time A* (LRTA*)
- Let's start with one agent. Worst case.
	- Our current node is $i$. Set this to $s$ (start node) when initializing.
	- Until we reach $t$:
		- For each neighbor $j$:
			- Add their current $h(j)$ and our weight $w(i,j)$. Set my output value for $f(j)$ to this.
				- *Initially, $h(j)$ will be 0. As trials take place, this will only grow.*
		- Get the node with the smallest estimated path $arg min_jf(j)$. Set *i'* to this.
		- Before moving, let's decide if we need to update and grow our current estimate. $h(i) \leftarrow max(h(i), f(i'))$
			- "If the newly estimated shortest path through $j$ called $f(i')$ is greater than the old estimated path to $h(i)$, update it." 
	- Think like this: 
		- We find the initial shortest path.
		- Run through more trials. If there's any legitimate reason to try out another path, take it.
		- Continue until we don't try out any other paths - until we repeat the same path twice in a row.
			- We don't need to try every single path!
- LRTA* has some assumptions:
	- Initialize node $h(i)$ to 0
	- Each node has at least one path to $t$
- LRTA* has some properties:
	- $h$ values never decrease and remain *admissible* (never more than $h*(i)$)
		- Ensure this by initializing nodes to 0 and only adding when a new shortest local path has been found.
	- LRTA* always terminates. Also, each successful completion of a pass of LRTA* is called a trial.
	- Repeating trials of LRTA* will converge on the shortest path when maintaining $h$-values.
	- If LRTA* finds the same shortest path on two sequential runs, this is the shortest path.
- LRTA* is a centralized algorithm, but can be executed (and sped up) with multiple agents. See book example in Figure 2.5
	- This is especially helpful to break up ties.


## APPENDIX C - MARKOV DECISION PROBLEMS (MDPs)

### C.1 - The model
- "A Markov Decision Problem (MDP) is a model for decision making in a dynamic, uncertain world."
- An MDP is a tuple $(S,A,p,r)$
	- $S$ - Set of states
	- $A$ - Set of actions
	- $p$ - Function $p : S \times A \times S \mapsto \mathbb{R}$ which specifies the *state transition probability* among states.
		- $p(s,a,s')$ - "When I am at state $s$ and I take action $a$, what is the probability that I end up in state $s'$?
	- $r$ - Function $r : S \times A \mapsto \mathbb{R}$ which specifies the reward given by taking an action $a$ while in state $s$
		- Rewards are aggregated in two ways
			- Limit-average reward
				- $\lim^\infty_{T=1}\frac{\sum^T_{t=1}r^(t)}{T}$
					- "$r^(t)$ is the reward you get at time step $t$."
					- "Average out all the rewards you get over the span of performing $T$ total time steps"
					- "As T grows, the average converges on a certain value."
			- Future-discounted reward
				- $\sum^{\infty}_{t=1}\beta^t r^{t}$
					- "No averaging this time - we're just summing all the reward values"
					- "However, we're multiplying each value by a coefficient raised to the power of the time step."
						- Coefficient is $\beta^t$, where $0 < \beta < 1$
					- "This means that as the time step gets larger, $\beta^t$ grows exponentially smaller - converging on 0"
						- Also meaning that the future-discounted reward is a finite sum!
					- "When $\beta \rightarrow 0$, future rewards are more heavily discounted - prioritizing short-term reward"
					- "When $\beta \rightarrow 1$, future rewards are less heavily discounted - prioritizing long-term reward"
- A (stationary, deterministic) policy $\pi : S \rightarrow A$ maps each state to an action.
- For the future examples, we'll use future-discounted reward

### C.2 - Solving known MDPs via value iteration
- Each policy yields a total reward under each reward aggregation scheme.
- A policy that maximizes total reward is called an *optimal policy*
	- Generally, the algorithm to find an optimal policy is the computational task at hand
- While linear programming can solve MDPs in polynomial time, let's instead use *value iteration* for two reasons
	- LP-formulation of MDP is often too large to compute realistically, despite being in polynomial time
		- Value iteration is the basis of more practical solutions, including approximations of very large MDPs
	- Value iteration is relevant to the discussion of learning in MDPs
- Value iteration (VI) operates as such:
	- VI defines a value function $V^{\pi} : S \rightarrow \mathbb{R}$
		- $V^{\pi}(s) = Q^{\pi}(s, \pi(s))$
			- Meaning given a state $S$, find the value of acting according to policy $\pi$
			- In my mind, this is "naive." We only care about following the exact action prescribed in the policy.
	- VI also defines a state-action function $Q^{\pi} : S \times A \rightarrow \mathbb{R}$
		- $Q^{\pi}(s,a) = r(s,a) + \beta \sum_{\hat{s}} p(s,a,\hat{s})V^{\pi}(\hat{S})$
		- Meaning given a state $s$ and an action $a$, find the value of starting in $s$, taking action $a$, and continuing according to policy $\pi$
	- $V$ gives us the value of following a policy action at a state. $Q$ gives us the value of taking an action at a state, plus the values of all following states.
- Instead of the naive equations, imagine that we have an optimal policy $\pi^*$
	- The second equation would become $V^{\pi^*}(s) = \arg_a \max Q^{\pi^*}(s,a)$
		- Meaning that instead of choosing based on a state-action hardcoded by the policy, choose the action for which $Q^{\pi^*}(s,a)$ yields the greatest reward.
	- When working with $V^{\pi^*}$ and $Q^{\pi&*}$, the equations are referred to as the *Bellman equations*
	- The Bellman equations also give us a procedure for calculating the Q and V values of the optimal policy - ergo, they give us the optimal policy itself.
	- Consider:
		- $Q_{t+1}(s,a) \leftarrow r(s,a) + \beta \sum_{\hat{s}} p(s,a,\hat{s}V_t(\hat{s}))$
		- $V_t(s) \leftarrow \arg_a \max Q_t(s,a)$
			- **THIS RETURNS THE OPTIMAL ACTION**
	- Given an MDP and initializing Q values to an arbitrary value, repeatedly iterate the two sets of assignment operators.
	- After some set amount of time, Q and V values converge on an optimal policy.
	- What the heck does this even mean?
		- For $Q_{t+1}(s,a)$:
			- Every state is paired with every action. This tuple is assigned a value which eventually converges on the total (current + future reward) of taking action $a$ in state $s$.
		- For $V_t(s)$:
			- Every state is assigned an action which is the most optimal (meaning it has the largest total reward.) This is done by calculating $Q_{t+1}(s,a)$ for each $a$ and setting $V_t(s)$ to the highest yielding $a$
- In the real world, it's not so simple as performing these things.
	- The MDP may not be fully known beforehand, requiring learning.
	- The MDP may be too large to iterate over all instances of the equation.
		- Example: instead of a single value, states are represented as feature values where a state can take on many values.
			- This would result in exponential state number - 2 states can be `[[], [a], [b], [a,b]]` - 4 states.
			- We'd like to solve this in time polynomial to the number of features
		- To solve this, exploit independence properties.
			- Maybe this is like the subgames?
	- We discuss something similar to the above problem, but instead dealing with modularity of *actions* rather than states.
		- In a *multiagent MDP* (think each node calculating for itself), any action $a$ is really a vector of local actions $(a_1, a_2, ..., a_n)$.
			- This means that the number of global actions is exponential to the number of agents.
			- **But why???? What makes these actions different? It's not like they are $Q$ or $V$ values? Need to ask.**
				- **ANSWER**: Each "action" is a vector of local actions by each of $n$ agents. This means that each agent moves simultaneously. $A = A_1 * A_2$
	- Let us consider a further subproblem
		- Suppose that the $Q$ values for the optimal policy are already computed. How hard is it to decide the action each agent should take?
		- In Appendix C, we mention that once an optimal policy has been converged on, we recover the optimal action in state $s$ with $argmax_a Q^{\pi^*} (s,a)$
			- This is "easy" to do with single agent. $A^n$ where $n = 1$ is linear. 
			- This is "hard" to do with multiple agents. $A^n$ where $n > 1$ is exponential time, so as $n$ grows, choosing the max $a$ takes exponentially longer.
			- Can we do better?
				- Generally, no. But interaction among agent actions can be quite limited.
				- We can exploit this. Think about calculating overall Q for multiple agents as calculating the Q (total reward) for each agent $i$ for each joint action in $A$, then adding all those indivual Q values up. Pretty intuitive.
				- Our overall Q function becomes $Q(s,a) = \sum^n_{i=1}Q_i (s,a)$
					- Our maximization problem becomes the max value of this new sum. We're finding the joint action which gives us the greatest joint total reward.
					- $\arg_a \max \sum^n_{i=1}Q_i (s,a)$ 
				- This doesn't solve our $A^n$ problem, butgenerally speaking, actions can be treated as discrete and noneffective on one another.
					- "Each $Q_i$ depends only on a small subset of variables"
