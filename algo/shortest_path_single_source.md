# Single Source Shortest Path

The problem of finding length of shortest path in unweighted graph between 2 nodes source ```s``` and
destination ```d```, can be solved using BFS.
This can be done by maintaining an array dist where ```dist[v]``` stores shortest distance from ```s``` to
```v```. During BFS traversal, as new vertex is explored, set it’s distance as parent’s distance + 1.

## Shortest path in weighted graph

BFS does not find shortest path in weighted graph. This happens because BFS as such doesn’t consider
edge weights during traversal.
One easy way to make BFS find shortest path in weighted graph is to replace an edge of cost ```c``` with
```c``` edges incident on dummy intermediate nodes. Each edge then will have weight 1. After this 
transformation of graph, one can easily run BFS to find shortest path.
Although this approach theoretically works, it’s not practical as it can very easily bloat graph
with many dummy vertices and edges.

# Dijkstra’s Algorithm

Core idea behind Dijkstra’s algorithm is similar to a graph search algorithm. A search algorithm
can be thought of as starting at some source vertex in a graph, and “search” the graph by walking
along the edges and marking the vertices.

Dijkstra’s algorithm proceeds with graph search with a set containing only source node s say set
```X```. At each iteration a new node from set ```V - X``` (V being set of all nodes) is added to set ```X```.
To pick up node to explore next from set ```V - X```, Dijkstra’s algorithm considers all edges crossing
from set ```X``` to set ```V - X``` and picks edge ```(v1, v2)``` with minimum cost where ```v1``` is in set ```X```
while ```v2``` is in set ```V - X```. Once vertex ```v2``` is selected to be added to set ```X```, algorithm will
evaluate if vertices adjacent to ```v2``` can be reached via ```v2``` in a shorter path and if so, update
this information.

At implementation level, essentially, Dijkstra’s algorithm maintains a distance array dist which
maintains shortest path to each vertex ```v``` from source ```s```.

It is a greedy approach where in each step we choose a vertex ```v``` with minimum distance from source
and try to see if distance of it’s each adjacent vertex ```i``` can be reduced by going through path
```s->...->v->i```.
Primary idea behind this is since ```v``` was the closest vertex (to the source), any other path to ```v```
that we might discover during subsequent iterations must be longer than ```dist[v]```.

Here’s pseudo code

```
for (i = 0; i < n; ++i) {
    dist[i] = INFINITY;
    visited[i] = false;
}
dist[s] = 0;

while (1) {
    min = INFINITY;

    // find vertex with min dist
    for (i = 0; i < n; ++i) {
        if (min < dist[i] && !visited[i]) {
            min = dist[i];
            v = i;
        }
    }
    if (min == INFINITY) {
        // no more vertex to process, stop
        break;
    }
    visited[v] = true;
    for (each vertex 'i' adjacent to 'v') {
        if (dist[i] > dist[v] + edge_cost(v, i)) {
            // node 'i' can reached in shorter path if we go through
            // node 'v', so update dist[i] with cost of newly found short path
            dist[i] = dist[v] + edge_cost(v, i);
        }
    }
}
```

## Time Complexity of naive implementation

Time Complexity of above algorithm is ```O(VE)``` where ```V``` denotes number of vertices in graph and ```E```
represents number of edges.
Reasoning is in every iteration we’re finding a vertex with minimum distance and then traversing all
adjacent vertices to it.

## Efficient implementation of Dijkstra’s Algorithm

In above implementation, one of the time consuming task is to find minimum in dist array in every
iteration. This can be improved by making use of good data structure suitable for this task which is
min heap (aka priority queue).

With this, Dijkstra’s algorithm can be considered as generalization of BFS. In BFS, we make sure a
node at front of the queue is closet to source in terms of edges. So if we replace queue in BFS with
priority_queue and we make sure node at top of priority_queue is closet to source in terms of
distance, then we’re done and essentially can reuse BFS code structure with some changes.

One thing that needs to be taken care of is when we choose a vertex say ```v1``` closet to source
vertex say ```s```, we need to update distance of it’s adjacent vertex say ```v2``` if going from ```s``` to
```v2``` is cheaper via ```v1```.
This will need updating ```dist[v2]``` in heap. This update operation is not implemented in most of the
standard heap implementations. So one needs to implement own heap to achieve this.
One other idea to overcome above issue is not updating existing ```dist[v2]``` entry but instead adding
new ```dist[v2]``` entry as a new element in heap. This can work in this context because we can safely
ignore previous entries in the heap as update of element happens only when its value is reduced so
pop operation will always pop most recent value which is lowest value corresponding to ```dist[v2]```.

