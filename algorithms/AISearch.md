---
title: Navigating the Maze - A Guide to AI Search Algorithms
description: Explore the world of AI Search Algorithms! From Uninformed methods like BFS and DFS to Informed strategies like A* and Greedy Best-First, and Local Search techniques like Hill Climbing and Genetic Algorithms, understand how AI finds optimal solutions.
date: 2023-03-03
draft: false
slug: /pensieve/ai-search
tags:
  - Algorithms
  - AI
---

Hey everyone, and welcome back to the blog! Artificial Intelligence is transforming countless fields, from robotics and game playing to natural language processing and complex problem-solving. At the heart of many AI systems lies a fundamental capability: the ability to **search** through a vast space of possibilities to find an optimal, or at least a good, solution.

Whether it's a robot planning a path, a game AI deciding its next move, or a language model trying to find the best translation, AI search algorithms provide the roadmap. Today, let's explore the different types of AI search algorithms, how they work, and where they shine. Understanding these can be crucial for anyone working on AI systems, even here in tech hubs like Bengaluru where AI innovation is booming!

## What are AI Search Algorithms? Finding the Path to a Solution

**AI search algorithms** are techniques used to systematically explore a problem's "search space"—the set of all possible states or solutions—to find a state that meets a specific goal or optimizes a certain criterion. The challenge often lies in the sheer size of this search space, making brute-force exploration impossible. AI search algorithms employ various strategies to navigate this space efficiently.

These algorithms are used in a wide range of applications, including robotics (path planning), game playing (finding optimal moves), natural language processing (parsing sentences, machine translation), operations research, and more.

## Types of AI Search Algorithms

AI search algorithms can be broadly classified into a few key categories based on how much information they use about the problem domain beyond the problem definition itself.

### 1. Uninformed Search Algorithms (Blind Search)
These algorithms operate without any domain-specific knowledge or heuristics to guide the search. They only know the problem definition: a set of states, a start state, a goal state (or a goal test function), and available actions/operators. They systematically explore the search space.

#### a. Breadth-First Search (BFS)
* **Concept:** BFS explores the search tree level by level. It visits all nodes at the present depth before moving on to the nodes at the next depth level. It typically uses a queue to manage the frontier (nodes to be explored).
* **Use Cases:** Finding the shortest path from a source node to a target node in an *unweighted* graph (where path length is measured by the number of edges). Checking for connectivity.
* **Properties:** Complete (guaranteed to find a solution if one exists), Optimal (finds the shallowest goal node). Time and space complexity are O(b<sup>d</sup>), where `b` is the branching factor and `d` is the depth of the solution.
* **Pseudocode (Conceptual):**
    ```pseudocode
    function BFS(graph, source, target):
        queue = new Queue()
        visited = new Set()

        queue.enqueue(source)
        visited.add(source)

        while queue is not empty:
            current_node = queue.dequeue()

            if current_node is target:
                return true // Path found

            for each neighbor of current_node in graph:
                if neighbor is not visited:
                    visited.add(neighbor)
                    queue.enqueue(neighbor)
       
        return false // Path not found
    ```

#### b. Depth-First Search (DFS)
* **Concept:** DFS explores as far as possible along each branch before backtracking. It dives deep into the search tree. It typically uses a stack (either explicitly or via recursion) to manage the frontier.
* **Use Cases:** Finding a path between two nodes, cycle detection in graphs, topological sorting, solving puzzles with a single solution path (like mazes). Your provided text mentioned it's used for "shortest path in a weighted graph" for DFS, which is incorrect; DFS doesn't guarantee shortest paths in weighted (or even unweighted) graphs. Dijkstra's or A* are for shortest paths in weighted graphs. DFS simply finds *a* path.
* **Properties:** Complete (if the search space is finite and no infinite loops), Not optimal (might find a suboptimal solution first). Time complexity O(b<sup>m</sup>) where `m` is max depth, or O(V+E) for graph traversal. Space complexity O(b*m) for explicit stack or O(m) for recursive stack.
* **Pseudocode (Conceptual):**
    ```pseudocode
    function DFS(graph, source, target):
        stack = new Stack()
        visited = new Set()

        stack.push(source)
        // Mark as visited when pushed, or when popped depending on implementation detail
        // For consistency with BFS above, let's mark when pushed.

        while stack is not empty:
            current_node = stack.pop()
           
            if current_node is visited and was marked_at_pop: // For mark-at-pop strategy
                continue

            visited.add(current_node) // If not marked_at_pop

            if current_node is target:
                return true // Path found

            // Push unvisited neighbors in reverse order to explore in standard order
            for each neighbor of current_node in graph (in reverse order of exploration preference):
                if neighbor is not visited:
                    stack.push(neighbor)
                    // If marking when pushed: visited.add(neighbor) here
       
        return false // Path not found
    ```

