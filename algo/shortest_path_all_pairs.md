# All Pairs Shortest Path

Dijkstra and Bellman-Ford algorithm are used for finding shortest path from a single source to all
remaining vertices.
All pairs shortest path(APSP) refers to problem where we want to find shortest path between all pairs of
vertices and not just from one source vertex.

# Naive implementation

A very simple way to solve this problem will be to invoke single source shortest path algorithms
(like Dijkstra, Bellman-Ford) multiple times from each of the vertex.

## Time Complexity of naive implementation

Time Complexity if implemented with Dijkstra’s shortest path ```O(V * (E log V))```
Time Complexity if implemented with Bellman-Ford ```O(V * (VE))```.

Although this may seem okay, note that number of edges ```E``` in a dense graph is proportional to ```O(V^2)```.

This will make algorithm complexity for dense graph ```O(V^3 log V)``` if implemented using Dijkstra’s
algorithm and ```O(V^4)``` if implemented using Bellman-Ford algorithm.

# Floyd-Warshall Algorithm

Similar to Bellman-Ford, Floyd-Warshall is a dynamic programming based algorithm.
Similar to Bellman-Ford and just like any other dynamic programming based algorithm, Floyd-Warshall
algorithm will build solution from earlier computed partial solutions.

## Key idea

This algorithm starts with allowing only vertex number 1 as intermediate node and finds shortest
path between all pairs of vertices ```(v1, v2)``` which go through this intermediate node.
In subsequent iteration it will allow vertices 1 and 2 to appear as intermediate nodes and
computes shortest path between all pairs of vertices ```(v1, v2)``` with either node 1 and 2 as
intermediate nodes and so on. So essentially in iteration k, algorithm essentially allows only
vertices ```v1, v2,..,vk``` which can be used as intermediate nodes.

This yields essentially a very simple algorithm

```
// Initialization
for (i = 1; i <= n; ++i) {
    for (j = 1; j <= n; ++j) {
        if (i == j) {
            dist[i][i] = 0;
        } else {
            if (a[i][j] != 0) {
                dist[i][j] = a[i][j];
            } else {
                dist[i][j] = INFINITY;
            }
        }
    }
}

for (k = 1; k <= n; ++k) {
    for (i = 1; i <= n; ++i) {
        for (j = 1; j <= n; ++j) {
            // Find path from 'i' to 'j' through 'k'

            // update shortest path if going through 'k' is more beneficial, else retain already
            found shortest paths
            dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
        }
    }
}
```

## Time Complexity

Time complexity of Floyd Warshall algorithm is ```O(n^3)```.

# Application of Floyd-Warshall algorithm

One other use case for Floyd-Warshall algorithm apart from APSP is finding transitive closure of a
binary relation i.e. finding if there exists a path from one node to other. This can be done by
maintaining only one bit information about if two nodes are connected or not in relation matrix and
doing Floyd-Warshall on it with a simple modification of instead of

```
dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);

```
we can do
```
dist[i][j] = dist[i][j] || (dist[i][k] && dist[k][j]);
```

## Detecting negative weight cycle

Similar to Bellman-Ford, Floyd-Warshall algorithm can detect if there is negative weight cycle in graph.
Negative weight cycle in graph exists if after running Floyd-Warshall algorithm dist[i][i] < 0 for
any vertex i.

# Johnson Algorithm

Floyd-Warshall works great for APSP for dense graph and in fact there is no known algorithm which
can solve APSP on dense graph with better than ```O(n^3)``` algorithm.
However there is possible to solve APSP for sparse graph in better way by Johnson’s algorithm.

# Key idea : Reweighing technique

Key idea behind this algorithm is to use multiple invocations of Dijkstra’s to solve APSP as seen in
naive way. But simply doing this will not work if graph has negative edges.
Johnson’s algorithm does a preprocessing on input graph so that simply doing ‘n’ invocations of
Dijkstra’s algorithm for APSP will work even in presence of negative edge weights in input graph.

A simple way to possibly change input graph so that all negative edge weights are converted to
positive is to add a constant to every edge so that no edge weight remains negative i.e. say if
input graph had edges with cost -1, -5, 10, 20, one can simply add 5 to each edge to change them to
4, 0, 15, 25 respectively. Unfortunately even though this is a clever trick, this doesn’t work as it is
(i.e. adding a constant number to every edge won’t work).

Consider a simple graph with 3 nodes

