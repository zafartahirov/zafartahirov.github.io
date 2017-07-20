---
layout: post
title: "Hungarian Algorithm"
modified: 2017-07-19 13:43:27 -0700
tags: [algorithms, bipartite matching, assignment]
categories: [python]
description:
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
---

## Problem Statement

The problem of the assignment is fairly straight-forward.
To describe it, let us consider several examples

- There are $$n$$ workers and $$n$$ tasks.
You know exactly how much you need to pay each worker to perform one or another task.
You also know that every worker can only perform one task.
Your goal is to assign each worker some a task, while minimizing your expenses.
- Given matrix $$M$$ of size $$n\times n$$, you need to select one element per row and column, such that the sum of all selected cells is minimized.
- Given a matrix $$M$$ of size $$n\times n$$, find permutation $$p$$ of length $$n$$, such taht $$\sum{A[i][p[i]]}$$ is minimized
- Given a complete bipartite graph with $$n$$ vertices with every edge having some weight.
Find perfect matching for the graph with minimum weight.

**Note:** All the problems above assume a "square matrix", that is $$n\times n$$ dimensions.
In practice this is rarely the case, and the dimensions are $$n\times m, n\neq m$$. In this case the goal is to match $$\min{n, m}$$ vertices.

**Note:** The minimization problem above could be converted to maximization problem by multiplying all weights by $$-1$$.

<!-- more -->

