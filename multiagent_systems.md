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

### 2.2 - Solving known MDPs via value iteration
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
		- Example of "Q subdivision": metal reprocessing plant.
			- In -> Station 1 (Load and Unload) -> Station 2 (Clean) -> Station 3 (Process) -> Station 4 (Eliminate Waste) -> Station 1 (Circular) -> Out
			- Each station is an agent.
			- Each agent can choose to send more material to the next station, or to suspend flow. ("Pass" or "Suspend")
			- We can reason that each station chooses actions based on the station after it - "downstream."
				- As Station 3, I'm not going to send metal to Station 4 if I know it can't handle the new load.
			- Therefore, the global $Q$ function becomes $Q(a_1, a_2, a_3, a_4) = Q_1(a_1, a_2) + Q_2(a_2, a_3) + Q_3(a_3, a_4) + Q_4(a_4, a_1)$
				- We want to compute the max of the the global Q. More specifically, we want to find the actions $a_1, a_2, a_3, a_4$ which yield the greatest $Q$.
					- $\arg_{a_1, a_2, a_3, a_4} \max Q(a_1, a_2, a_3, a_4) = Q_1(a_1, a_2) + Q_2(a_2, a_3) + Q_3(a_3, a_4) + Q_4(a_4, a_1)$
				- Here, we can employ a *variable elimination* algorithm, where we optimize one agent at a time. 
					- Break the $\arg \max$ into parts. Let's start with agent 4.
						- $\max_{a_1, a_2, a_3} Q_1(a_1, a_2) + Q_2(a_2, a_3) + \max_{a_4} [Q_3(a_3, a_4) + Q_4(a_4, a_1)]$
						- "I want to find the only local $Q_i$ functions where $a_4$ is used, then group them and mark them as depending on $a_1$ and $a_3$"
							- "I can find the optimal value of $a_4$ if I am given the optimal values of $a_1$ and $a_3$"
					- Represent this internal $\max$ expression with a new function $e_4(A_2, A_3)$. This gives us the optimal action $a_4^*$ based on $a_1^*$ and $a_3^*$
						- $e_4(a_3,a_1) = max_{a_4}[Q_3(a_3,a_4) + Q_4(a_4,a_1)]$
					- Substitute this into our original global function
						- $\max_{a_1, a_2, a_3} Q_1(a_1, a_2) + Q_2(a_2, a_3) + e_4(a_3, a_1)$
					- We have now "eliminated" agent 4. Rather, we know that we can calculate $a_4$ given the other actions. Repeat this process.
						- $e_3(a_2,a_1) = max_{a_3}[Q_2(a_2,a_3) + e_4(a_3,a_1)]$
						- $\max_{a_1, a_2} Q_1(a_1, a_2) + e_3(a_2, a_1)$
						- $e_2(a_1) = max_{a_2}[Q_2(a_1,a_2) + e_3(a_2,a_1)]$						
						- $e_1 = \max_{a_1} e_2(a_1)$
					- With this, we calculate each optimal action in order and use it in the next step.
						- Calculate $a_1^*$ with $\arg_{a_1} \max e_2 (a_1)$
						- **ASK ABOUT THIS. DOESN'T MAKE SENSE**
			- We can implement this procedure in a number of ways. You're still taking an exponential pass over agents, but it ends up being fewer operations.
		
### 2.3 - Negotiation, auctions, and optimization
- Here, we discuss distributed problem solving in an economic context.

#### 2.3.1 - From contract nets to auction-like optimization
- Here's our scenario.
	- Imagine we have a problem. This problem is too large to do by a single agent, so we split it up into smaller parts and distribute it among agents.
		- For each agent $i$, there is a function $c_i$
		- For any set of tasks $T$, $c_i(T)$ is the cost that agent $i$ incurs when completing **all the tasks** in $T$.
		- Each agent has different capabilities ("strengths and weaknesses.")
			- Therefore, certain tasks will be easier (less costly) while others will be "harder" (more costly)
		- At the beginning of the problem, each agent starts out with some set of subtasks 
			- We assume that this is not optimal and therefore needs adjusting.
				- "Not optimal" = the sum of all agent's costs are not the lowest
		- Agents then enter a "negotiation process" which improves on their assignment, culminating in optimality
			- This process can have an "anytime property" - even if interrupted prematurely, the resulting assignment is still more optimal than the initial assignment.
		- Negotiation consists of:
			- Agents contracting out assignments among themselves
				- Each "contract" consists of exchanging tasks *and money*
	- For the previous algorithm, we can imagine a scenario where an agent "bids" their cost (or marginal cost) on an assignment and the "owning agent" gives it to the lowest bidder, repeating.
		- This generally requires agents to enter "money-losing contracts." But there are other contract types which don't require money losing!
			- "Cluster contracts" - contracting for a bundle of tasks
			- "Swap contracts" - a swap of two tasks between agents
			- "Multi-agent contracts" - simultaneous transfers among many agents.
- Three questions arise.
	- How can we minnimize costs of subproblems if there's a monolithic larger problem to be dealt with?	
		- To be discussed later
	- When/how do agents actually make offers?
		- Through predetermined negotiation scehemes. In this case, we're looking at auctions.
			- Each scheme consists of:
				- "Bidding rules" - Permissible ways of making offers
				- "Market clearing rules" - definition of the outcome based on offers
				- "Information dissemination rules" - information made available to agents throughout the process
			- Our auction has a explicit centralized "auctioneer"
	- Since we're in a cooperative setting, why is "money-losing" relevant?
		- Auctions in particular are a means of allocating scarce resources among *self-interested agents*
			- "Self-interested" = game theoretic
			- Our discussion of contract-nets has some similarities to the game-theoretic auction but does not apply directly.
- We'll discussing:
	- A linear program (LP) to address the problem of *weighted matching in a bipartite graph* - AKA the *assignment problem*
	- An integer program (IP) to address the problem of *scheduling*

#### 2.3.2 - The assignment problem and linear programming

- The assignment problem, or "problem of weighted matching in a bipartite graph", goes as follows:
	- A (symmetric) assignment problem consists of:
		- A set $X$ of $n$ objects
		- A set $N$ of $n$ agents
		- A set $M \subseteq N \times X$ of possible assignment pairs
		- A function $v : M \rightarrow \mathbb{R}$ giving the value of each assignment pair
	- An assignment is a set of pairs $S \subseteq M$ such that each agent $i \in N$ and each object $j \in X$ are in at most one pair in $S$
		- Meaning an agent or object can only be in up to one pair at once
	- A *feasible assignment* is one where all agents are assigned an object
	- A feasible assignment $S$ is *optimal* if it maximizes $\sum_{(i,j) \in S} v(i,j)$
		- "... if it maximizes the sum of the pair values"
		- Note that we're trying to maximize here - not minimize like in the previously mentioned contract-net example.
- See the book example. The solution is obvious in smaller problems, but in larger problems, this isn't the case.
	- So how do we find the optimal assignment algorithmically?
- First, encode the problem as a linear problem.
	- Specifically, we can make a matrix to indicate assignment, or "assignment matrix"
		- When pair $(i,j)$ is selected, $x_{i,j} = 1$. Otherwise, $x_{i,j} = 0$
	- We use this to "activate" or "deactivate" values. Now, we can express the linear program as follows:
		- Maximize $\sum_{(i,j) \in M} v(i,j) x_{i,j}$ ...
		- Subject to:
			- $\sum_{j | (i,j) \in M} x_{i,j} \leq 1 \forall i \in N$
				- Each $i$ can only be in up to one pair at once
			- $\sum_{i | (i,j) \in M} x_{i,j} \leq 1 \forall j \in X$
				- Each $j$ can only be in up to one pair at once
			- If you examine the equations, you can tell they allow for fractional matches - continuous. but we can solve this integrally. 
