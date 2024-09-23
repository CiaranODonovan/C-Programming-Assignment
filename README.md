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


![map_generation_values](https://github.com/user-attachments/assets/a9541c6b-e8a6-4a3e-ae55-edefa71b753a)



 
•	The grid is 10x10
•	The average number of points per cell is 3.
•	The total number of points averages 300.

Drand48(), which is a disabled function for the client (competitor submitted solution) is used to generate point values, preventing a direction straightforward simulation.
 However the seedPRNG(1) is a custom defined PRNG function with a hardcoded (1), which is responsible for storing the planet IDs mapped to points in an array,  is sequentially deterministic and remains the same values across all seed values that dictate the actual spatial X Y point values.



![prng_seed](https://github.com/user-attachments/assets/e901b69e-e002-402f-8a75-6b1db9559e85)



From here we can observe behaviour and patterns of planetIDs across any number of seeds and their statistical relationships with point values, as well as replicate the planet Id array, enabling us to move to any node in the graph before encountering it as a local connection during traversal. 


Destination node across a 1000 seed trials:



![destination_nodes](https://github.com/user-attachments/assets/2844a85d-8e2a-4ae2-b154-ecb61c2ca073)


•	The inverse pattern is true of the starting node (horizontally flipped & vertically)
•	Note: the outlier at point 0,0 occurs in any instance where the starting node begins in a position of X > 5, due to wrapping logic. Part of the solution is to check planet id 0 before moving on to the rest of the algorithm as this immediately solves a significant amount of seeds.

Chosen base case node (first node to visit after checking 0 outliers) over 1000 seed trials:



![base_case_node](https://github.com/user-attachments/assets/a99bc3bf-3910-4aa8-bfa8-2a3e9254ea83)



•	Rather than traversing from the initial starting node on the other side of the graph, or randomly jumping across the graph to get within a certain range, we can pick a node ID such as ID: 3675447994 (the graph above, which is chosen due to empirical performance in my solution) which is statistically likely to be close to the destination node. Due to the variance of node Y positions in each node distribution, we can simply iterate through the array for our initial traversals + or – some index value paired with a conditional Euclidean distance value to the destination node as a loop break that performs the best empirically to greatly reduce the search space from the alternative options before activating the main pathfinding algorithm.


#Main pathfinding algorithm
The main pathfinding algorithms arguably less complex than the initial graph structure exploitation, but still critical once a certain Euclidean distance is achieved, especially the denser the graph gets.  

The algorithm operates by storing a map struct that serves as the function’s memory of the graph between function calls to the server and back to the client. Inside the map struct are Known_Planet structs which represent nodes, and all available information about them. When a node is discovered, the id index is immediately stored as a connection to the currently occupied node. Each time a node is traversed to, we store its Euclidean distance and set a visited flag, so the distance value cannot be changed (as we know this value is 100% correct). Each of the connections then have their “mean_distance” updated as an averaged probability value of their true Euclidean distance to the destination node.  As we know connections are proximity based by a value of 1 unit, we know for absolute certain each connection falls within a [-1,+1] range of the currents node true distance, hence the mean distance is the same as the current nodes value. This -1 +1 range is a distribution of the maximum/minimum possible value of the true distance parameter, which can be updated as we traverse more nodes and discover more overlapping connections. 

For example, Node A is connected to both Node B and Node C. We have visited Node B and Node C but not Node A. Node A’s mean distance is updated based on the information we gathered from Node B & C which gives us a greater certainty of Node A's actual distance. Effectively, Node A’s mean_distance is updated, and the absolute maximum distance value +1 -1 range is reduced to a smaller range.
(E.G in some circumstance, we may be able to say Node A’s maximum possible distance value is -0.7, from multiple connection values we gathered, which is valuable in terms of risk assessment of neighbour choice. This is why it gets more important the denser and more connected the graph gets, de-prioritizing neighbours more likely to be in the wrong direction.)

A tuning parameter in the heuristic allows the algorithm to be adjusted to be more risk averse, or greedier in its approach. When testing very dense graphs, risk aversion becomes more important, however the default value the competition was run on is not very dense, so the algorithm prefers a small tuning parameter for a greedier approach, while still maintaining a small aversion to less probable neighbours in terms of certainty of the maximum range values.

The heuristic makes its ultimate choice based on the lowest calculated mean distance of all known, but non-visited nodes in combination with the risk assessment value which was empirically tested for the best performance. This implicitly backtracks from mistakes and moves the algorithm in the correct direction simply based on dynamically updated mean values of node connections per traversal. 