With above trick, STL has implementation of priority_queue which you can readily use to make your
implementation efficient.

## Time Complexity of efficient implementation

Time Complexity of above algorithm is ```O(E log V)``` where ```V``` denotes number of vertices in graph and ```E```
represents number of edges.
Reasoning is priority_queue maintained for storing distances will have at most ```V``` elements in it.
deleteMin() operation on heap with ```V``` elements will ```O(log V)```.
Furthermore even though algorithm is implemented with vertex centric view (i.e. for each vertex do
some processing), algorithm essentially is processing each edge graph only once and during that
processing can trigger a priority_queue operation to either delete or add an entry in priority_queue.
So Complexity is ```O(E log V)```.

## *NOTE*

STL priority_queue implementation is a max-heap so to make it work as minimum heap, you need
to store weight as negative. Better and proper implementation is to use a function object / functor
and writing your own comparator which makes it behave as min-heap. Refer to priority_queue class
details for constructor which allows specifying custom functor.

# Bellman Ford Algorithm

One of the disadvantage of Dijkstra’s algorithm is it can only work with graph having non-negative
weight. Remember intuition behind Dijkstra’s algorithm is if in current iteration, ```v``` is the
closet vertex to source then any other path to ```v``` that we might discover during subsequent
iterations must be longer than ```dist[v]```. This assumption is not true when edge weight can be negative
since subsequent iteration might discover shorter path by going over an edge with negative length.

Dijkstra’s algorithm is a generalization of the BFS algorithm – meaning that Dijkstra’s is itself a
graph search algorithm. These search algorithms do not make use of the fact that we already know
beforehand the entire structure of the graph. This explains why Dijkstra’s algorithm cannot handle
negative weights – it can only search from what we have seen so far, and does not expect new
“discoveries” at some later stage would affect what we have already processed.
The Bellman Ford algorithm is a Dynamic Programming algorithm that solves the shortest path problem.
It looks at the structure of the graph, and iteratively generates a better solution from a previous
one, until it reaches the best solution.

The idea is to start with a base case solution ```S0```, a set containing the shortest distances from ```s``` to
all vertices, using no edge at all. In the base case, ```d[s] = 0```, and ```d[v] = INFINITY``` for all other
vertices ```v```. We then proceed to update distance of vertex ```v``` considering all edges incident on ```v```,
building the set ```S1```. This new set is an improvement over ```S0```, because it contains all the shortest
distances using one edge – i.e. ```d[v]``` is minimal in ```S1``` if the shortest path from ```s``` to ```v``` uses one edge.
Now, we repeat this process iteratively, building ```S2``` from ```S1```, then ```S3``` from ```S2```, and so on… Each set
```Sk``` contains all the shortest distances from ```s``` using ```k``` edges – i.e. ```d[v]``` is minimal in ```Sk``` if the
shortest path from ```s``` to ```v``` uses at most ```k``` edges.

Since shortest path in a graph of ```n``` nodes will have at most ```n - 1``` edges, repeating above procedure
of ```n - 1``` times ensures, we’ll find shortest path to each vertex.

Here’s pseudo code

```
for (i = 0; i < n; ++i) {
    dist[i] = INFINITY;
    visited[i] = false;
}
dist[s] = 0;

for (i = 0; i < n - 1; ++) {
    for (each edge (u,v) in graph) {
        if (dist[u] < INFINITY && dist[v] > dist[u] + edge_cost(u, v)) {
            dist[v] = dist[u] + edge_cost(u, v);
        }
    }
}
```

## Time Complexity of Bellman-Ford algorithm

Time Complexity of Bellman-Ford is easy to analyze and apparent from above pseudo-code which is
```O(VE)```.

## Graphs with negative weight cycle

A graph doesn’t have shortest path if it has negative weight cycle (cycle in graph where total weight
of edges participating in cycle is negative), since one can go over that cyclic path as many times
as one wants to find path shorter than earlier found.

Bellman ford algorithm can be used to detect if graph has a negative weight cycle. It’s very simple
run Bellman Ford once. Then run one more iteration of part of Bellman Ford algorithm which updates
dist array, if we can update distance of any vertex, then graph has a negative weight cycle.