- **Lemma 2.3.2**: *The LP encoding of the assignment problem has a solution such that for eery $i,j$ it is the case that $x_{i,j} = 0$ or $x_{i,j} = 1$. Furthermore, any optimal fractional solution can be converted in polynomial time to an optimal integral solution.*
- Since any LP can be solved in polynomial time, **Corollary 2.3.3**: *The assignment problem can be solved in polynomial time*
	- This doesn't solve our problems, though. 
		- The polynomial time solution is $O(n^3)$
		- The solution isn't parallelizable.
		- If one of the parameters changes, we have to do the whole calculation over again.
	- Let's instead explore the economic principle of competitive equilibrium
		- Imagine that:
			- Each $j \in X$ has an associated price
			- The price vector is $p = (p_1, \cdots, p_n)$ where $p_j$ is the price of object $j$
		- ..., given:
			- An assignment $S \subseteq M$ 
			- A price vector $p$
		- ..., define the "utility" from an assignment $j$ to agent $i$ as $u(i,j) = v(i,j) - p_j$
			- Think of $p_j$ as the value cost to acquire $j$. 
			- Think of $v(i,j)$ as the value provided by object $j$ to agent $i$ once acquired.
		- An assignment and a set of prices are in *competitive equilibrium* when each agent is assigned the object that maximizes their utility given current prices.
		- Formally, *a feasible assignment $S$ and a price vector $p$ are in *competitive equilibrium* when, for every pairing $(i,j) \in S$, it is the case that $\forall k, u(i,j) \geq u(i,k)$ *
- **Theorem 2.3.5**: *If a feasible assignment $S$ and a price vector $p$ satisfy the competitive equilibrium condition then $S$ is an optimal assignment. Furthermore, for any optimal solution S, there exists a price vector such that $p$ and $S$ satisfy the competitive equilibrium condition.*
	- What does this mean?
		- We have the values of each assignment pair. The value may be different per pair.
		- We also have a (universal) price for each object, stored in a vector
			- An agent will "spend" some amount to "earn" some amount as payoff. This "utility" (marginal gain/loss) for the pair is $u(i,j)$
		- We're aiming to create an assignment pair for each agent. This set of assignment pairs is $S$.
		- Given an assignment $S$ and a price vector $p$, if each pair in the assignment cannot be improved by swapping objects between agents, then $S$ is optimal.
		- We can also see this the other way around. If we know $S$ is optimal, we know there's a $p$ such that the two together satisfy the competitive equilibrium condition.
	- What are the implications?
		- We can "search the space of competitive equilibria" (increment or iterate through versions of pairings and $S$) to find equilibrium (and therefore optimality)
			- One way to do this is with auction-like procedures where agents "bid" in a predetermined way.
			- We will look at "open outcry" (ascending auction-like procedures)
				- But only after we discuss optimization and competitive equilibrium.
- To understand why CE applies to optimization, let's look at a general form of our LP.
	- Maximize $\sum^n_{i=1} c_ix_i$
	- Subject to:
		- $\sum^n_{i=1} a_{i,j}x_i \leq b_j \forall j \in {1, ..., m}$
		- $x_i \geq 0 \forall i \in {1, ..., n}$
	- What does this mean?
		- These are the parts we discussed earlier for the linear program
		
## CHAPTER 3 - NONCOOPERATIVE GAME THEORY

- So far, we've only had cooperative situations. What about self-interested agents?
- We model an agent's interests with a *utility function.*
	- A mapping from the natural world to real numbers

### 3.1 - Self-interested agents

- This means the agent is interested in maximizing its own utility - not necessarily for good/bad things to happen to others.

### 3.1.1 - Friends and enemies

- See movie example (BOTS flavored)

### 3.1.2 - Preferences and utility

- Why is it alright to model as utility? The real world is much more complex
	- Utility theorists would say that utility represents *preference*. We often look to von Neumann and Morgenstein's equations.
		- Let $O$ denote a finite set of outcomes for an agent.
		- For any pair $o_1, o_2 \in O$:
			- Let $o_1 \succeq o_2$ denote that the agent **weakly prefers** $o_1$ to $o_2$
			- Let $o_1 \sim o_2$ denote that the agent **is indifferent** between $o_1$ and $o_2$
				- We can also write this as $o_1 \succeq o_2 \wedge o_2 \succeq o_1$
			- Let $o_1 \succ o_2$ denote that the agent **strictly prefers** $o_1$ to $o_2$
				- We can also write this as $o_1 \succeq o_2 \wedge \neg (o_2 \succeq o_1)$
	- There may be some uncertainty with how outcomes are selected. This is codified through **lotteries**
		- A lottery is the random selection of one of a set of outcomes according to specified probabilities.
		- i.e. a lottery is a probability distribution over outcomes written $[p_1 : o_1 ,..., p_k : o_k]$ where:
			- $o_i in O$
			- $p_i \geq 0$
			- $\sum^k_{i=1} p_1 = 1$ - all probabilities add to 1
		- Let $\mathcal{L}$ denote the set of all lotteries.
			- With this set, we can extend the $\succeq$ relation to apply to lotteries as well.
				- Meaning that an agent may strictly prefer/weakly prefer/be indifferent to one lottery over another
