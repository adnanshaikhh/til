# Backtracking
Many of the hard problems needs exhaustive search over all possible solution space.

The solutions to such problems is complicated and consists of many small parts. So we will try all
possibilities for picking the first part. For each of these, we will try all possibilities of
picking the second part. And so on until we finally build a complete solution. Or perhaps we are
looking for the best solution. Then we will generate all solutions part by part and remember the
solution we find.

Backtracking is a systematic way to go through all possible solutions of a problem. Backtracking
solution can be considered as state transition diagram where from current state, we go to next state
and so on and until we reach final state.

During course of backtracking we must avoid repetition and missing out any possible state.
Backtracking constructs a tree of partial solutions, where each vertex is a partial solution.
There is an edge from ```x``` to ```y``` if node ```y``` was created by advancing from ```x```.
This tree of partial solution provides an alternative way to think about about backtracking, for
the process of constructing the solutions corresponds exactly to doing a depth-first traversal of
backtrack tree.

Viewing backtracking as depth-first search(DFS) yields a natural recursive implementation
of basic algorithm as follows.
```
backtrack(curState, finalState)
{
    if (curState == finalState) {

        found answer;
        return true;
        /* To find all possible solutions,
        * just fool parent by returning false
        * instead of true.
        */
    }

    for (each possible choice 'i') {

        compute nextState based on choice i;

        if (nextState is valid && not already visited)  {

            mark nextState as visited;

            if (backTrack(nextState, finalState) == true) {

                // Found solution, indicate to parent about sucess
                return true;
            }

            if (AnyStateModified) {
                revert state to original state;
            }
        }
    }
    /* None of the choices from this state worked,
    * indicate parent about failure.
    */

    retun false;
}
```
With this template, for every backtracking problem simply try to answer following questions and then
you should be able to simply plugin those details in above template and get the working code.

What is initial and final state. How do you know you’ve reached final solution?
What all choices we’ve for every state to go to next state. For e.g. when solving maze, at each
point, we can go in any four directions so we get 4 choices.
Out of all possible choices, what are VALID choices at any point of time. For e.g. in maze, even
though you’ve 4 possible choices, some of them may not be valid since they might lead you to go
out of maze or path may be blocked for a particular choice.
How will you ensure you don’t get into same state again. For maze problem, simply keeping
tracking of visited array is sufficient. For other problems this may not be very trivial to
understand.

## Complexity analysis

Backtracking solutions will result in exponential time complexity.
Since backtracking is a traversal over solution space which can be considered as a tree,
complexity is proportional to number of nodes in tree.
If you’ve ```‘n’``` choices at every point and your tree can be of height ```‘m’``` then complexity will be
```O (n ^ m)```.
For e.g. if your solution space tree is full binary tree then complexity will be ```O(2 ^ n)```.