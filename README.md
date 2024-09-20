# C-Programming-Assignment
My second year college assignment for the C-Programming module was a voluntary competition for additional marks in which I came first place. 
The assignment was a space themed graph-traversal problem aimed at optimizing the least amount of node traversals between a source and destination node over 1000 random seeds.
A Euclidean distance heuristic from the current position is provided with each traversal, however information about the graph is only discovered locally within a single unit of euclidean distance.
The only information given per traversal is the updated euclidean distance to the destination node, and connected node IDs allowing the algorithm to traverse to them. Node connections are proximity
based and cann only occur within the bounds of 1 unit distance.

Space_explorer.c and Space_explorer.h are run on a seperate machine and cannot be altered. Competitors can only provide code in space_solution.c which returns a nodeID value to space_explorer.c 
through the space_hop() function.

Full description TBC.