- **Axioms of utility theory**
	- **Axiom 3.1.1 - Completeness**
		- $\forall o_1, o_2, o_1 \succ o_2 or o_2 \succ o_1 or o_1 \sim o_2$
		- This tells us that there is an order in preference, with ties allowed
		- For every pair, the agent either prefers one over the other, or is indifferent.
	- **Axiom 3.1.2 - Transitivity**
		- $If \o_1 \succeq o_2 \wedge \o_2 \succeq o_3, then o_1 \succeq o_3$
		- There is an order between preferring one pair and another with overlapping elements. Otherwise, you could have infinite loss.
			- Consider the money pump problem
	- **Axiom 3.1.3 - Substitutability**
		- $If o_1 \sim o_2, then for all sequences of one or more outcomes o_3,...,o_k and sets of probabilities p, p_3,...,p_k for which p + \sum^k_{i=3} p_i = 1, [p:o_1, p_3:o_3,...,p_k:o_k] \sim [p:o_1, p_3:o_3,...,p_k:o_k]$
			- $P_{\mathcal{l}}(o_i)$ denotes the probability that outcome $o_i$ is selected by lottery $\mathcal{l}$
		- If an agent is indifferent between two outcomes, they are indifferent between lotteries that only vary in that outcome.
		- Lotteries can be nested inside one another, too. See book example.
	- **Axiom 3.1.4 - Decomposability**
		- $If \forall o_i \in O, P_{\mathcal{l}_1}(o_i) = P_{\mathcal{l}_2}(o_i) then \mathcal{l}_1 \sim \mathcal{l}_2$ 
		- An agent is always indifferent between lotteries which induce the same probabilities over outcomes, regardless of whether lotteries are single or nested.
		- Also sometimes called the *"no fun in gambling"* axiom because it implies that rolling the dice more times has no impact on preference.
	- **Axiom 3.1.5 - Monotonicity**
		- $If o_1 \succ o_2 and p > q then [p:o_1, 1-p:o_2] \succ [q:o_1, 1-q:o_2]$
		- If an outcome is preferred over another, the agent will prefer the lottery that gives more probability to that first outcome.
		- Called monotonicity because the value of the probability isn't important - just the relationship between lotteries
	- **Lemma 3.1.6**
		- $If a preference relation \succeq satisfies the axioms of$
			- $completeness$ (where all outcome pairs have $\succ, \prec, or \sim$ relations),
			- $transitivity$ (where preference forms an order between overlapping pairs),
			- $decomposability$ (where identical lotteries are indifferent regardless of form), and 
			- $monotonicity$ (where lotteries giving more of a preferred outcome are preferred), 
		- $... and if...$
			- $o_1 \succ o_2$
			- $o_2 \succ o_3$
		- $... then there exists some probability p such that for all p' < p, o_2 \succ [p':o_1, (1-p'):o_3]$
			- "If $o_1 \succ o_2 \succ o_3$ and we abide by the rules, then there's some probability of getting $o_1$ where below that point, I would rather take $o_2$ instead of risking it for $o_3$"
		- $... and for all p'' > p, [p'':o_1, (1-p''):o_3] \succ o_2$
			- "... and above that point, I would rather risk it for $o_1$ instead of taking a guaranteed $o_2$"
		- See the proof!
	- **Axiom 3.1.7 - Continuity**
		- $If o_1 \succ \o_2 and o_2 \succ o_3, \exists p \in [0,1] such that o_2 \sim [p:o_1 , (1-p):0_3]
	- If we accept completeness, transitivity, decomposability, monotonicity, and continuity, we must accept the existance of utility functions.
		- This is the following theoreom:
	- **Theoreom 3.1.8 - von Neumann and Morgenstern**
		- $If a preference relation satisfies completeness, transitivity, decomposability, monotonicity, and continuity:$
		- $... there exists a function u : \mathcal{L} such that:$
			- $u(o_1) \geq u(o_2) iff o_1 \succeq o_2, and $
				- "The utility of a preferred outcome is greater than the utility of a nonpreferred outcome in a pair"
			- $u([p:o_1,...,p_k:o_k]) = \sum^k_{i=1}p_iu(o_i)$	
				- "The utility of a lottery is the sum of (each outcome times its probability)"
			- See proof!
	- Why not use real values instead of utility?
		- Because utility and direct material payoff aren't always linearly related.
	- What if we don't want our utility to merely fit within $[0,1]$?
		- We can write a secondary utility function that transforms it - think $u'(o) = au(o) + b$ is also a utility function for an agent, so long as a and b are constant and a is positive.

### 3.2 - Games in normal form
- Normal form is the most common representation of a game.
	- States of the world only depend on the agents's combined actions
	- Most other forms of games can be reduced to this
- Normal-form games are defined as a tuple, where
	- $N$ is a finite set of players, indexed by $i$
	- $A = A_1 \times ... \times A_n$, where $A_i$ is a finite set of **actions** available to player $i$
		- Each vector $a = (a_1, ..., a_n) \in A$ is called an **action profile**
		- $A$ is every possible combination of actions by all players. Each of these combinations is an action profile.
	- $u = (u_1, ..., u_n)$ where $u_i : A \mapsto \mathbb{R}$ is a real-valued **utility/payoff function** for player $i$
		- This should map from the set of **outcomes**, not the set of actions (important!) Here, we assume that $O = A$

#### 3.2.3 - Examples of normal-form games
- Prisoner's dilemma
	- Both players can choose to defect or cooperate.
	- Any instance of $c > a > d > b$ is considered in the form of prisoner's dilemma
	- It's always worth it to defect
- Driver's side game
	- Two drivers in opposite directions decide to drive on the left or right. 
	- If they choose their own same side, they both get $x$. Otherwise, they get $0$
	- Example of **common payoff game**
		- A common payoff game is a game in which for all action profiles $a \in A_1 \times ... \times A_n$ and any pair of agents $i,j$, it is the case that $u_i(a) = u_j(a)$
	- Also called **pure coordination games** or **team games.** When there is no competition, players must focus on coordinating to gain the greatest possible payoff.
- Matching pennies game
	- Two players flip pennies.
	- If the coins are the same side up, Player 1 gets both coins. Otherwise, Player 2 gets both coins.
	- Example of a **zero-sum game**, which is in itself an example of a **constant-sum game**
		- A two-player normal-form game is **constant-sum** if there exists a constant c such that for each strategy profile $a \in A_1 \times A_2$ it is the case that $u_1(a) + u_2(a) = c$
		- Zero-sum means payoffs sum to 0 at each action profile. One player wins, the other player loses an equal amount
		- Rock-paper-scissors is another example of a zero-sum game
	
#### 3.2.4 - Strategies in normal-form games
- We've defined the actions available to a player, but not the strategies or available choices.
- Here are some options:
	- **Pure strategy**
		- Where a player selects a single action and plays it.
		- A choice of pure strategy for each player together is called a **pure strategy profile**
	- **Mixed strategy**
		- Where a player randomizes over the set of available actions according to some probability distribution
		- Why would we introduce randomness into our own choice? We'll talk about it later
		- Let $(N,A,u)$ be a normal-form game, and for any set $X$ let $\Pi(X)$ be the set of all probability distributions over $X$. Then the set of **mixed strategies** for player $i$ is $S_i = \Pi(A_i)$
		- The set of **Mixed-strategy profiles** is simply the Cartesian product of the individual mixed-strategy sets $S_1 \times ... \times S_n$
		- $s_i(a_i)$ denotes the probability that an action $a_i$ will be played under mixed strategy $s_i$
		- The subset of actions that are assigned positive probability by the mixed strategy $s_i$ is called the **support** of $s_i$
			- Def: the **support** of a mixed strategy $s_i$ for a player $i$ is the set of pure strategies $\{ a_i | s_i(a_i) > 0 \}$
	- A pure strategy is simply a mixed strategy where the support is a single action
		- On the other end, a fully mixed strategy is a strategy with full support (i.e. every action has a nonzero probability)
	- Payoff of a pure strategy is easy - just look at the matrix. However, it's not the same with mixed strategy. We must instead calculate the **expected utility**
		- Given a normal-form game $(N,A,u)$, the expected utility $u_i$ for player $i$ of the mixed-strategy profile $s = s_1, ..., s_n$ is defined as:
			- $u_i(s) = \sum_{a \in A} u_i(a) \prod^n_{j=1}s_j(a_j)$
			- "We calculate the utility of an action profile times the probability that it actually happens. That probability is the total product of all players playing that action."

### 3.3 - Analyzing games: from optimality to equilibrium

#### 3.3.1 - Pareto Optimality
- Intuitively, some games are more preferable to others
- We can formalize the intuition of preferring one outcome to another as Pareto domination.
	- Strategy profile $s$ **Pareto dominates** strategy profile $s'$ if $\forall i \in N, u_i(s) \geq u_i(s')$, and there exists some $j \in N$ for which $u_j(s) > u_j(s')$
		- "In a Pareto-dominated strategy, some player can be made better off without making any other player worse off."
	- Strategy profile $s$ is **Pareto optimal**, or **strictly Pareto efficient**, if there does not exist another strategy profile $s' \in S$ that Pareto dominates $s$
		- Pareto domination is a partial ordering over strategy profiles. Some profiles dominate others, but it's not absolute.
		- It's hard to identify a single "best" strategy, but we can have multiple noncomparable optima.
	- Each game has at least one Pareto optimum, and there always exists at least one such optimum where all players play pure strategies.
	- Some games have multiple optima.
		- In zero-sum games, *all* strategy profiles are strictly Pareto efficient.
		- In common-payoff games, all Pareto optimal strategy profiles have the same payoffs.

#### 3.3.2 - Defining best response and Nash equilibrium
- Look at the game from a single agent's point of view.
	- If I were a single agent and I knew what everyone else was going to play, my objective is to choose the best play in response to the others.
		- This is **best response.**
		- Player $i$'s best response to the strategy profile $s_-i$ is a mixed strategy $s_i^* \in S$ such that $u_i(s_i^*, s_{-i}) \geq u_i(s_i,s_{-i})$ for all strategies $s_i \in S_i$
			- Where $s = (s_i, s_{-i})$ is a way to write strategy profile $s$ as consisting of player $i$'s strategy and the strategy profile of all other players (denoted $s_{-i}$)
			- "The best response is the strategy that gives you equal or higher payout than every other strategy in response to everyone else's played strategy profile."
		- Not necessarily unique - unless best response is a pure strategy, there are an infinite number of best responses.
			- In the same vein, any mixture of two pure best responses is also a best response.
	- We don't know what everyone else is going to play, though. A best response doesn't identify an interesting set of outcomes - not a solution concept. We use it for Nash equilibrium, though.
- **Nash equilibrium**
	- A strategy profile $s = (s_1, ..., s_n)$ is a Nash equilibrium if, for all agents $i$, $s_i$ is a best response to $s_{-i}$
		- "A strategy profile is a Nash equilibrium if every agent is playing its best against every other agent"
		- NE is a *stable* strategy profile, meaning each agent would not benefit from switching its own strategy if it knew the play of others.
	- We can divide NE into two parts: **strict** and **weak** Nash
		- Strict Nash - a *unique* best response
			- A strategy profile $s = (s_1, ..., s_n)$ is a *strict Nash equilibrium* if, for all agents $i$ and for all strategies $s'_i$, $u_i(s_i,s_{-i}) > u_i(s'_i, s_{-i})$
		- Weak Nash - a *nonunique* best response
			- A strategy profile $s = (s_1, ..., s_n)$ is a *weak Nash equilibrium* if, for all agents $i$ and for all strategies $s'_i$, $u_i(s_i,s_{-i}) \geq u_i(s'_i, s_{-i})$ and $s$ is not a strict Nash equilibrium
			- Weak NE are "less stable" - in this, at least one player has a best response that isn't their equilibrium strategy
