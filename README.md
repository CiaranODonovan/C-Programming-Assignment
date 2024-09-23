# C-Programming-Assignment
My second year college assignment for the C-Programming module class was also an opt-in leaderboard competition for additional marks in which I came first place. 
The assignment was a space themed graph-traversal problem aimed at optimizing the least amount of node traversals between a source and destination node over 1000 random graph seeds.
A Euclidean distance heuristic from the current position is provided with each traversal, however information about the graph is only discovered locally within a single unit of euclidean distance from the node the algorithm currently occupies.
The only information given per traversal is the updated euclidean distance to the destination node, and connected node IDs within 1 unit distance allowing the algorithm to traverse to them by returning their ID to main logic. 

Space_explorer.c and Space_explorer.h are run on a seperate machine and cannot be altered. Competitors can only provide code in space_solution.c which returns a nodeID value, and a graph memory value to space_explorer.c through the space_hop() function.

The particular problem of information only being attainable at cost of the optimization goal prevents common shortest path solutions and algorithms such as Djikstra's algorithm, A* or any other variant which optimizes the shortest path when the graph structure is known or requires no cost to discover the structure.The requirements of the project means any information obtained needs to be utilized to its fullest extent, without regard to speed or performance by the algorithmic solution, unlike many graph traversal algorithms. My solution utilizes a mixture of probabilistic heuristics from the euclidean distance and node connection properties, and exploitation of graph probability distribution information deducable from the main code's PRNG logic. 
In a graph of 300 nodes, this algorithm averages 13 traversals to the destination node, the lowest number achieved in the competition.

# Graph structure exploitation
The start node is always at least 60% of the entire graphs size away from the destination node. As a result a common strategy was to split the algorithm into two stages, one where it randomly jumps until a certain distance is achieved, and then a main pathfinder to hone in on the destination node. My algorithm instead shortcuts this process by finding a way to gain every nodeID before its first traversal by exploiting the PRNG used to calculate them. An analysis of the graph structure was required for this solution. 
