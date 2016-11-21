# Breadth First Search

As discussed in backtracking chapter that backtracking can be viewed as depth first search (DFS)
over space solution tree, we can very well use breadth first search (BFS) to solve problems which
require exhaustive search over solution space.

BFS is a graph search technique and it expands all the steps one step away from initial state, then
expands all states two steps away from initial state, then three steps and so on until final state
is reached. Since it always expands all nodes at a current depth before expanding any nodes at a
greater depth, the first solution path found by BFS will be one with shortest steps.

Finding any solution or all solutions of a problem needing exhaustive search, can very well be
solved by DFS and BFS both. In such cases, in fact it is much better to use DFS instead of BFS. Some
of the reasons for this are explained later.

However as mentioned earlier, an important property which DFS doesn’t have but BFS has is we end up
finding shortest path in terms of edges to each node reachable from source.
So any problem needing exhaustive search over solution space and needs answer with minimum number of
steps can potentially be solved using BFS. BFS can find solution with minimum number of steps as
long cost of transitioning from one state(or node) to other is fixed.
BFS cannot find solution with minimum number of steps if different edges in graph can have different
cost.

BFS is an iterative algorithm implemented with help of queue.

Here’s general structure of BFS code which finds solution with minimum number of steps for a problem
needing exhaustive search over solution space.

    BFS(initialState, finalState) {
        q.push(initialState);
        visited[initialState] = true;

        while (!q.empty()) {
            curState = q.pop();

            if (curState == finalState) {
                // answer found.
                return true;
            }
            for (each possible move 'i' from curState) {
                compute nextState from curState using move 'i';

                if (nextState is valid and !visited[nextState]) {
                    visited[nextState] = true;
                    q.push(nextState);
                }
            }
        }
        // no answer found
        return false;
    }

## Comparison of BFS and DFS

As mentioned earlier, DFS should be used for finding all or any solution of problem and BFS should be
used if we need to find solution requiring minimum number of steps from initial state to final state.
The reason for this is amount of storage required.
In DFS, current state of search is completely represented by th path from root to current state
node, which requires space proportional to height of the tree.
In BFS, the queue stores all nodes at current level, which is proportional to the width of the
search tree. For many problems, the width of tree will grow exponentially in it’s height.

Iterative deepening depth first search (IDDFS)

Iterative deepening depth first search gives best of both worlds.

Iterative deepening allows us to simulate BFS with multiple runs of DFS at the cost of doubling the
running time, but in most cases – the extra cost is insignificant. The main idea is to add a depth
limit to the DFS and make the recursive function return immediately once the limit is reached. We
first run it with depth limit 0. This will only visit the starting vertex, S. Then we run it again
with depth limit 1. This time DFS will visit S and all of its immediate neighbours. We continue
this way, each time increasing the depth limit by 1 until we find the destination.

## Informed Search techniques

Most of the problems needing solution in minimum number of steps in general programming competition
will work with BFS also.

All DFS, BFS, IDDFS fall into “uninformed” search techniques where algorithm tries to go through
solution space but doesn’t know if exploraring a particular path will lead to solution or not.

In most practical applications however its too much work to explore using DFS/BFS/IDDFS and instead
“informed” search algorithms perform better. Informed search algorithms work by knowing exploring
which states will lead to solution and will explore only those parts. This prunes search space a lot
and can speed up execution considerably.

For implementing informed search, one needs to come up with heuristics specific to the problem in
hand and using that to decide which state to explore first.
Some of the well known informed search techniques are Hill Climbing, Beam Search, A*.

These are beyond scope of this course to cover and are mostly heavyweight for usage in programming
competition environment.