#### 3.3.3 - Finding Nash equilibria
- Mixed-strategy NS are necessarily weak, while pure-strategy NE can be strict or weak (depends on game)
	- What's the intuition behind this? Using example of Battle of Sexes
		- Wife = row, husband = col.
		- Both choosing WL = (2,1)
		- Both choosing LW = (1,2)
		- No coordination = (0,0)
	- Mixed NE for wife is $(\frac{2}{3}, \frac{1}{3})$ with expected payoff of $\frac{2}{3}$
		- What does this mean? It means that if she plays this strategy, the husband has no preference among his own strategies.
			- As in he can play either pure strategy to also get an expected payoff of $\frac{2}{3}$
				- Since he's indifferent between pure strategies, he's also indifferent between any mixed strategy of the two.
	- Mixed NE for the husband is $(\frac{1}{3}, \frac{2}{3})$
	- When both play their mixed NE, they are playing one of infinite best responses to one another's NE.
		- There is no **strict** incentive to deviate, and if either deviates, the other player is incentivized to play a pure strategy in a later game (due to updating beliefs about $p$ or $q$)

#### 3.3.4 - Nash's theoreom: proving the existence of Nash equilibria
- Lots of math here... skipping

### 3.4 - Further solution concepts for normal-form games
- NE is a **solution concept**, in which we identify interesting subsets of game outcomes. Let's explore some others.

#### 3.4.1 - Maxmin and minmax strategies
- **Maxmin**
	- Maxmin *strategy* - the strategy that maximizes player $i$'s worst-case outcome
		- The maxmin strategy for player $i$ is $\arg \max_{s_i} \min{s_{-i}} u_i(s_i, s_{-i})$
	- Maxmin *value* - the minimum amount of payoff guaranteed by a maxmin strategy. AKA security level.
		- The maxmin value for player $i$ is $\max_{s_i} \min{s_{-i}} u_i(s_i, s_{-i})$
	- Think of it temporally.
		- Player $i$ commits to a (pure/mixed) strategy.
		- Player(s) $-i$ commit to strategies such that player i's payoff is minimized
		- Maxmin is the highest possible value that player $i$ can play into the worst-case strategies.
	- Other agents probably won't conspire against you, but it's just the "safe guarantee" without making assumptions about other agents.
- **Minmax**
	- Minmax *strategy* - the strategy which minimizes player $-i$'s best-case outcome.
		- In a two-player game, the minmax strategy for player $i$ against player $−i$ is $\arg \min_{s_i} \max_{s_{−i}} u_{−i} (s_i, s_{−i})
		- In an n-player game, the minmax strategy for player $i$ against player $j \neq i$ is $i$’s component of the mixed-strategy profile $s_{−j}$ in the expression $\arg \min_{s_{−j}} \max_{s_j} u_j(s_j,s_{−j})$, where $−j$ denotes the set of players other than $j$.
	- Minmax *value* - the highest value that player $-i$ can achieve given minmax play by player $i$
		- In a two-player game, player $−i$’s minmax value is $\min_{s_i} \max_{s_{−i}} u_{−i}(s_i,s_{−i})$.
		- In an n-player game, the minmax value is $\min_{s_{−j}} \max_{s_j} u_j(s_j,s_{−j})$
	- Think of it temporally
		- Player $i$ / players $j$ commit to a (pure/mixed) strategy to minimize $-i$/$-j$'s expected best response. 
		- Player $-i$/$-j$ receeive their minimax value if they best respond to this.
- Neither maxmin nor minmax are dependent on the other players' actions, so this is a fairly straightforward solution concept.
- Given a mixed strategy profile $s = (s_1, s_2, ...)$
	- We call it a *maxmin strategy profile* of a given game if everyone plays their maxmin strategy
		- In two-player games, analagous to *minmax strategy profiles*
		- In two-player, zero-sum games, very tight connection between minmax and maxmin strategy profiles
- See proof - in 0sum 2p games, $v_i = \overline{v}_i = \underline(v)_i$
- This allows to conclude that, in 0sum 2p games:
	- Each player's maxmin value is equal to their minmax value. This is the "value of the game" for player $i$
	- The set of maxmin strategies coincides with the set of minmax strategies
	- Any maxmin/minmax strategy profile is a NE. These strategy profiles are the only NE. All NE have the same payoff vectors.
- View a NE in 0-sum games as a "saddle-point" in high-dimensional space.

#### 3.4.2 - Minimax regret

- Maxmin is useful to guarantee maximum payout against a minimizing opponent. However, what about an entirely unpredictable player for whom we have no belief (i.e. not Bayesian?)
- Consider the example game in the book. You are the row player.
  - Let's say we don't know (and don't care) about player C's payoffs. Just considering our own.
    - In fact, we're uncertain about player C, which is why we're playing a minimax strategy.
  - Our maxmin strategy would be to play B, since B contains our best worst-case response.
  - If we don't think the other player is malicious, then we would lose out on a ton of potential payoff by not choosing T.
  - This "potential loss" is called **regret**.
- **Regret**
  - An agent $i$'s *regret* for playing an action $a_i$ if the other agents adopt action profile $a_{-i'}$ is defined as $[\max_{a'_i \in A_i}u_i(a'_i, a_{-i})] - u_i(a_i, a_{-i})$
  - AKA the difference between your payoff and the max amount of payoff you could have gotten by choosing another action given the opponent's action.