#### c. Uniform-Cost Search (UCS)
* **Concept:** UCS explores nodes in the order of their increasing path cost from the source node. It's essentially Dijkstra's algorithm without a heuristic. It uses a priority queue where priority is given by the path cost $g(n)$ (cost from start to node $n$).
* **Use Cases:** Finding the shortest path (least cost path) from a source node to a target node in a *weighted* graph where edge costs are non-negative.
* **Properties:** Complete, Optimal (guaranteed to find the least cost path). Time and space complexity can be O(b<sup>1 + floor(C*/ε)</sup>) where C* is cost of optimal solution and ε is min step cost, or O(E + V log V) with efficient priority queue.
* **Pseudocode (Conceptual):**
    ```pseudocode
    function UCS(graph, source, target):
        // Priority queue stores (node, path_cost_to_node)
        // Ordered by path_cost_to_node (ascending)
        priorityQueue = new PriorityQueue()
        visited_costs = new Map() // Stores node -> min_cost_found_so_far

        priorityQueue.enqueue((source, 0))
        visited_costs[source] = 0

        while priorityQueue is not empty:
            (current_node, current_cost) = priorityQueue.dequeue()

            // If we already found a shorter path to current_node, skip
            if current_cost > visited_costs[current_node] and current_node in visited_costs:
                 continue

            if current_node is target:
                return true // Path found with its cost current_cost

            for each neighbor, edge_cost_to_neighbor of current_node in graph:
                new_cost_to_neighbor = current_cost + edge_cost_to_neighbor
                if neighbor not in visited_costs or new_cost_to_neighbor < visited_costs[neighbor]:
                    visited_costs[neighbor] = new_cost_to_neighbor
                    priorityQueue.enqueue((neighbor, new_cost_to_neighbor))
       
        return false // Path not found
    ```

### 2. Informed Search Algorithms (Heuristic Search)
These algorithms use additional information about the problem, typically in the form of a **heuristic function** $h(n)$, to guide the search process more efficiently towards the goal. A heuristic function estimates the cost from a node $n$ to the goal.

#### a. A* Search
* **Concept:** A* search is a widely used informed search algorithm that aims to find the least-cost path from a start node to a goal node. It evaluates nodes by combining $g(n)$ (the cost from the start node to node $n$) and $h(n)$ (the heuristic estimate of the cost from $n$ to the goal). It prioritizes nodes with the lowest $f(n) = g(n) + h(n)$.
* **Use Cases:** Pathfinding in games, route planning in maps, robotics, parsing, many optimization problems.
* **Properties:** Complete and Optimal if the heuristic $h(n)$ is *admissible* (never overestimates the true cost to the goal) and, for graph search, *consistent* (monotonic).
* **Pseudocode (Conceptual):**
    ```pseudocode
    function AStarSearch(graph, source, target, heuristic_function):
        // Priority queue stores (node, f_cost = g_cost + h_cost)
        // Ordered by f_cost (ascending)
        priorityQueue = new PriorityQueue()
       
        g_costs = new Map() // Stores g_cost (path cost from source) for each node
       
        g_costs[source] = 0
        f_cost_source = g_costs[source] + heuristic_function(source, target)
        priorityQueue.enqueue((source, f_cost_source))

        // To reconstruct path, often store parent pointers: came_from = new Map()

        while priorityQueue is not empty:
            (current_node, current_f_cost_ignored_here) = priorityQueue.dequeue()
            // We use g_costs[current_node] as the true current path cost

            if current_node is target:
                return true // Path found (reconstruct path using came_from)

            for each neighbor, edge_cost_to_neighbor of current_node in graph:
                tentative_g_cost = g_costs[current_node] + edge_cost_to_neighbor
                if neighbor not in g_costs or tentative_g_cost < g_costs[neighbor]:
                    // came_from[neighbor] = current_node // For path reconstruction
                    g_costs[neighbor] = tentative_g_cost
                    f_cost_neighbor = tentative_g_cost + heuristic_function(neighbor, target)
                    priorityQueue.enqueue((neighbor, f_cost_neighbor))
       
        return false // Path not found
    ```