```
cost[s][v1] = 1;
cost[v1][d] = 1;
cost[s][d]  = 3;
```

Shortest path from s to d which is s->v1->d has cost of 2.
Now say if we add constant 4 to each edge then graph will change to

```
cost[s][v1] = 5;
cost[v1][d] = 5;
cost[s][d]  = 7;
```

Now the shortest path from s to d is actually s->d which has cost of 7.

So essentially *simply adding a constant number to each edge won’t preserve the shortest path*
This scheme fails because simply adding a constant to every edge modifies cost of different paths
differently. For e.g. if there is a path will 5 intermediate nodes, adding constant to every edge
will change cost of that path by 5 times. However in different path with only 2 intermediate nodes,
it will change by 2 times.

Johnson algorithm proposes a re-weighting technique which ensures that each path is adjusted by same
amount and hence it will preserve shortest paths in original graph.

Johnson’s algorithm modifies every edge from vertex ```v1``` to ```v2``` as

```
new_cost(v1, v2) = cost(v1, v2) + P(v1) - P(v2)
```

This ensures that all paths from v1 to v2 are shifted by exactly same amount ```P(v1) - P(v2)```

For e.g. consider path v1 -> v3 -> v2

```
// Cost of v1 -> v3
cost(v1, v3) = cost(v1, v3) + P(v1) - P(v3)

// Cost of v3 -> v2
cost(v3, v2) = cost(v3, v2) + P(v3) - P(v2)

Cost of v1 -> v3 -> v2
cost (v1, v3, v2) = cost (v1, v3) + P(v1) - P(v3) + cost(v3, v2) + P(v3) - P(v2)
                  = cost (v1, v3) + cost(v3, v2) + P(v1) - P(v2)
                  = originalCost + P(v1) - P(v2)
```

# Finding ```P(v)``` for each vertex

With the proof that above re-weighting preserves cost of each path, if we ensure for every edge
```(v1, v2)``` if adding ```P(v1) - P(v2)``` to it makes it non-negative then we can essentially transform
entire graph with negative edge weights to graph with positive edge weights but having same shortest
paths as of original input. With this transformation, we can easily use Dijkstra’s algorithm
multiple times to find shortest path from each vertex to every other vertex.
This works because re-weighting doesn’t change shortest path, simply adds ```P(v1) - P(v2)``` to its length.

As it turns out, we can easily compute ```P(v)``` satisfying above requirement simply by doing one
invocation of Bellman-Ford on original input graph.
To do this, add a dummy vertex node ```s``` in input graph such that ```s``` is connected to every node in
the graph with edge cost of 0.
Running Bellman-Ford on this graph with source as ```s``` will give length of shortest path to every
node ```v``` from this dummy node. As it turns out ```P(v)``` of vertex v is essentially length of
shortest path computed by Bellman-Ford from dummy node ```s```.

To see why this works and how it ensures that ensures adding ```P(v1) - P(v2)``` to edge ```(v1, v2)``` will
make it non-negative

* consider an edge ```(v1, v2)```.
* ```P(v1)``` is length of shortest path from dummy node ```s``` to ```v1``` and ```P(v2)``` is length shortest path
* from dummy node ```s``` to ```v2```.
* Now lets say path M is shortest path from ```s``` to ```v1``` which will have length ```P(v1)```.
* Now path ```M + (v1, v2)``` will have length ```P(v1) + cost(v1, v2)```
* Shortest path from s to ```v2``` is of length ```P(v2)```
* So, ```P(v2) <= P(v1) + cost(v1, v2)```
* So, ```0 <= cost(v1, v2) + P(v1) - P(v2)```
* This implies adding ```P(v1) - P(v2)``` to every edge will make it non-negative.

## Actual algorithm

Johnson algorithm is essentially

Modifying original graph ```g``` to ```g'``` by adding a dummy source node s and adding edge ```(s, v)```
for every vertex ```v```.
One invocation of Bellman-Ford algorithm with ```s``` as source vertex.
Using information in (1) to find ```P(v)``` for each ```v``` and doing re-weighting of every edge.
n invocations of Dijkstra’s algorithm to find all pairs shortest path
Readjusting computed shortest path for every pair ```(v1, v2)``` by readjusting to nullify effect due
to re-weighting in step (3).

## Time Complexity

Time Complexity will be ```O(V) + O(VE) + O(E) + O(VE log V) + O(V^2)``` so ```O(VE log V)```.
This is much better than Floyd Warshall for sparse graphs.
​