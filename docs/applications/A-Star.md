# Path finding algorithm A-Star
### Content:
- [Problem](#problem)
- [Solutions](#solutions)
	- [Breadth First Search](#breadth-first-search)
	- [Dijkstra's algorithm](#dijkstras-algorithm)
	- [Greedy Best-First-Search](#greedy-best-first-search)
	- [A-Star Algorithm](#a-star-algorithm)
- [A-Star Implementation](#a-star-implementation)
- [Comparison with Dijkstras](#comparison-with-dijkstras)

## Problem

The **graph** is one of the most significant structures in the algorithms, because this structure can represent many real life cases, starting from streets and ending by network.

And one is the most popular problem is:
:twisted_rightwards_arrows:**Find the least sum of graph edges for given start and end points** 

Generally we need determine input and output data:  
- **Input data:** graph map and end or start point/node (or both for certain path)
- **Output data:** paths (or intermediate points/nodes) with the least sum of graph edges as result

## Solutions
Today there are a variety of algorithms for solving this problem, and solutions have own advantages and disadvantages regarding the task, so let's consider main of them to find solution to the problem:

### Breadth First Search
This is the simplest algorithm for graph traversing. It starts at the tree root (it may be start/end node) and explores all of the neighbor nodes at the present depth prior to moving on to the nodes at the next depth level. 
| Origin Graph | Result Tree| Animation
|---|---|---|
|![](https://upload.wikimedia.org/wikipedia/commons/thumb/a/ad/MapGermanyGraph.svg/250px-MapGermanyGraph.svg.png)|![](https://upload.wikimedia.org/wikipedia/commons/thumb/6/63/GermanyBFS.svg/250px-GermanyBFS.svg.png)|![](https://upload.wikimedia.org/wikipedia/commons/4/46/Animated_BFS.gif)
|

Obviously this algorithm has low performance: $$O(|V| + |E|) = O(b^d)$$, where **b** is *branch factor* (average quantity of children nodes in tree, e.g. for binary tree $$b=2$$) and **d** is depth/distance from root.

---
### Dijkstra's algorithm
| Description | Animation|
|---|---|
|Dijkstra’s Algorithm (also called Uniform Cost Search) lets us prioritize which paths to explore. Instead of exploring all possible paths equally (like in [Breadth First Search](#breadth-first-search)), it favors lower cost paths.|_________________________![](https://upload.wikimedia.org/wikipedia/commons/2/23/Dijkstras_progress_animation.gif)
---
### Greedy Best-First-Search
With Breadth First Search and Dijkstra’s Algorithm, the frontier expands in all directions. This is a reasonable choice if you’re trying to find a path to all locations or to many locations. However, a common case is to find a path to only one location. 
Let’s make the frontier expand towards the goal more than it expands in other directions. First, we’ll define a **heuristic function** that tells us how close we are to the goal. E.g. on flat map we can use function like $$H(A, B) = |A.x - B.x| + |A.y - B.y|$$ , where **A** and **B**  are nodes with coordinates **{x, y}**.
Let's consider not only shortest edges, but also use the estimated distance to the goal for the priority queue ordering. The location closest to the goal will be explored first.

| Result of Heuristic function | Animation |
|---|---|
| We can see that firstly nodes, that are closer to target are considered at first. But when algorithm finds a barrier, then it tries to find the path to walk around, but this path is best from the corner, not from the start position, so the result path is not the shortest. This is a result of the *heuristic function*. To solve this problem let's consider next algorithm |_________________________![_________________________](https://upload.wikimedia.org/wikipedia/commons/8/85/Weighted_A_star_with_eps_5.gif)

---
### A-Star Algorithm
Dijkstra’s Algorithm works well to find the shortest path, but it wastes time exploring in directions that aren’t promising. Greedy Best First Search explores in promising directions but it may not find the shortest path. The A* algorithm uses _both_ the actual distance from the start and the estimated distance to the goal.
| Result of Cost and Heuristic function | Animation |
|---|---|
| Because of considering both **cost** and result of **heuristic functuion** as result metric for  Dijkstra’s algorithm, we can find the shortest path faster, than raw Dijkstra’s algorithm, and precisely, than Greedy Best-First-Search |_________________________![](https://upload.wikimedia.org/wikipedia/commons/5/5d/Astar_progress_animation.gif)|

## A-Star Implementation
let's take a closer look at this algorithm and analyze it with code example. First of all You need to create a *Priority Queue* because you should consider points, which are closer to destination from start position. *Priority does not equal cost*. This Queue contains possible *points*, that are to be considered as possible shortest way to destination.
```python
# Only main methods
class PriorityQueue:
	# Puts item in collection, sorted by priority.
	def put(self, item, priority):
	# Returns the most priority item.
	def get(self):
```
Also you  need a class, that describes your Graph Model with 2 methods. First finds neighbors of current node, and second returns cost between current node and next. This methods allows to implement any structure, be neither grid, hexagonal map or graph.

```python
# Only main methods
class SquareGrid:
	# Returns neigbours of 'id' cell
	# according to map and 'walls'.
	def neighbors(self, id):
	# Returns cost (or distance) between 2 cells.
	# Applicable for neighbors only.
	def cost(self, current, next):
```
Also you need to add your heuristic function too. Because, e.i. on a grid, a cost is always equals to 1 (if you don't use diagonals), so it would be like a Breadth First Search, but you know a destination point, so you can use direction.
```python
def heuristic(a, b):
	(x1, y1) = a
	(x2, y2) = b
	return ((x1 - x2)**2 + (y1 - y2)**2)**.5
```
Now we can implement our A-Star algorithm. First of all we need to init our algorithm: **frontier** stores points according to priority. We will store information in dictionaries:
- **came_from** like pair <TO point \: FROM point>
- **cost_so_far** like pair <Point \: Distance from start>

Firstly add our start point to them. Than, for each point (temporary as *origin* we find its neighbors and for each calculate the cost as: cost from *origin* to neighbor. If there is no information about this node in Queue or the cost is less than before, that add this point to queue with priority = cost + heuristic. Last step allows to consider more closer point to destination at first.
```python
def  a_star_search(graph, start, goal):
	## Create a queue and add start point
	frontier = PriorityQueue()
	frontier.put(start,  0)
	# Dictionaries with init for start point
	came_from = {}
	cost_so_far = {}
	came_from[start] = None
	cost_so_far[start] = 0
	# Not all neighbors are visited
	while  not frontier.empty():
		# Get next node (firstly it is start one) 
		current = frontier.get()
		if current == goal:
			break
		# Find all neighbor nodes
		for  next  in graph.neighbors(current):
			new_cost = cost_so_far[current] + graph.cost(current,  next)
			# Not visited or cost to it is less
			if  next  not  in cost_so_far or new_cost < cost_so_far[next]:
				cost_so_far[next] = new_cost
				priority = new_cost + heuristic(goal,  next)
				frontier.put(next, priority)
				came_from[next] = current
	return came_from, cost_so_far
	
```

## Comparison with Dijkstras
Now let's try to compare Dijkstras algorithm with A-Star. For this task we will generate map with size from 5 to 50 with step equal 3. Start position is in left top corner, and End position is opposite. Also, we will generate corners (the quantity is SIZE^0.4), with random length for one side and other side to the end of the map. Generated Maps you can find below, there is only example and comparison plot of iterations depending on the map size.
| Dijkstras | A-Star | Result |
|---|---|---|
|![Example](https://lh3.googleusercontent.com/LXZ8lb6CbuhHM2BeGlTKP3tN7RnBlAW3URpVD1DFnMDZke4xbRk-oHOIUebZtnu0-lllgPzgLoI)|![Example](https://lh3.googleusercontent.com/nMB_FmMktLemUT1gEz-BjROu1m1CWqAkeE5x3_tHCGA3QFL6Nyy_hH-2mjTBCX9kCubjged1qB0)|![Comparison](https://lh3.googleusercontent.com/00dSiGJAx94MuxI13qg6oJNR9Ry27HDDbBctiNgikVKZzeqLj5nOqdyVINBO1vOmHZ8fV_tFnbU)|
|Due to the fact that the algorithm does not know the final position, it considers all possible directions, including dead ends, which affects the number of iterations|Due to the fact that the algorithm knows the final position, it first considers those points that are closest to the target, so in some cases it does not go into false dead ends| It is seen that in most cases A-Star finds faster. However, there are situations where heuristics do not help, and in this case A-Star works the same way as Dijkstra|
### Thanks for reading :wink:
Full work code and comparison of algorithms you can find below:
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg#button)](https://colab.research.google.com/drive/11ZYsuAhJM5yRrFFgacdKWmaz5XF_cgUo) 