#### b. Greedy Best-First Search
* **Concept:** Greedy Best-First Search always expands the node that appears to be closest to the goal, as estimated by the heuristic function $h(n)$ alone. It doesn't consider the cost already incurred to reach the node ($g(n)$).
* **Use Cases:** Can be faster than A* if the heuristic is good, but it's not guaranteed to find the optimal path. Useful when finding *any* solution quickly is more important than finding the best one.
* **Properties:** Not complete (can get stuck in loops if not careful with visited sets), Not optimal.
* **Pseudocode (Conceptual):**
    ```pseudocode
    function GreedyBestFirstSearch(graph, source, target, heuristic_function):
        // Priority queue stores (node, h_cost)
        // Ordered by h_cost (ascending)
        priorityQueue = new PriorityQueue()
        visited = new Set()

        priorityQueue.enqueue((source, heuristic_function(source, target)))
        // Mark visited when enqueued or dequeued. For simplicity, let's assume we handle cycles.
       
        while priorityQueue is not empty:
            (current_node, current_h_cost_ignored) = priorityQueue.dequeue()

            if current_node in visited: // Avoid cycles or re-processing
                continue
            visited.add(current_node)

            if current_node is target:
                return true // Path found

            for each neighbor of current_node in graph:
                if neighbor not in visited: // Or not in priority queue with a better h_cost
                    priorityQueue.enqueue((neighbor, heuristic_function(neighbor, target)))
       
        return false // Path not found
    ```

#### c. Iterative Deepening A* Search (IDA\*)
* **Concept:** IDA* is a graph traversal algorithm that combines the depth-first search's space efficiency with A*'s heuristic guidance and optimality. It performs a series of depth-first searches with a progressively increasing cost limit (f-cost limit). In each iteration, it only explores paths whose total estimated cost $f(n) = g(n) + h(n)$ does not exceed the current limit.
* **Use Cases:** Problems where A* might run out of memory due to a large open set (priority queue), like solving puzzles (e.g., 15-puzzle) or pathfinding in very large graphs.
* **Properties:** Complete and Optimal (like A*, if $h(n)$ is admissible/consistent). Much better space complexity than A* (typically O(d) or O(bm) where d is depth).

### 3. Local Search Algorithms
These algorithms operate on a single current state (or a small set of states) and iteratively try to improve it by making local changes, rather than systematically exploring a search tree from a start state. They are often used for optimization problems where the path to the solution is not important.

#### a. Hill Climbing
* **Concept:** A simple local search algorithm that starts with an initial solution and continuously moves in the direction of increasing value (uphill) according to some evaluation function. It stops when it reaches a "peak" where no neighbor has a higher value.
* **Use Cases:** Optimization problems where a good-enough local optimum is acceptable.
* **Properties:** Not complete (can get stuck in local optima, plateaus, or ridges). Very simple and fast.
* **Pseudocode (Conceptual):**
    ```pseudocode
    function HillClimbing(problem): // problem object has initialSolution, getBestNeighbor, value_function
        current_solution = problem.initialSolution()
       
        loop:
            best_neighbor = problem.getBestNeighbor(current_solution)
            if problem.value_function(best_neighbor) <= problem.value_function(current_solution):
                // No better neighbor found, current_solution is a local optimum
                return current_solution
            current_solution = best_neighbor
    ```