- **Max regret**
  - An agent $i$'s *maximum regret* for playing an action $a_i$ is defined as $\max_{a_-i \in A_-i} ([\max_{a'_i \in A_i} u_i(a'_i, a_{-i})] - u_i(a_i, a_{-i}))$
  - AKA the maximum amount of regret (resulting from a maximizing action by $-i$) that agent $i$ can recieve for playing $a_i$
- **Minimax regret**
  - Minimax regret actions for agent $i$ are defined as $\arg \min_{a_i \in A_i}[\max_{a_-i \in A_-i} ([\max_{a'_i \in A_i} u_i(a'_i, a_{-i})] - u_i(a_i, a_{-i}))]$
  - AKA the action that yields the smallest maximum regret.
    - Note that this is the regret of an *action*, not a *strategy*. Try proving this!

#### 3.4.3 - Removal of dominated strategies

- One strategy dominates another for player $i$ if the first strategy yields $i$ a greater payoff than the second strategy, for any strategy profile of the remaining players.
  - Three levels to this, though.
    - **Strict domination**: If $u_i(s_i, s_{-i}) > u_i(s'_i, s_{-i}) \forall s_{-i} \in S_{-i}$
      - AKA $s$ strictly dominates $s'$ if it's better in all situations 
    - **Weak domination**: If $u_i(s_i, s_{-i}) \geq u_i(s'_i, s_{-i}) \forall s_{-i} \in S_{-i}$ and for at least one $s_{-i}$, $u_i(s_i, s_{-i} > u_i(s'_i, s_{-i}))$
      - AKA $s$ weakly dominates $s'$ if it's better in at least one situation and equal in all others
    - **Very weak domination**: $u_i(s_i, s_{-i}) \geq u_i(s'_i, s_{-i}) \forall s_{-i} \in S_{-i}$
- A strategy is (some variant of) **dominant** for an agent if it dominates any other strategy for that agent.
  - Strategy profiles where all strategies for agents are dominant for $i$ is a NE.
    - This forms an *equilibrium in dominant strategies* (between players)
  - An equilibrium made of strictly dominant strategies is the unique NE.
    - Note the Prisoner's Dilemma. The equilibrium at (D,D) is also the only outcome that's *not* Pareto optimal.
- A strategy $s_i$ is dominated if some other strategy $s'_i$ dominates $s_i$ in some manner
  - All strictly dominated strategies can be ignored, or **eliminated**
  - Eliminating one strategy can cause other strategies to become dominated
  - A pure strategy may be dominated by a mixture of other pure strategies without being dominated by any of them independently
- Elimination is a pretty weak solution concept, but we can use it to trim down matrices for computational advantage.
  - Sometimes (but not often), a game can be completely solved this way, by iteratively eliminating down to a single cell.
    - We would say such a game is solvable by iterated elimination
- Order of elimination doesn't matter for strictly dominated strategies (*Church-Rosser* property.)
- Which one should we use?
  - It depends. Strict -> weak -> very weak gives us more and more reduced matrices at the cost of potentially dubious eliminations caused by order of elimination.

#### 3.4.4 - Rationalizability


