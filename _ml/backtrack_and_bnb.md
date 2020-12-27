---
title: "Backtracking and Branch-And-Bound"
excerpt: " "
classes: wide
header:
  teaser: assets/images/ml/bnb.png
---

## Introduction

In this post, we'll discuss two search algorithms, backtracking, and branch and bound.  Both of these algorithms are useful when solving constraint satisfaction problems, where the solution is naturally built up one candidate at a time, and all possible solutions need to be enumerated.

## Background
Lets start with an example to motivate the need for the backtracking, and the branch and bound algorithm.  Suppose that we need an algorithm to make change, given a certain set of coins that we have available.  For example, suppose we need to make 28 cents of change, using only dimes (10c), nickles (5c), and pennies (1c).  Without any further specifications, there are several solutions to this problem.  2 dimes, 1 nickle, and 3 pennies; or, 2 dimes and 8 pennies, etc all do the trick.  A formal description of how we solved this problem might go as follows: first, we recognize that the solution consists of several positions to be filled (in this case, each position consists of a coin choice), and that we need to fill these positions with candidates such that the constraints of the problem are satisfied (we don't want to provide less than, or greather than 28c of change).  

There are several characteristics to this type of problem that are important to be recognized:
  1. The solution consists of a series of "slots" that need to be filled.
  2. At each slot, there is a set of potential candidates.
  3. We can evaluate whether a sequence of filled candidates is a partial and/or full solution to the problem.

If an algorithmic search problem contains these characteristics, then backtracking and/or branch and bound might be a good candidate to solve the problem.

## Backtracking
Backtracking is an algorithmic approach to solving problems with the given characteristics above.  At a high level, backtracking builds up a solution, element by element, from a set of candidates.  The algorithm explores all permutations of candidates and returns the first solution it finds that satisfies all specified constraints.  For problems like this, we can imagine the space of solutions that are built up by candidates as paths in a graph.  Each node of the graph represents a choice of a candidate to fill that particular slot of the solution, and a path which satisfies all constraints of the problem represents a solution. Depending on the nature of the problem to be solved, there could be more than one possible solution.  Backtracking is guaranteed to find a solution, if it exists, although it does not necessarily find the optimal solution.  We discuss optimality later in this post in the [Branch And Bound](#branch-and-bound) section.

Backtracking is analogous to depth first search (DFS) of a graph.  The primary difference to keep in mind is that typically, when doing a DFS, we already have a graph that we want to search.  In backtracking, we are not given the graph a-priori, but build it up implicitly as we search the space of possible solutions.  The close similarity between a DFS and backtracking is evident if one examines their pseudocodes, side-by-side.

```python
def dfs(G, v):                         ; def backtrack(A, k):
                                       ;     if is_a_full_solution(A, k): return A
                                       ;     if not is_a_partial_solution(A, k): return
    v.discovered = True                ;     k += 1
    for neighbor in v.neighbors():     ;     for c in get_candidates(A, k):
        if not neighbor.discovered:    ;         A[k] = c
            dfs(G, neighbor)           ;         backtrack(A, k)
```

First, lets discuss the backtracking algorithm.  We initialize the search with an empty solution vector, $$A$$, and index, $$k$$ which represents the index upto which $$A$$ is valid.  We begin by checking if the current solution vector upto index $$k$$ is a full solution, and if so, return it.  Otherwise, we get all the candidates for the current slot, $$k$$, and iterate over them.  For each selected candidate, we determine if it is a partial solution and recursively continue to add to it, or we move on to the next candidate.  

The pseudocode shows that DFS is almost exactly the same.  In DFS, adjacent nodes are analogous to candidates in backtracking.  Both algorithms iterate over the candidates/neighbors.  In DFS, we only go to the next level of a tree if the node has not yet been discovered.  In backtracking, we only continue to the next level of the graph if the candidate is a partial solution to the problem we are trying to solve.  We should also note that by only continuing down a path in the solution graph if it is a partial solution, we efficiently do not explore unnecessary paths.  

## Branch And Bound
Branch and bound is an extension of backtracking, which adds the notion of optimality.  It allows the user to specify a notion of "goodness", which it can optimize for when searching for solutions. Instead of stopping when a solution is found, it continues to search the space for a more optimal solution.  Pseudocode for adding branch and bound to backtracking looks as follows:

```python
def backtrack_bnb(A, k):
    k += 1
    for c in get_candidates(A, k):
        A[k] = c
        cur_soln_goodness = goodness(A, k)
        if is_a_partial_solution(A, k):
            if goodness(A, k) better_than goodness(best_soln) and is_a_full_solution(A, k):
                best_soln = copy(A, 0, k)
            backtrack_bnb(A, k)
    backtrack_bnb(A, k-1)
```

As in plain backtracking, backtracking with branch and bound begins by initializing the search with an empty solution vector, $$A$$, and index, $$k$$, which represents the index upto which $$A$$ is valid.  We iterate through each of the candidates for each step in the tree.  For each candidate, we check whether it is a solution.  If it is, we compute a notion of goodness of that solution.  Goodness is a user defined function that computes a metric of goodness of the current solution.  This, along with a notion of how to order the metric of goodness (for example, minimize, or maximize, etc...) lets the algorithm decide which is the "best" solution.  The pseudocode above is eeirly close to the pseudocode for plain backtracking.  The two main differences to keep in mind are:
  1. We compare our current solution to the best solution, and update our best solution if the current one is better.
  2. We do not stop after finding a solution, but continue searching the tree.  This is given by the last line in the pseudocode, where we go up one level in the tree, and continue our search.

## Implementation

### Backtracking
A full working implementation of backtracking is provided here, and discussed below.

```python
from typing import Callable

# maximum possible size of a solution.
MAX_CANDIDATES = 100


def backtrack(construct_candidates: Callable, is_a_solution: Callable, max_soln_sz):
    """
    A backtracking engine - useful for searching a space of solutions in an efficient way.
      Based heavily on this excellent tutorial:
      https://cs.lmu.edu/~ray/notes/backtracking/
    Generalized slightly with inspiration from Skiena's Algorithm Design Manual
    :param construct_candidates: a callable function which takes 2 keyword arguments as follows:
        construct_candidates(solution_vector=solution, valid_idx=ii):
            solution_vector: a 1-D list/numpy-array of the solution
            valid_idx: the index upto which the solution is valid
        This function constructs the next possible set of candidates, given the current
        partial solution
    :param is_a_solution: a callable function which takes 2 keyword arguments as follows:
        is_a_solution(solution_vector=solution, valid_idx=ii)
            solution_vector: a 1-D list/numpy-array of the solution
            valid_idx: the index upto which the solution is valid
        This function returns a boolean of whether the solution upto the valid_idx is a
        partial or full solution
    :param max_soln_sz: the maximum size of an acceptable solution
    :return: a found solution
    """
    max_soln_sz = min(max_soln_sz, MAX_CANDIDATES)
    solution = [None] * max_soln_sz

    def find_next(ii):
        """
        Finds a compatible solution in the solution vector for index ii
        :param ii: the index for which to find a solution
        """
        candidates = construct_candidates(solution_vector=solution, valid_idx=ii)
        for c in candidates:
            solution[ii] = c
            # if this candidate is a valid solution, then ...
            if is_a_solution(solution_vector=solution, valid_idx=ii):
                # Here, we return solution if the solution has reached the maximum allowable
                # size.  Otherwise, we look for a solution in the next index, and find_next(ii+1)
                # evaluates to True if something other than None is returned.  But since that is a
                # recursive call, we effectively build up our solution, if a valid solution
                # exists, until we reach the maximum solution size.  Otherwise, we return None (L48)
                if ii >= max_soln_sz-1 or find_next(ii + 1):
                    return solution
        return None

    final_solution = find_next(0)
    # remove None from the solution, so that we can allow for shorter than max-solution-size
    #  solutions
    final_solution = [x for x in final_solution if x is not None]
    return final_solution
```

We see above a Python implementation of a general backtracking engine.  There are three inputs that are described in detail by the docstring.  At a high level, the `construct_candidates` takes the current solution, and determines the next set of candidates that should be searched by the backtracking engine.  In the case of the the coin changing example, this function returns all the coin denominations which are available to be used for making change (10c, 5c, 1c).  The `is_a_solution` takes a current candidate solution vector, and determines whether the candidate solution vector meets the constraints of the problem.  Relating again to the coin changing problem, if a candidate solution vector was $$[10, 10, 5, 1, 1, 1]$$, this function would return True, whereas if a candidate solution vector was $$[10, 10, 5, 1, 1, 5]$$, this function would return False.  Finally, the `max_soln_sz` lets the algorithm know how large the solution vector can be.  Again, in the coin changing problem, if `max_soln_sz` was $$3$$, there would be no solutions found, where as if the `max_soln_sz` was $$6$$, a solution does exist.  If the max solution size was $$30$$, then depending on the order in which candidates are returned by `construct_candidates`, the potential first solution found and returned by the function would be a sequence of 28 1's ($$[1, 1, 1, ...]$$).

The actual algorithm implementation first defines a solution list initialized to None, and a function `find_next(ii)`, which is supposed to find the next valid candidate for a solution.  It works by constructing all the candidates, given the current solution vector, and iterating over them in a for-loop.  If the candidate is a partial solution (checked by `is_a_solution`), then we store it, and recursively call `find_next(ii)`.  Notice that because the call to `find_next(ii)` is nestled inside the for-loop, this amounts to a depth-first-search. If a candidate doesn't work for the current solution vector, it is replaced ,`solution[ii] = c`, and the next candidate is tested.  This process is repeated until a solution is found, or all solutions are exhaustively searched.  Finally, to allow for solutions which are potentially shorter than the configured `max_soln_sz`, None values are removed from the final solution, and this list is returned.

### Backtracking with Branch and Bound
A full working implementation of backtracking with branch and bound is provided here, and discussed below.

```python
from typing import Callable, Union
import math
import copy

# maximum possible size of a solution.
MAX_CANDIDATES = 100


def goodness_min(cur_best, new_metric):
    """
    Determins if the new_metric is less than the current best, and if so,
    returns True.  Otherwise, returns False
    :param cur_best: the current best metric
    :param new_metric: the metric of the new solution
    """
    if new_metric < cur_best:
        return True
    else:
        return False


def goodness_max(cur_best, new_metric):
    """
    Determins if the new_metric is greater than the current best, and if so,
    returns True.  Otherwise, returns False
    :param cur_best: the current best metric
    :param new_metric: the metric of the new solution
    """
    if new_metric > cur_best:
        return True
    else:
        return False


def backtrack_bnb(construct_candidates: Callable,
                  is_a_partial_solution: Callable,
                  is_a_full_solution: Callable,
                  goodness_fn: Callable,
                  max_soln_sz,
                  max_solutions: int = 100,
                  goodness_fn_comparator: Union[Callable, str] = goodness_min,
                  best_metric_start=math.inf):
    """
    A backtracking engine with branch-and-bound - useful for searching a space of solutions
    in an efficient way, and eliminating solutions based on a goodness metric.
    Based heavily on this excellent tutorial:
      https://cs.lmu.edu/~ray/notes/backtracking/
    :param construct_candidates: a callable function which takes 2 keyword arguments as follows:
        construct_candidates(solution_vector=solution, valid_idx=ii):
            solution_vector: a 1-D list/numpy-array of the solution
            valid_idx: the index upto which the solution is valid
        This function constructs the next possible set of candidates, given the current
        partial solution
    :param is_a_partial_solution: a callable function which takes 2 keyword arguments as follows:
        is_a_partial_solution(solution_vector=solution, valid_idx=ii)
            solution_vector: a 1-D list/numpy-array of the solution
            valid_idx: the index upto which the solution is valid
        This function returns a boolean of whether the solution upto the valid_idx is a
        partial solution
    :param is_a_full_solution: a callable function which takes 2 keyword arguments as follows:
        is_a_full_solution(solution_vector=solution, valid_idx=ii)
            solution_vector: a 1-D list/numpy-array of the solution
            valid_idx: the index upto which the solution is valid
        This function returns a boolean of whether the solution upto the valid_idx is a
        full solution
    :param goodness_fn: a callable function which takes 2 keyword arguments as follows:
        goodness_fn(solution_vector=solution, valid_idx=ii)
            solution_vector: a 1-D list/numpy-array of the solution
            valid_idx: the index upto which the solution is valid
        This function computes a metric on how "good" the solution is.
    :param max_soln_sz: The maximum size of a solution
    :param max_solutions: The maximum number of solutions to search before declaring
        the current best as optimal
    :param goodness_fn_comparator: A callable which takes 2 inputs as follows:
        goodness_fn_comparator(cur_best, new_metric)
            cur_best - the value of the current best solution, according to a metric returned by goodness_fn
            new_metric - the value of the goodness_fn on the current solution being evaluated
        Returns True if new_metric is *strictly* better than cur_best, otherwise False.  This is used
            to determine if we continue down a certain branch, or move to the next branch
        This can also be a string 'min' or 'max', if it is desired to either minimize or maximize
        the goodness metric
    :param best_metric_start: the initial value of the best metric. Automatically set if
        goodness_fn_comparator is either 'min' or 'max'.  Otherwise, this value will influence
        which solution is selected as the best
    :return: the best solution amongst the searched solutions, with best being measured by
        goodness_fn
    """
    max_soln_sz = min(max_soln_sz, MAX_CANDIDATES)
    solution = [None] * max_soln_sz

    # setup the comparison functions, and the current best metric to start the search
    if isinstance(goodness_fn_comparator, str):
        if goodness_fn_comparator == 'min':
            goodness_fn_comparator = goodness_min
            best_metric = math.inf
        elif goodness_fn_comparator == 'max':
            goodness_fn_comparator = goodness_max
            best_metric = -math.inf
        else:
            raise ValueError("Unrecognized goodness_fn_comparator")
    else:
        best_metric = best_metric_start

    best_soln = []
    num_soln_found = 0

    def find_next(ii):
        """
        Finds a compatible solution in the solution vector for index ii
        :param ii: the index for which to find a solution
        """
        nonlocal best_soln
        nonlocal best_metric
        nonlocal num_soln_found

        candidates = construct_candidates(solution_vector=solution, valid_idx=ii)
        for c in candidates:
            solution[ii] = c
            cur_soln_goodness = goodness_fn(solution_vector=solution, valid_idx=ii)
            # determine if the currently proposed solution has a goodness metric "better" than
            # the current, otherwise move onto the next candidate
            continue_search = goodness_fn_comparator(best_metric, cur_soln_goodness)
            if is_a_partial_solution(solution_vector=solution, valid_idx=ii) and continue_search:
                # if the current candidate is a partial solution, then ...
                if (is_a_full_solution(solution_vector=solution, valid_idx=ii)) or \
                   (ii >= max_soln_sz - 1) or \
                   (find_next(ii + 1)):
                    # if the current solution is a full solution, or if we have reached the maximum
                    # solution size, update our notion of the best solution w/ the goodness_fn provided.
                    # Alternatively, we recursively call to see if we can add onto the solution first
                    cur_soln_goodness = goodness_fn(solution_vector=solution, valid_idx=ii)
                    is_best_soln = goodness_fn_comparator(best_metric, cur_soln_goodness)
                    if is_best_soln:
                        best_soln = copy.copy(solution[0:ii+1])
                        best_metric = cur_soln_goodness

                    num_soln_found += 1
                    if num_soln_found < max_solutions:
                        # move back one level in the tree and continue the search
                        find_next(ii)
                    else:
                        break

        return None

    find_next(0)

    return best_soln
```

We see above a Python implementation of a general backtracking with branch and bound engine.  The inputs are very similar to the plain backtracking engine, with some slight modifications.  The additional inputs are described here.  In this implementation, we have the notion of a partial solution and a full solution.  A partial solution is one where the current candidate solution vector meets the constraints of the problem, but the solution is not complete.  A full solution is one where the solution vector meets all the constraints of the problem and the solution is complete.  To differentiate, lets refer back to our coin changing problem.  An example of a partial solution is $[10, 10]$, where as a full solution would be $$[10, 10, 5, 1, 1, 1]$$.  By definition, if a solution is a full solution it is also a partial solution, whereas a partial solution does not imply a full solution.  

An additional concept in this implementation of backtracking with branch and bound is "goodness".  Recall that branch and bound tries to find an optimal solution, so the algorithm needs to be able to evaluate how good a potential solution is.  The purpose of the `goodness_fn` input is to return a metric of goodness for the current solution vector.  Relatedly, the `goodness_fn_comparator` input configures whether the goodness metric returned by `goodness_fn` should be maximized or minimized, or compared in some other custom manner through the definition of a callable.  `max_solutions` determines how many solutions the branch and bound algorithm will search before returning the most optimal one found so far.  This can be useful when solving problems with large search spaces.


#### Coin Changing
Lets use the backtracking to solve the coin changing problem!

```python
def solve_coinchange_backtrack(change_needed, coins):
    max_soln_sz = 10

    def construct_change_candidates(solution_vector=None, valid_idx=0):
        # assume that we have an infinite amount of coins of each kind,
        # so our candidates here are the entire set of coins
        return coins

    def is_a_coinchange_solution(solution_vector=None, valid_idx=0):
        current_soln = solution_vector[0:valid_idx + 1]
        current_change_made = sum(current_soln)
        remainder = change_needed - current_change_made
        if len(current_soln) < max_soln_sz:
            # check if a partial solution is valid.  Here, we haven't filled in
            # all the slots, so any time we have a remainder of >= 0, we are still
            # valid
            if remainder >= 0:
                return True
            else:
                return False
        else:
            # this is a full solution b/c all the slots are filled.  Here, a solution
            # is only valid if it sums to 0
            if remainder == 0:
                return True
            else:
                return False

    a_soln = backtrack.backtrack(construct_change_candidates, is_a_coinchange_solution, max_soln_sz)
    print(a_soln)
```

The implementation above defines the two callback functions needed to use the backtracking algorithm, the `get_candidates` and the `is_a_solution` functions.  The driver function takes two inputs, the change needed, and a list of coin denominations which are allowable in the solution.  The `construct_candidates` implementation just returns the list of allowable coins, and the `is_a_solution` returns True or False depending on whether the solution is valid, or not.

Notice here that the solution depends on the order in which the `construct_candidates` returns the candidates.  If it returns pennies first, the first solution that backtracking will find is a vector of 28 pennies, due to backtracking being analogous to depth first search.  On the other hand, if the first solution returned by `construct_candidates` is a dime, then you can be sure that the solution will contain two dimes, and thus be shorter.

Now, lets solve the same problem using branch and bound.  

```python
def solve_coinchange_bnb(change_needed, coins):
    max_soln_sz = 10
    max_search = 100

    def get_remainder(solution_vector, valid_idx):
        current_soln = solution_vector[0:valid_idx + 1]
        current_change_made = sum(current_soln)
        remainder = change_needed - current_change_made
        return remainder

    def construct_change_candidates(solution_vector=None, valid_idx=0):
        # assume that we have an infinite amount of coins of each kind,
        # so our candidates here are the entire set of coins
        return coins

    def is_a_partial_coinchange_solution(solution_vector=None, valid_idx=0):
        if is_a_full_coinchange_solution(solution_vector, valid_idx):
            return True
        remainder = get_remainder(solution_vector, valid_idx)
        current_soln_len = valid_idx + 1
        if current_soln_len < max_soln_sz:
            # check if a partial solution is valid.  Here, we haven't filled in
            # all the slots, so any time we have a remainder of >= 0, we are still
            # valid
            if remainder >= 0:
                return True
            else:
                return False
        else:
            # this is a full solution b/c all the slots are filled.  Here, a solution
            # is only valid if it sums to 0
            if remainder == 0:
                return True
            else:
                return False

    def is_a_full_coinchange_solution(solution_vector=None, valid_idx=0):
        remainder = get_remainder(solution_vector, valid_idx)
        if remainder == 0:
            return True
        else:
            return False

    def coinchange_goodness_fn(solution_vector=None, valid_idx=0):
        # the goodness is the # of coins used.  By minimizing this, we
        # minimize the total number of coins used to make change
        return valid_idx+1

    best_soln = backtrack_bnb.backtrack_bnb(construct_change_candidates,
                                            is_a_partial_coinchange_solution,
                                            is_a_full_coinchange_solution,
                                            coinchange_goodness_fn,
                                            max_soln_sz,
                                            max_search,
                                            'min')
    print(best_soln)
```

Here, we define the required callable functions to use the branch and bound engine.  `construct_candidates` remains the same, and the `is_a_partial_solution` defines whether the solution is partial (or full, by definition).  The `is_a_full_solution` determines whether the remainder is 0 after deducting the candidate coin values, and only returns True if the remainder is $$0$$, indicating the solution is complete.  The goodness function returns the length of the solution, meaning that this configuration of the branch and bound algorithm is attempting to find the solution which minimizes the number of coins needed to make the required change.

Full runnable, working code is here:
1. [Coin change with Backtracking](https://github.com/kkarrancsu/algorithms/blob/master/search/backtrack_coinchanging.py)
2. [Coin change with Branch and Bound](https://github.com/kkarrancsu/algorithms/blob/master/search/bnb_coinchanging.py)

#### Traveling Salesman Problem
As another example, lets use the backtracking and branch-and-bound engines developed above to solve the traveling salesman problem (TSP)!  Recall that the traveling salesman problem requires us to find a minimum cost path that an agent can take, which visits every city in the network exactly once, and returns to where it started.  Here, the cost of each leg is indicated by the weight of the graph from City A to City B.  

Note that this problem inherently requires us to find an optimal solution.  This means that we need to use Branch and Bound for this problem, and not backtracking, since backtracking has no concept of optimality.  However, we could still use backtracking to find *a* solution, just not the *best* solution.  So, lets begin again with solving this problem using backtracking.  In order to easily use the backtracking and branch-and-bound algorithms, we internally will represent the solution to the TSP problem as a list, with each element of the list representing the city to visit next.  For example, a solution vector $$[0, 1, 4, 2, 5, 3, 0]$$ would mean that the agent started in City 0, then went to City 1, then City 4, then City 2, then City 5, then City 3, and then back to City 0. With this setup, lets implement the necessary callback functions to use the engines above.


```python
def solve_tsp_backtrack(tsp_problem):
    soln_sz = len(tsp_problem) + 1  # make it a roundtrip

    def construct_tsp_candidates(solution_vector=None, valid_idx=0):
        # a valid candidate is the next valid move we can take, from our current
        # location.
        current_city = solution_vector[valid_idx]
        if (valid_idx == 0 and current_city is None) or (valid_idx == soln_sz - 1):
            return [0]  # start & end on the same city-A

        valid_next_cities = []
        for ii in range(len(tsp_problem)):  # len(tsp_problem) gets us the # of cities
            if ii not in solution_vector[0:valid_idx]:
                valid_next_cities.append(ii)

        return valid_next_cities

    def is_a_tsp_solution(solution_vector=None, valid_idx=0):
        # in the context of TSP, a solution is one in which we don't visit
        # the same city twice.  Hence, in our solution vector, we simply
        # see if all the elements are unique.  If so, this means that it we
        # have not revisited and thus, we have a partial solution
        if valid_idx < soln_sz - 1:
            solution_vector_valid = solution_vector[0:valid_idx + 1]
            solution_set = set(solution_vector_valid)  # casting as a set removes redundant elements
            # easy to understand this code, but probably
            # inefficient

            if len(solution_vector_valid) == len(solution_set):
                return True
            else:
                return False
        else:
            # here, we only deem it a solution if the path is complete
            # meaning that we started at 0 and ended at 0.
            # we don't need to check the middle againk b/c that should have
            # been taken care of with the if block above
            if solution_vector[0] == 0 and solution_vector[-1] == 0:
                return True
            else:
                return False

    a_soln = backtrack.backtrack(construct_tsp_candidates, is_a_tsp_solution, soln_sz)
    print(a_soln)
```

To use backtracking, we again implement `is_a_solution` and `construct_candidates`.  Notice that the `construct_candidates` function here returns all cities which have not yet been visited.  We assume for this problem that there is a path between every city, so no additional checks for whether a path exists needs to be checked.  If that were an additional constraint, it could be easily incorporated by updating the `construct_candidates` function accordingly.  The initial/final conditions of the `construct_candidates` also force the agent to start at City 0, and end at City 0.  In this manner, the constraint that the agent needs to make a complete path is met.  

The `is_a_solution` implementation returns True if the solution is a full or partial solution.  In the partial solution case, we ensure that the cities we've visited are unique.  In the full solution case, we simply check that we've made a complete path.  Running this with the backtracking engine, we find that we simply are returned a path which traverses the cities in the order that is returned by `construct_tsp_candidates`, because every solution is "valid".

Lets now examine how to solve this with backtracking.  The implementation is here:

```python
def solve_tsp_bnb(tsp_problem):
    soln_sz = len(tsp_problem) + 1  # needs to be a roundtrip
    max_solutions = 100

    def construct_tsp_candidates(solution_vector=None, valid_idx=0):
        # a valid candidate is the next valid move we can take, from our current
        # location.

        current_city = solution_vector[valid_idx]
        if (valid_idx == 0 and current_city is None) or (valid_idx == soln_sz - 1):
            return [0]  # start & end on the same city-A

        valid_next_cities = []
        for ii in range(len(tsp_problem)):  # len(tsp_problem) gets us the # of cities
            if ii not in solution_vector[0:valid_idx]:
                valid_next_cities.append(ii)

        return valid_next_cities

    def is_a_partial_tsp_solution(solution_vector=None, valid_idx=0):
        # in the context of TSP, a solution is one in which we don't visit
        # the same city twice.  Hence, in our solution vector, we simply
        # see if all the elements are unique.  If so, this means that it we
        # have not revisited and thus, we have a partial solution
        if valid_idx < soln_sz - 1:
            solution_vector_valid = solution_vector[0:valid_idx + 1]
            solution_set = set(solution_vector_valid)  # casting as a set removes redundant elements
                                                       # easy to understand this code, but probably
                                                       # inefficient

            if len(solution_vector_valid) == len(solution_set):
                return True
            else:
                return False
        else:
            # here, we only deem it a solution if the path is complete
            # meaning that we started at 0 and ended at 0.
            # we don't need to check the middle againk b/c that should have
            # been taken care of with the if block above
            if solution_vector[0] == 0 and solution_vector[-1] == 0:
                return True
            else:
                return False

    def is_a_full_tsp_solution(solution_vector=None, valid_idx=0):
        if valid_idx < soln_sz - 1:
            return False
        else:
            # here, we only deem it a solution if the path is complete
            # meaning that we started at 0 and ended at 0.
            # we don't need to check the middle againk b/c that should have
            # been taken care of with the if block above
            if solution_vector[0] == 0 and solution_vector[-1] == 0:
                return True
            else:
                return False

    def tsp_goodness_fn(solution_vector=None, valid_idx=0):
        # from the TSP perspective, we'd like to minimize our total distance
        # so, if our metric is to minimize the total weight of the graphs.
        # the goodness metric is the total weight of the path
        total_weight = 0
        for ii in range(1, valid_idx+1):
            from_node = solution_vector[ii-1]
            to_node = solution_vector[ii]
            total_weight += tsp_problem[from_node][to_node]

        return total_weight

    best_soln = backtrack_bnb.backtrack_bnb(construct_tsp_candidates, is_a_partial_tsp_solution,
                                            is_a_full_tsp_solution,
                                            tsp_goodness_fn, soln_sz, max_solutions, 'min')
    print(best_soln)
```

The only meaningfully different function to discuss here between backtrack and branch and bound is the `goodness_fn`.  Recall that the weight of the path represents the cost of traveling between the two cities that that path connects.  The total weight or cost of taking a certain route is then the sum of the weights of all of the paths that are taken by the agent.  Using this, we can see that the branch and bound will find the optimal solution.

Full runnable, working code is here:
1. [TSP with Backtracking](https://github.com/kkarrancsu/algorithms/blob/master/search/backtrack_tsp.py)
2. [TSP with Branch and Bound](https://github.com/kkarrancsu/algorithms/blob/master/search/bnb_tsp.py)

## Conclusion
In this post, we went through backtracking and branch and bound.  In my mind, the most important takeaway is to try to understand when these algorithmic approaches can be applied to problems of interest.

## References
1. Toal, R. (n.d.). Backtracking. Retrieved December 27, 2020, from https://cs.lmu.edu/~ray/notes/backtracking/
2. Skiena, S. S. (2012). The algorithm design manual. London: Springer.