#### b. Simulated Annealing
* **Concept:** A probabilistic local search algorithm inspired by the annealing process in metallurgy. It attempts to escape local optima by sometimes accepting moves that worsen the current solution. The probability of accepting a "worse" move decreases over time (as "temperature" cools).
* **Use Cases:** Global optimization problems like the Traveling Salesman Problem, circuit design.
* **Properties:** Can find globally optimal solutions if cooled slowly enough (probabilistically complete).
* **Pseudocode (Conceptual):**
    ```pseudocode
    function SimulatedAnnealing(problem): // problem has initialSolution, getRandomNeighbor, value_function, initialTemp, coolingSchedule
        current_solution = problem.initialSolution()
        temperature = problem.initialTemperature()

        while temperature > VERY_SMALL_VALUE: // Or for a fixed number of iterations
            random_neighbor = problem.getRandomNeighbor(current_solution)
           
            value_current = problem.value_function(current_solution)
            value_neighbor = problem.value_function(random_neighbor)
            delta_energy = value_neighbor - value_current // Assuming lower value is better

            if delta_energy < 0: // Neighbor is better
                current_solution = random_neighbor
            else: // Neighbor is worse, accept with a probability
                if random_float_0_to_1() < exponent(-delta_energy / temperature):
                    current_solution = random_neighbor
           
            temperature = problem.coolingSchedule(temperature) // Decrease temperature
           
        return current_solution
    ```

#### c. Genetic Algorithms
* **Concept:** A class of local search and optimization algorithms inspired by the process of natural selection and evolutionary biology. They maintain a "population" of candidate solutions and iteratively apply genetic operators like selection, crossover (recombination), and mutation to evolve better solutions over generations.
* **Use Cases:** Complex optimization and search problems, parameter tuning, scheduling, machine learning (e.g., evolving neural network architectures).
* **Properties:** Probabilistic, can find good solutions in large and complex search spaces, not guaranteed to find the global optimum.
* **Pseudocode (Conceptual):**
    ```pseudocode
    function GeneticAlgorithm(problem): // problem has initialPopulation, populationSize, selectParent, crossover, mutate, isSolution, fitness_function
        population = problem.initialPopulation()

        while not problem.isSolutionFound(population): // Or for a fixed number of generations
            new_population = new Array()
            for i = 1 to problem.populationSize:
                // Selection: Select two parents based on fitness
                parent1 = problem.selectParent(population)
                parent2 = problem.selectParent(population)
               
                // Crossover: Create a child by combining parts of parents
                child = problem.crossover(parent1, parent2)
               
                // Mutation: Introduce small random changes to the child
                child = problem.mutate(child)
               
                new_population.add(child)
           
            population = new_population // Replace old population with new
           
        return problem.getBestSolution(population)
    ```

## Core Properties of AI Search Algorithms

* They are designed to find an optimal or satisfactory solution to a problem by navigating a (potentially very large) space of possible solutions.
* Their applications are diverse, ranging from robotics and game AI to natural language processing and operations research.
* They can be broadly classified into Uninformed (blind), Informed (heuristic), and Local Search categories, each with distinct strategies and trade-offs.

## Applications Across Domains

AI search algorithms are fundamental to many intelligent systems:
* **Robotics:** Motion planning for robots, finding the optimal path for a robot to navigate obstacles and reach a target location.
* **Game Playing:** Powering AI opponents in games like chess, checkers, Go, and video games by searching for optimal moves in the game tree.
* **Natural Language Processing (NLP):** Used in parsing sentences (finding the most likely grammatical structure), machine translation (searching for the best translation of a sentence), and speech recognition.
* **Route Planning:** GPS navigation systems use algorithms like A* to find the shortest or fastest routes.
* **Logistics and Scheduling:** Optimizing delivery routes, job-shop scheduling, and other complex planning problems.

## Key Takeaways

* AI search algorithms are essential for problem-solving in AI, enabling systems to find solutions by exploring a space of possibilities.
* **Uninformed search** algorithms (BFS, DFS, UCS) systematically explore without domain-specific guidance.
* **Informed search** algorithms (A*, Greedy Best-First) use heuristics to guide the search more efficiently towards a goal.
* **Local search** algorithms (Hill Climbing, Simulated Annealing, Genetic Algorithms) iteratively improve a current solution by making local changes, often used for optimization.
* The choice of algorithm depends on the problem's characteristics, whether an optimal solution is required, available information (heuristics), and constraints on time and memory.

Understanding these diverse search strategies is key to building intelligent systems that can effectively navigate complexity and find solutions!