[Most info here comes from here (Russian)](http://e-maxx.ru/algo/assignment_hungary)

## History

[Hungarian algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm) was developed and published by Harold Kuhn in 1955.
Kuhn named the algorithm "Hungarian" because he based his work on the works of two hungarian ,athematicians, Dénis König and Jenö Egerváry.

In 1957 James Munkres has shown that Hungarian algorithm is strictly polynomial.
Because of the contributions of Munkres, the algorithm is sometimes referred to as "Kuhn-Munkres algorithm", "Kuhn algorithm", or "Munkres algorithm".
Original algorithm had asymptotic complexity $$O(n^4)$$, and it were Jack Edmonds,  Richard Karp, and Tomizawa have shown a version of the algorithm with $$O(n^3)$$ complexity.

## Algorithm description

To avoid any future confusion let us state beforehand taht we are considering the assignment problem in the matrix context (i.e. given matrix $$M$$, pick $$n$$ cells in different rows and columns).
Indexing of the arrays in the current analysis will be 1-based, i.e. $$A[1\ldots n][1\ldots n]$$.
Also, note that all numbers in $$A[][]$$ are non-negative.
If there are negative numbers in the matrix, you can always add some value to all elements to make them positive.

### $$O(n^4)$$ algorithm

Rephrasing the [Wiki page][wiki-hung], there are two arbitrary arrays $$u[1 \ldots n]$$ and $$v[1 \ldots n]$$ defined as **potential**, such that

$$
 u[i] + v[j] \le A[i][j], (i = 1 \ldots n, j = 1 \ldots n)
$$

_Note that $$u[i]$$ and $$v[j]$$ correspond to rows and columns of matrix $$M$$ respectively._

In that case, we can define the value of the potential as

$$
f = \sum_{i=1}^n u[i] + \sum_{i=1}^n v[i].
$$

From one side one can see that the cost of the final optimal solution $$C_{solution}$$ is not less than the potential (proof left as an exercise for the reader):

$$
 C_{solution} \ge f.
$$

From another side, it appears, there is always a solution and a potential where the inequality turns into the equality (Hungarian algorithm is the proof of it).
Meanwhile, note that if some solution has some associated cost equal to its potential, this solution will be considered optimal.

Let's call the edge $$(i, j)$$ **tight** iff:

$$
 u[i] + v[j] = A[i][j]
$$

In terms of a [bipartite graphs](https://en.wikipedia.org/wiki/Complete_bipartite_graph), we can view this assumptions as follows:

Given a bipartite graph $$H$$ consisting of only tight edges, the Hungarian algorithm maintains the matched pairs list $$M$$ which maximizes the number of pairs for a given potential $$f$$. The moment $$M$$ has $$n$$ elements, the edges of this pairs list will be the optimal solution (it is optimal because the potential will be equal to the solution cost).

**Now to the algorithm istself**

Given two parts of the bipartite graph: first $$F$$ and second $$S$$,

1. Initially the potential is zero $$\forall i,j u[i] = v[i] = 0$$, and the pair list is empty $$\|M\|=0$$
2. While maintaining the potential, try to find maximum number of matches.
To achieve that we can use [any matching algorithm][wiki-hung].
Hint: BFS and DFS can be quite efficient at this step.
  - If the solution is found, return the solution
  - If the solution is not found, but the optimal pairset $$M$$ was updated, repeat this step
  - If the solution is not found, and the optimal pairset $$M$$ was not updated, go to the next step.
3. Modify the way you acquire the pairs, and the potentials, and go to step 2.
One way of doing it is, if you were looking for pairs from $$F$$ to $$S$$, now look from $$S$$ to $$F$$ -- from matrix point of view, if you were looking into minimizing the rows cost, continue with minimizing the column costs (not really! see the note below).

**Note on step 3:** Formally, this step is a way of changing the potentials $$u$$ and $$v$$, such that a viable solution is still possible.
Without providing the proofs, say we have $$Z_1 \subseteq F$$ and $$Z_2 \subseteq S$$, such that $$Z_1, Z_2$$ contain the visited vertices.
In that case we can compute strictly positive potential gain $$\Delta$$

$$
\Delta = \min_{i \in Z_1, j \notin Z_2} \left\{ A[i][j] - u[i] - v[j]\right\}
$$

Now we can update the potentials, without the total potential being different, as follows

$$
\begin{align}
\forall i \in Z_1, \forall j \in Z_2&: \\
u[i] &= u[i] + \Delta \\
v[j] &= v[j] - \Delta
\end{align}
$$

Note that even though we changed the potential, the partially computed $$M$$ is still valid.
At this point we can note that potential changes cannot continue indefinitely, because at every iteration the number of visited vertices that could be connected only increases ($$\|Z_1\| + \|Z_2\|$$can only increase However, we cannot claim that the number of tight edges increases).
That means that the maximum number of changes in the potential that can happen does not exceed $$n$$.

### $$O(n^3)$$ algorithm

Now, let us describe a $$O(n^3)$$ algorithm (or a $$O(n^2m)$$ for non-square matrices).

Key idea: Don't consider the whole matrix at once, but append a new row at every iteration. In that case the algorithm above could be rewritten as:

1. Add the next row in $$A$$ to the "used" list
2. While the $$\|M\|$$ is not increasing, that is the number of edges that originate in the current row doesn't increase (or the potential for this row is non-zero), keep on changing the potentials using the **Note on step 3** described above.
3. The moment we notice that the number of pairs has increased, that is the potential for the vertex is reduced to zero, we add this vertex to the found pairset by iterating through all the found pairsets that were found before, and repeat the process for the next row (step 1).

Steps 2 and 3 have asymptotic complexity of $$O(nm)$$, and they have to be repeated $$n$$ times.
The proof for this is left as an exercise for the reader.

To achieve the given asymptotics, we need to note the following key ideas:

- To find if the pairset size started increasing, we don't need to revisit all the nodes to check for potential changes ([Kuhn traversal](https://cs.stackexchange.com/questions/42400/why-is-one-traversal-sufficient-for-the-kuhns-maximal-matching-problem-algorith)).
Instead, after each update of the potentials, we look at the $$F$$ vertex of the edge, and if it was previously visited, we mark the $$S$$ side of the vertex as visited as well, and continue iterating from that point on.
- Given the previous key idea, we can amend the algorithm: at every iteration of the algorithm, if we are updating the potentials, we check which of the columns ($$S$$ vertex) was just found.
We check if this column is already used, and if not we found another pair to add to $$M$$.
At this point, we mark the column as used.
- To compute the potentials quickly, we can keep track of thew minimum vertex found for every column $$j$$, which will be updated every time the potentials change.

$$
\begin{align}
minv[j] &= \min_{i\in Z_1}{A[i][j] - u[i] - v[j]} \\
\Delta &= \min_{j\notin Z_2}{minv[j]}
\end{align}
$$

Now we can find $$\Delta$$ in linear time.

## Hungarian algorithm implementation in $$O(n^2m)$$ time

Given a matrix $$M$$, rows represent vertices of $$F$$ part of the bipartite graph, and columns represent the other part $$S$$.
The matching needs to be done from rows to columns.

The codes in this section are written by [Andrej Lopatin](https://medium.com/@VeeRoute/yet-another-win-of-the-team-trained-by-andrey-lopatin-acm-icpc-2016-9296d03553a9).

The code below solves the matching problem for a matrix $$A[1\ldots n][1\ldots m]$$, where $$n \le m$$.
It uses 1-indexing to simplify and shorted the codes: the "fake-0" index is used as a flag, and avoids additional checks if the computation was done :).

Arrays $$ u[0 \ldots n]$$ and $$v[0 \ldots m]$$ store the potentials, which are initilized to 0 (same as 0-row matrix).
Also, the current implementation works even if there are negative numbers in the edge weights.

Array $$p[0\ldots m]$$ holds information about matched pairs: for every column $$i = 1\ldots m$$, it holds the value of the corresponding row (or 0 if still no matching found).
$$p[0]$$ keeps track of the current row, which is done to simplify the code and avoid checks.

Array $$minv[1\ldots m]$$ holds the temporary minimums in the potentials for each column such that $$minv[j] = \min_{i\in Z_1}{A[i][j] - u[i] - v[j]}$$.
This is used for a quick re-computation and update of the potentials.

Array $$way[1\ldots m]$$ holds information about where the minimums are achieved, and is used to reconstruct the graph matching.
One might think that array $$way[]$$ should hold information about every row and that we require additional array: for every row we need to keep track of the corresponding column.
However, note taht Kuhn algorithm reaches the rows from columns after visiting edges of that pair.
That's why to reconstruct the matching of the graph, we can always get the rows from the matched pair array (i.e. $$p[]$$).
Thus, $$way[j]$$ has information about the previous column, or 0 if there is no match.

The implementation of the algorithm includes a for-loop over the rows of the matrix.
Inside the loop we keep track of the current row by placing it in the $$p[0]$$, and start a $$do..while..$$ loop which finds the first unused column $$j0$$.
Every iteration of the loop marks column $$j0$$ as used (previously found column), and gets the appropriate pair $$i0$$.
At this point, because we have a new used row $$i0$$, we recompute the $$minv[]$$, and find $$delta$$ while we are at it.
We also find in which column $$j1$$ the minimum was found.
If $$delta$$ is zero, we don't have to update the potential -- we already have a matching pair.
After that potentials $$u[],v[]$$ are updated keeping in mind the changes we introduced in $$minv[]$$.
After the $$do..while..$$ loop finishes, we will have found a chain of pairs, and to reconstruct the pairset, we can "unwind" $$j0$$ using the ancestry array $$way[]$$.

{% highlight c++ linenos %}
// Given the cost matrix "vector<vector<int>> A {...};"
// Find the maximum matching "vector<pair<int,int>>result;" with all pairs
// As well as total cost "int C;" with the minimum assignment cost.
vector<int> u (n+1), v (m+1), p (m+1), way (m+1);
for (int i=1; i<=n; ++i) {
	p[0] = i;
	int j0 = 0;
	vector<int> minv (m+1, INF);
	vector<char> used (m+1, false);
	do {
		used[j0] = true;
		int i0 = p[j0],  delta = INF,  j1;
		for (int j=1; j<=m; ++j)
			if (!used[j]) {
				int cur = A[i0][j]-u[i0]-v[j];
				if (cur < minv[j])
					minv[j] = cur,  way[j] = j0;
				if (minv[j] < delta)
					delta = minv[j],  j1 = j;
			}
		for (int j=0; j<=m; ++j)
			if (used[j])
				u[p[j]] += delta,  v[j] -= delta;
			else
				minv[j] -= delta;
		j0 = j1;
	} while (p[j0] != 0);
	do {
		int j1 = way[j0];
		p[j0] = p[j1];
		j0 = j1;
	} while (j0);
}

vector<pair<int, int>> result;
for (int i = 1; i <= m; ++i){
  result.push_back(make_pair(p[i], i));
}

int C = -v[0];
{% endhighlight %}

## References

[E-Maxx](http://e-maxx.ru/algo/assignment_hungary)

[Wiki article](https://en.wikipedia.org/wiki/Hungarian_algorithm)

[wiki-hung]: https://en.wikipedia.org/wiki/Hungarian_algorithm
