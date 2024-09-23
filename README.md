# C-Programming-Assignment
My second year college assignment for the C-Programming module class was also an opt-in leaderboard competition for additional marks in which I came first place. 
The assignment was a space themed graph-traversal problem aimed at optimizing the least amount of node traversals between a source and destination node over 1000 random graph seeds.
A Euclidean distance heuristic from the current position is provided with each traversal, however information about the graph is only discovered locally within a single unit of euclidean distance from the node the algorithm currently occupies.
The only information given per traversal is the updated euclidean distance to the destination node, and connected node IDs within 1 unit distance allowing the algorithm to traverse to them by returning their ID to main logic. 

Space_explorer.c and Space_explorer.h are run on a seperate machine and cannot be altered. Competitors can only provide code in space_solution.c which returns a nodeID value, and a graph memory value to space_explorer.c through the space_hop() function.

The particular problem of information only being attainable at cost of the optimization goal prevents common shortest path solutions and algorithms such as Djikstra's algorithm, A* or any other variant which optimizes the shortest path when the graph structure is known or requires no cost to discover the structure.The requirements of the project means any information obtained needs to be utilized to its fullest extent, without regard to speed or performance by the algorithmic solution, unlike many graph traversal algorithms. My solution utilizes a mixture of probabilistic heuristics from the euclidean distance and node connection properties, and exploitation of graph probability distribution information deducable from the main code's PRNG logic. 
In a graph of 300 nodes, this algorithm averages 13 traversals to the destination node, the lowest number achieved in the competition.

# Graph structure exploitation
The start node is always at least 60% of the entire graphs size away from the destination node. As a result a common strategy was to split the algorithm into two stages, one where it randomly jumps until a certain distance is achieved, and then a main pathfinder to hone in on the destination node. My algorithm instead shortcuts this process by finding a way to gain every nodeID before its first traversal by exploiting the PRNG used to calculate them. An analysis of the graph structure was required for this solution. (Essentially, all native rand() functions are disabled for clients)


![map_generation_1](https://github.com/user-attachments/assets/50f64ce9-8e86-4a0b-a4d1-f91700843b21)

The grid size is determined by the grid_size variable, which is initialized to 1.0.
The number of cells along the x-axis and y-axis is calculated using the formula:
int num_cells_x = (int)ceil((xlim[1] - xlim[0]) / grid_size);
int num_cells_y = (int)ceil((ylim[1] - ylim[0]) / grid_size);
•  xlim[0] and xlim[1] are the limits for the x-axis, and similarly, ylim[0] and ylim[1] are the limits for the y-axis.
•  num_cells_x and num_cells_y represent the number of grid cells horizontally and vertically, respectively, based on these limits and the grid_size.
•	The average number of points per cell is based on the density and the area of each cell (grid_size * grid_size).
•	The expected number of points per cell is calculated as:
double expected_num_points = density * grid_size * grid_size;
•	Points are populated in a sequential order from left to right (increasing i index) and top to bottom (increasing j index). This is controlled by the following nested loops:
for (int i = 0; i < num_cells_x; i++) {
    for (int j = 0; j < num_cells_y; j++) {
•	This conditional ensures that generated points lie within the user-specified bounds.
if (x <= xlim[1] && y <= ylim[1]) {

Given the following values:





