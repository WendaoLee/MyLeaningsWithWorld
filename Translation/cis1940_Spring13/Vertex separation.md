#Vertex #Wholemeal

顶点分割

From https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html
# Vertex separation[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#vertex-separation "Permalink to this headline")

This module implements several algorithms to compute the vertex separation of a digraph and the corresponding ordering of the vertices. It also implements tests functions for evaluation the width of a linear ordering.

Given an ordering v1,⋯,vn of the vertices of V(G), its _cost_ is defined as:

c(v1,...,vn)=max1≤i≤nc′({v1,...,vi})

Where

c′(S)=|NG+(S)∖S|

The _vertex separation_ of a digraph G is equal to the minimum cost of an ordering of its vertices.

**Vertex separation and pathwidth**

The vertex separation is defined on a digraph, but one can obtain from a graph G a digraph D with the same vertex set, and in which each edge uv of G is replaced by two edges uv and vu in D. The vertex separation of D is equal to the pathwidth of G, and the corresponding ordering of the vertices of D, also called a _layout_, encodes an optimal path-decomposition of G. This is a result of Kinnersley [[Kin1992]](https://doc.sagemath.org/html/en/reference/references/index.html#kin1992) and Bodlaender [[Bod1998]](https://doc.sagemath.org/html/en/reference/references/index.html#bod1998).

**This module contains the following methods**

[`pathwidth()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.pathwidth "sage.graphs.graph_decompositions.vertex_separation.pathwidth")

Compute the pathwidth of `self` (and provides a decomposition)

[`path_decomposition()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.path_decomposition "sage.graphs.graph_decompositions.vertex_separation.path_decomposition")

Return the pathwidth of the given graph and the ordering of the vertices resulting in a corresponding path decomposition

[`vertex_separation()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation "sage.graphs.graph_decompositions.vertex_separation.vertex_separation")

Return an optimal ordering of the vertices and its cost for vertex-separation

[`vertex_separation_exp()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation_exp "sage.graphs.graph_decompositions.vertex_separation.vertex_separation_exp")

Compute the vertex separation of G using an exponential time and space algorithm

[`vertex_separation_MILP()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation_MILP "sage.graphs.graph_decompositions.vertex_separation.vertex_separation_MILP")

Compute the vertex separation of G and the optimal ordering of its vertices using an MILP formulation

[`vertex_separation_BAB()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation_BAB "sage.graphs.graph_decompositions.vertex_separation.vertex_separation_BAB")

Compute the vertex separation of G and the optimal ordering of its vertices using a branch and bound algorithm

[`lower_bound()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.lower_bound "sage.graphs.graph_decompositions.vertex_separation.lower_bound")

Return a lower bound on the vertex separation of G

[`is_valid_ordering()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.is_valid_ordering "sage.graphs.graph_decompositions.vertex_separation.is_valid_ordering")

Test if the linear vertex ordering L is valid for (di)graph G

[`width_of_path_decomposition()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.width_of_path_decomposition "sage.graphs.graph_decompositions.vertex_separation.width_of_path_decomposition")

Return the width of the path decomposition induced by the linear ordering L of the vertices of G

[`linear_ordering_to_path_decomposition()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.linear_ordering_to_path_decomposition "sage.graphs.graph_decompositions.vertex_separation.linear_ordering_to_path_decomposition")

Return the path decomposition encoded in the ordering L

## Exponential algorithm for vertex separation[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#exponential-algorithm-for-vertex-separation "Permalink to this headline")

In order to find an optimal ordering of the vertices for the vertex separation, this algorithm tries to save time by computing the function c′(S) **at most once** once for each of the sets S⊆V(G). These values are stored in an array of size 2n where reading the value of c′(S) or updating it can be done in constant (and small) time.

Assuming that we can compute the cost of a set S and remember it, finding an optimal ordering is an easy task. Indeed, we can think of the sequence v1,...,vn of vertices as a sequence of _sets_ {v1},{v1,v2},...,{v1,...,vn}, whose cost is precisely maxc′({v1}),c′({v1,v2}),...,c′({v1,...,vn}). Hence, when considering the digraph on the 2n sets S⊆V(G) where there is an arc from S to S′ if S′=S∩{v} for some v (that is, if the sets S and S′ can be consecutive in a sequence), an ordering of the vertices of G corresponds to a _path_ from ∅ to {v1,...,vn}. In this setting, checking whether there exists a ordering of cost less than k can be achieved by checking whether there exists a directed path ∅ to {v1,...,vn} using only sets of cost less than k. This is just a depth-first-search, for each k.

**Lazy evaluation of** c′

In the previous algorithm, most of the time is actually spent on the computation of c′(S) for each set S⊆V(G) – i.e. 2n computations of neighborhoods. This can be seen as a huge waste of time when noticing that it is useless to know that the value c′(S) for a set S is less than k if all the paths leading to S have a cost greater than k. For this reason, the value of c′(S) is computed lazily during the depth-first search. Explanation :

When the depth-first search discovers a set of size less than k, the costs of its out-neighbors (the potential sets that could follow it in the optimal ordering) are evaluated. When an out-neighbor is found that has a cost smaller than k, the depth-first search continues with this set, which is explored with the hope that it could lead to a path toward {v1,...,vn}. On the other hand, if an out-neighbour has a cost larger than k it is useless to attempt to build a cheap sequence going though this set, and the exploration stops there. This way, a large number of sets will never be evaluated and _a lot_ of computational time is saved this way.

Besides, some improvement is also made by “improving” the values found by c′. Indeed, c′(S) is a lower bound on the cost of a sequence containing the set S, but if all out-neighbors of S have a cost of c′(S)+5 then one knows that having S in a sequence means a total cost of at least c′(S)+5. For this reason, for each set S we store the value of c′(S), and replace it by max(c′(S),minnext) (where minnext is the minimum of the costs of the out-neighbors of S) once the costs of these out-neighbors have been evaluated by the algorithm.

Note

Because of its current implementation, this algorithm only works on graphs on less than 32 vertices. This can be changed to 64 if necessary, but 32 vertices already require 4GB of memory. Running it on 64 bits is not expected to be doable by the computers of the next decade :−D

**Lower bound on the vertex separation**

One can obtain a lower bound on the vertex separation of a graph in exponential time but _small_ memory by computing once the cost of each set S. Indeed, the cost of a sequence v1,...,vn corresponding to sets {v1},{v1,v2},...,{v1,...,vn} is

maxc′({v1}),c′({v1,v2}),...,c′({v1,...,vn})≥maxc1′,...,cn′

where ci is the minimum cost of a set S on i vertices. Evaluating the ci can take time (and in particular more than the previous exact algorithm), but it does not need much memory to run.

## MILP formulation for the vertex separation[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#milp-formulation-for-the-vertex-separation "Permalink to this headline")

We describe below a mixed integer linear program (MILP) for determining an optimal layout for the vertex separation of G, which is an improved version of the formulation proposed in [[SP2010]](https://doc.sagemath.org/html/en/reference/references/index.html#sp2010). It aims at building a sequence St of sets such that an ordering v1,...,vn of the vertices correspond to S0={v1},S2={v1,v2},...,Sn−1={v1,...,vn}.

**Variables:**

-   yvt – Variable set to 1 if v∈St, and 0 otherwise. The order of v in the layout is the smallest t such that yvt==1.
    
-   uvt – Variable set to 1 if v∉St and v has an in-neighbor in St. It is set to 0 otherwise.
    
-   xvt – Variable set to 1 if either v∈St or if v has an in-neighbor in St. It is set to 0 otherwise.
    
-   z – Objective value to minimize. It is equal to the maximum over all step t of the number of vertices such that uvt==1.
    

**MILP formulation:**

Minimize:zSuch that:xvt≤xvt+1∀v∈V, 0≤t≤n−2yvt≤yvt+1∀v∈V, 0≤t≤n−2yvt≤xwt∀v∈V, ∀w∈N+(v), 0≤t≤n−1∑v∈Vyvt=t+10≤t≤n−1xvt−yvt≤uvt∀v∈V, 0≤t≤n−1∑v∈Vuvt≤z0≤t≤n−10≤xvt≤1∀v∈V, 0≤t≤n−10≤uvt≤1∀v∈V, 0≤t≤n−1yvt∈{0,1}∀v∈V, 0≤t≤n−10≤z≤n

The vertex separation of G is given by the value of z, and the order of vertex v in the optimal layout is given by the smallest t for which yvt==1.

## Branch and Bound algorithm for the vertex separation[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#branch-and-bound-algorithm-for-the-vertex-separation "Permalink to this headline")

We describe below the principle of a branch and bound algorithm (BAB) for determining an optimal ordering for the vertex separation of G, as proposed in [[CMN2014]](https://doc.sagemath.org/html/en/reference/references/index.html#cmn2014).

**Greedy steps:**

Let us denote L(S) the set of all possible orderings of the vertices in S, and let LP(S)⊆L(S) be the orderings starting with a prefix P. Let also c(L) be the cost of the ordering L∈L(V) as defined above.

Given a digraph D=(V,A), a set S⊂V, and a prefix P, it has been proved in [[CMN2014]](https://doc.sagemath.org/html/en/reference/references/index.html#cmn2014) that minL∈LP(V)c(L)=minL∈LP+v(V)c(L) holds in two (non exhaustive) cases:

or{N+(v)⊆S∪N+(S)v∈N+(S) and N+(v)∖(S∪N+(S))={w}

In other words, if we find a vertex v satisfying the above conditions, the best possible ordering with prefix P has the same cost as the best possible ordering with prefix P+v. So we can greedily extend the prefix with vertices satisfying the conditions which results in a significant reduction of the search space.

**The algorithm:**

Given the current prefix P and the current upper bound UB (either an input upper bound or the cost of the best solution found so far), apply the following steps:

-   Extend the prefix P into a prefix P′ using the greedy steps as described above.
    
-   Sort the vertices v∈V∖P′ by increasing values of |N+(P+v)|, and prune the vertices with a value larger or equal to UB. Let Δ be the resulting sorted list.
    
-   Repeat with prefix P′+v for all v∈Δ and keep the best found solution.
    

If a lower bound is passed to the algorithm, it will stop as soon as a solution with cost equal to that lower bound is found.

**Storing prefixes:**

If for a prefix P we have c(P)<minL∈LP(V)c(L)=C, then for any permutation P′ of P we have minL∈LP′(V)c(L)≥C.

Thus, given such a prefix P there is no need to explore any of the orderings starting with one of its permutations. To do so, we store P (as a set of vertices) to cut branches later. See [[CMN2014]](https://doc.sagemath.org/html/en/reference/references/index.html#cmn2014) for more details.

Since the number of stored sets can get very large, one can control the maximum length and the maximum number of stored prefixes.

## Authors[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#authors "Permalink to this headline")

-   Nathann Cohen (2011-10): Initial version and exact exponential algorithm
    
-   David Coudert (2012-04): MILP formulation and tests functions
    
-   David Coudert (2015-01): BAB formulation and tests functions
    

## Methods[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#methods "Permalink to this headline")

sage.graphs.graph_decompositions.vertex_separation.is_valid_ordering(_G_, _L_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.is_valid_ordering "Permalink to this definition")

Test if the linear vertex ordering L is valid for (di)graph G.

A linear ordering L of the vertices of a (di)graph G is valid if all vertices of G are in L, and if L contains no other vertex and no duplicated vertices.

INPUT:

-   `G` – a Graph or a DiGraph.
    
-   `L` – an ordered list of the vertices of `G`.
    

OUTPUT:

Returns `True` if L is a valid vertex ordering for G, and `False` otherwise.

EXAMPLES:

Path decomposition of a cycle:

sage: from sage.graphs.graph_decompositions import vertex_separation
sage: G = graphs.CycleGraph(6)
sage: L = G.vertices(sort=True)
sage: vertex_separation.is_valid_ordering(G, L)
True
sage: vertex_separation.is_valid_ordering(G, [1,2])
False

sage.graphs.graph_decompositions.vertex_separation.linear_ordering_to_path_decomposition(_G_, _L_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.linear_ordering_to_path_decomposition "Permalink to this definition")

Return the path decomposition encoded in the ordering L

INPUT:

-   `G` – a Graph
    
-   `L` – a linear ordering for G
    

OUTPUT:

A path graph whose vertices are the bags of the path decomposition.

EXAMPLES:

The bags of an optimal path decomposition of a path-graph have two vertices each:

sage: from sage.graphs.graph_decompositions.vertex_separation import vertex_separation
sage: from sage.graphs.graph_decompositions.vertex_separation import linear_ordering_to_path_decomposition
sage: g = graphs.PathGraph(5)
sage: pw, L = vertex_separation(g, algorithm = "BAB"); pw
1
sage: h = linear_ordering_to_path_decomposition(g, L)
sage: sorted(h, key=str)
[{0, 1}, {1, 2}, {2, 3}, {3, 4}]
sage: sorted(h.edge_iterator(labels=None), key=str)
[({0, 1}, {1, 2}), ({1, 2}, {2, 3}), ({2, 3}, {3, 4})]

Giving a non-optimal linear ordering:

sage: g = graphs.PathGraph(5)
sage: L = [1, 4, 0, 2, 3]
sage: from sage.graphs.graph_decompositions.vertex_separation import width_of_path_decomposition
sage: width_of_path_decomposition(g, L)
3
sage: h = linear_ordering_to_path_decomposition(g, L)
sage: h.vertices(sort=True)
[{0, 2, 3, 4}, {0, 1, 2}]

The bags of the path decomposition of a cycle have three vertices each:

sage: g = graphs.CycleGraph(6)
sage: pw, L = vertex_separation(g, algorithm = "BAB"); pw
2
sage: h = linear_ordering_to_path_decomposition(g, L)
sage: sorted(h, key=str)
[{0, 1, 5}, {1, 2, 5}, {2, 3, 4}, {2, 4, 5}]
sage: sorted(h.edge_iterator(labels=None), key=str)
[({0, 1, 5}, {1, 2, 5}), ({1, 2, 5}, {2, 4, 5}), ({2, 4, 5}, {2, 3, 4})]

sage.graphs.graph_decompositions.vertex_separation.lower_bound(_G_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.lower_bound "Permalink to this definition")

Return a lower bound on the vertex separation of G.

INPUT:

-   `G` – a Graph or a DiGraph
    

OUTPUT:

A lower bound on the vertex separation of D (see the module’s documentation).

Note

This method runs in exponential time but has no memory constraint.

EXAMPLES:

On a circuit:

sage: from sage.graphs.graph_decompositions.vertex_separation import lower_bound
sage: g = digraphs.Circuit(6)
sage: lower_bound(g)
1

sage.graphs.graph_decompositions.vertex_separation.path_decomposition(_G_, _algorithm='BAB'_, _cut_off=None_, _upper_bound=None_, _verbose=False_, _max_prefix_length=20_, _max_prefix_number=1000000_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.path_decomposition "Permalink to this definition")

Return the pathwidth of the given graph and the ordering of the vertices resulting in a corresponding path decomposition.

INPUT:

-   `G` – a Graph
    
-   `algorithm` – string (default: `"BAB"`); algorithm to use among:
    
    -   `"BAB"` – Use a branch-and-bound algorithm. This algorithm has no size restriction but could take a very long time on large graphs. It can also be used to test is the input (di)graph has vertex separation at most `upper_bound` or to return the first found solution with vertex separation less or equal to a `cut_off` value.
        
    -   `exponential` – Use an exponential time and space algorithm. This algorithm only works of graphs on less than 32 vertices.
        
    -   `MILP` – Use a mixed integer linear programming formulation. This algorithm has no size restriction but could take a very long time.
        
-   `upper_bound` – integer (default: `None`); parameter used by the `"BAB"` algorithm. If specified, the algorithm searches for a solution with `width < upper_bound`. It helps cutting branches. However, if the given upper bound is too low, the algorithm may not be able to find a solution.
    
-   `cut_off` – integer (default: `None`); parameter used by the `"BAB"` algorithm. This bound allows us to stop the search as soon as a solution with width at most `cut_off` is found, if any. If this bound cannot be reached, the best solution found is returned, unless a too low `upper_bound` is given.
    
-   `verbose` – boolean (default: `False`); whether to display information on the computations
    
-   `max_prefix_length` – integer (default: 20); limits the length of the stored prefixes to prevent storing too many prefixes. This parameter is used only when `algorithm=="BAB"`.
    
-   `max_prefix_number` – integer (default: 10**6); upper bound on the number of stored prefixes used to prevent using too much memory. This parameter is used only when `algorithm=="BAB"`.
    

OUTPUT:

A pair `(cost, ordering)` representing the optimal ordering of the vertices and its cost.

See also

-   [`Graph.treewidth()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph.html#sage.graphs.graph.Graph.treewidth "sage.graphs.graph.Graph.treewidth") – computes the treewidth of a graph
    

EXAMPLES:

The pathwidth of a cycle is equal to 2:

sage: from sage.graphs.graph_decompositions.vertex_separation import path_decomposition
sage: g = graphs.CycleGraph(6)
sage: pw, L = path_decomposition(g, algorithm = "BAB"); pw
2
sage: pw, L = path_decomposition(g, algorithm = "exponential"); pw
2
sage: pw, L = path_decomposition(g, algorithm = "MILP"); pw
2

sage.graphs.graph_decompositions.vertex_separation.pathwidth(_self_, _k=None_, _certificate=False_, _algorithm='BAB'_, _verbose=False_, _max_prefix_length=20_, _max_prefix_number=1000000_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.pathwidth "Permalink to this definition")

Compute the pathwidth of `self` (and provides a decomposition)

INPUT:

-   `k` – integer (default: `None`); the width to be considered. When `k` is an integer, the method checks that the graph has pathwidth ≤k. If `k` is `None` (default), the method computes the optimal pathwidth.
    
-   `certificate` – boolean (default: `False`); whether to return the path-decomposition itself
    
-   `algorithm` – string (default: `"BAB"`); algorithm to use among:
    
    -   `"BAB"` – Use a branch-and-bound algorithm. This algorithm has no size restriction but could take a very long time on large graphs. It can also be used to test is the input graph has pathwidth ≤k, in which cas it will return the first found solution with width ≤k is `certificate==True`.
        
    -   `exponential` – Use an exponential time and space algorithm. This algorithm only works of graphs on less than 32 vertices.
        
    -   `MILP` – Use a mixed integer linear programming formulation. This algorithm has no size restriction but could take a very long time.
        
-   `verbose` – boolean (default: `False`); whether to display information on the computations
    
-   `max_prefix_length` – integer (default: 20); limits the length of the stored prefixes to prevent storing too many prefixes. This parameter is used only when `algorithm=="BAB"`.
    
-   `max_prefix_number` – integer (default: 10**6); upper bound on the number of stored prefixes used to prevent using too much memory. This parameter is used only when `algorithm=="BAB"`.
    

OUTPUT:

Return the pathwidth of `self`. When `k` is specified, it returns `False` when no path-decomposition of width ≤k exists or `True` otherwise. When `certificate=True`, the path-decomposition is also returned.

See also

-   [`Graph.treewidth()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph.html#sage.graphs.graph.Graph.treewidth "sage.graphs.graph.Graph.treewidth") – computes the treewidth of a graph
    
-   [`vertex_separation()`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation "sage.graphs.graph_decompositions.vertex_separation.vertex_separation") – computes the vertex separation of a (di)graph
    

EXAMPLES:

The pathwidth of a cycle is equal to 2:

sage: g = graphs.CycleGraph(6)
sage: g.pathwidth()
2
sage: pw, decomp = g.pathwidth(certificate=True)
sage: sorted(decomp, key=str)
[{0, 1, 5}, {1, 2, 5}, {2, 3, 4}, {2, 4, 5}]

The pathwidth of a Petersen graph is 5:

sage: g = graphs.PetersenGraph()
sage: g.pathwidth()
5
sage: g.pathwidth(k=2)
False
sage: g.pathwidth(k=6)
True
sage: g.pathwidth(k=6, certificate=True)
(True, Graph on 5 vertices)

sage.graphs.graph_decompositions.vertex_separation.vertex_separation(_G_, _algorithm='BAB'_, _cut_off=None_, _upper_bound=None_, _verbose=False_, _max_prefix_length=20_, _max_prefix_number=1000000_, _solver=None_, _integrality_tolerance=0.001_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation "Permalink to this definition")

Return an optimal ordering of the vertices and its cost for vertex-separation.

INPUT:

-   `G` – a Graph or a DiGraph
    
-   `algorithm` – string (default: `"BAB"`); algorithm to use among:
    
    -   `"BAB"` – Use a branch-and-bound algorithm. This algorithm has no size restriction but could take a very long time on large graphs. It can also be used to test is the input (di)graph has vertex separation at most `upper_bound` or to return the first found solution with vertex separation less or equal to a `cut_off` value.
        
    -   `exponential` – Use an exponential time and space algorithm. This algorithm only works of graphs on less than 32 vertices.
        
    -   `MILP` – Use a mixed integer linear programming formulation. This algorithm has no size restriction but could take a very long time.
        
-   `upper_bound` – integer (default: `None`); parameter used by the `"BAB"` algorithm. If specified, the algorithm searches for a solution with `width < upper_bound`. It helps cutting branches. However, if the given upper bound is too low, the algorithm may not be able to find a solution.
    
-   `cut_off` – integer (default: `None`); parameter used by the `"BAB"` algorithm. This bound allows us to stop the search as soon as a solution with width at most `cut_off` is found, if any. If this bound cannot be reached, the best solution found is returned, unless a too low `upper_bound` is given.
    
-   `verbose` – boolean (default: `False`); whether to display information on the computations
    
-   `max_prefix_length` – integer (default: 20); limits the length of the stored prefixes to prevent storing too many prefixes. This parameter is used only when `algorithm=="BAB"`.
    
-   `max_prefix_number` – integer (default: 10**6); upper bound on the number of stored prefixes used to prevent using too much memory. This parameter is used only when `algorithm=="BAB"`.
    
-   `solver` – string (default: `None`); specify a Mixed Integer Linear Programming (MILP) solver to be used. If set to `None`, the default one is used. For more information on MILP solvers and which default solver is used, see the method [`solve`](https://doc.sagemath.org/html/en/reference/numerical/sage/numerical/mip.html#sage.numerical.mip.MixedIntegerLinearProgram.solve "(in Numerical Optimization v9.7)") of the class [`MixedIntegerLinearProgram`](https://doc.sagemath.org/html/en/reference/numerical/sage/numerical/mip.html#sage.numerical.mip.MixedIntegerLinearProgram "(in Numerical Optimization v9.7)").
    
-   `integrality_tolerance` – float; parameter for use with MILP solvers over an inexact base ring; see [`MixedIntegerLinearProgram.get_values()`](https://doc.sagemath.org/html/en/reference/numerical/sage/numerical/mip.html#sage.numerical.mip.MixedIntegerLinearProgram.get_values "(in Numerical Optimization v9.7)").
    

OUTPUT:

A pair `(cost, ordering)` representing the optimal ordering of the vertices and its cost.

EXAMPLES:

Comparison of methods:

sage: from sage.graphs.graph_decompositions.vertex_separation import vertex_separation
sage: G = digraphs.DeBruijn(2,3)
sage: vs,L = vertex_separation(G, algorithm="BAB"); vs
2
sage: vs,L = vertex_separation(G, algorithm="exponential"); vs
2
sage: vs,L = vertex_separation(G, algorithm="MILP"); vs
2
sage: G = graphs.Grid2dGraph(3,3)
sage: vs,L = vertex_separation(G, algorithm="BAB"); vs
3
sage: vs,L = vertex_separation(G, algorithm="exponential"); vs
3
sage: vs,L = vertex_separation(G, algorithm="MILP"); vs
3

Digraphs with multiple strongly connected components:

sage: from sage.graphs.graph_decompositions.vertex_separation import vertex_separation
sage: D = digraphs.Path(8)
sage: print(vertex_separation(D))
(0, [7, 6, 5, 4, 3, 2, 1, 0])
sage: D = digraphs.RandomDirectedAcyclicGraph(10, .5)
sage: vs,L = vertex_separation(D); vs
0
sage: K4 = DiGraph( graphs.CompleteGraph(4) )
sage: D = K4+K4
sage: D.add_edge(0, 4)
sage: print(vertex_separation(D))
(3, [4, 5, 6, 7, 0, 1, 2, 3])
sage: D = K4+K4+K4
sage: D.add_edge(0, 4)
sage: D.add_edge(0, 8)
sage: print(vertex_separation(D))
(3, [10, 11, 8, 9, 4, 5, 6, 7, 0, 1, 2, 3])

sage.graphs.graph_decompositions.vertex_separation.vertex_separation_BAB(_G_, _cut_off=None_, _upper_bound=None_, _max_prefix_length=20_, _max_prefix_number=1000000_, _verbose=False_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation_BAB "Permalink to this definition")

Branch and Bound algorithm for the vertex separation.

This method implements the branch and bound algorithm for the vertex separation of directed graphs and the pathwidth of undirected graphs proposed in [[CMN2014]](https://doc.sagemath.org/html/en/reference/references/index.html#cmn2014). The implementation is valid for both Graph and DiGraph. See the documentation of the [`vertex_separation`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#module-sage.graphs.graph_decompositions.vertex_separation "sage.graphs.graph_decompositions.vertex_separation") module.

INPUT:

-   `G` – a Graph or a DiGraph.
    
-   `cut_off` – integer (default: `None`); bound to consider in the branch and bound algorithm. This allows us to stop the search as soon as a solution with width at most `cut_off` is found, if any. If this bound cannot be reached, the best solution found is returned, unless a too low `upper_bound` is given.
    
-   `upper_bound` – integer (default: `None`); if specified, the algorithm searches for a solution with `width < upper_bound`. It helps cutting branches. However, if the given upper bound is too low, the algorithm may not be able to find a solution.
    
-   `max_prefix_length` – integer (default: 20); limits the length of the stored prefixes to prevent storing too many prefixes
    
-   `max_prefix_number` – integer (default: 10**6); upper bound on the number of stored prefixes used to prevent using too much memory
    
-   `verbose` – boolean (default: `False`); display some information when set to `True`
    

OUTPUT:

-   `width` – the computed vertex separation
    
-   `seq` – an ordering of the vertices of width `width`
    

EXAMPLES:

The algorithm is valid for the vertex separation:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: D = digraphs.RandomDirectedGNP(15, .2)
sage: vb, seqb = VS.vertex_separation_BAB(D)
sage: vd, seqd = VS.vertex_separation_exp(D)
sage: vb == vd
True
sage: vb == VS.width_of_path_decomposition(D, seqb)
True

The vertex separation of a N×N grid is N:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: G = graphs.Grid2dGraph(4,4)
sage: vs, seq = VS.vertex_separation_BAB(G); vs
4
sage: vs == VS.width_of_path_decomposition(G, seq)
True

The vertex separation of a N×M grid with N<M is N:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: G = graphs.Grid2dGraph(3,5)
sage: vs, seq = VS.vertex_separation_BAB(G); vs
3
sage: vs == VS.width_of_path_decomposition(G, seq)
True

The vertex separation of circuit of order N≥2 is 1:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: D = digraphs.Circuit(10)
sage: vs, seq = VS.vertex_separation_BAB(D); vs
1
sage: vs == VS.width_of_path_decomposition(D, seq)
True

The vertex separation of cycle of order N≥3 is 2:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: G = graphs.CycleGraph(10)
sage: vs, seq = VS.vertex_separation_BAB(G); vs
2

The vertex separation of `MycielskiGraph(5)` is 10:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: G = graphs.MycielskiGraph(5)
sage: vs, seq = VS.vertex_separation_BAB(G); vs
10

Searching for any solution with width less or equal to `cut_off`:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: G = graphs.MycielskiGraph(5)
sage: VS.vertex_separation_BAB(G, cut_off=11)[0] <= 11
True
sage: VS.vertex_separation_BAB(G, cut_off=10)[0] <= 10
True
sage: VS.vertex_separation_BAB(G, cut_off=9)[0] <= 9
False

Testing for the existence of a solution with width strictly less than `upper_bound`:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: G = graphs.MycielskiGraph(5)
sage: vs, seq = VS.vertex_separation_BAB(G, upper_bound=11); vs
10
sage: vs, seq = VS.vertex_separation_BAB(G, upper_bound=10); vs
-1
sage: vs, seq = VS.vertex_separation_BAB(G, cut_off=11, upper_bound=10); vs
-1

Changing the parameters of the prefix storage:

sage: from sage.graphs.graph_decompositions import vertex_separation as VS
sage: G = graphs.MycielskiGraph(5)
sage: vs, seq = VS.vertex_separation_BAB(G, max_prefix_length=0); vs
10
sage: vs, seq = VS.vertex_separation_BAB(G, max_prefix_number=5); vs
10
sage: vs, seq = VS.vertex_separation_BAB(G, max_prefix_number=0); vs
10

sage.graphs.graph_decompositions.vertex_separation.vertex_separation_MILP(_G_, _integrality=False_, _solver=None_, _verbose=0_, _integrality_tolerance=0.001_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation_MILP "Permalink to this definition")

Compute the vertex separation of G and the optimal ordering of its vertices using an MILP formulation.

This function uses a mixed integer linear program (MILP) for determining an optimal layout for the vertex separation of G. This MILP is an improved version of the formulation proposed in [[SP2010]](https://doc.sagemath.org/html/en/reference/references/index.html#sp2010). See the [`module's documentation`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#module-sage.graphs.graph_decompositions.vertex_separation "sage.graphs.graph_decompositions.vertex_separation") for more details on this MILP formulation.

INPUT:

-   `G` – a Graph or a DiGraph
    
-   `integrality` – boolean (default: `False`); specify if variables xvt and uvt must be integral or if they can be relaxed. This has no impact on the validity of the solution, but it is sometimes faster to solve the problem using binary variables only.
    
-   `solver` – string (default: `None`); specify a Mixed Integer Linear Programming (MILP) solver to be used. If set to `None`, the default one is used. For more information on MILP solvers and which default solver is used, see the method [`solve`](https://doc.sagemath.org/html/en/reference/numerical/sage/numerical/mip.html#sage.numerical.mip.MixedIntegerLinearProgram.solve "(in Numerical Optimization v9.7)") of the class [`MixedIntegerLinearProgram`](https://doc.sagemath.org/html/en/reference/numerical/sage/numerical/mip.html#sage.numerical.mip.MixedIntegerLinearProgram "(in Numerical Optimization v9.7)").
    
-   `verbose` – integer (default: `0`); sets the level of verbosity. Set to 0 by default, which means quiet.
    
-   `integrality_tolerance` – float; parameter for use with MILP solvers over an inexact base ring; see [`MixedIntegerLinearProgram.get_values()`](https://doc.sagemath.org/html/en/reference/numerical/sage/numerical/mip.html#sage.numerical.mip.MixedIntegerLinearProgram.get_values "(in Numerical Optimization v9.7)").
    

OUTPUT:

A pair `(cost, ordering)` representing the optimal ordering of the vertices and its cost.

EXAMPLES:

Vertex separation of a De Bruijn digraph:

sage: from sage.graphs.graph_decompositions import vertex_separation
sage: G = digraphs.DeBruijn(2,3)
sage: vs, L = vertex_separation.vertex_separation_MILP(G); vs
2
sage: vs == vertex_separation.width_of_path_decomposition(G, L)
True
sage: vse, Le = vertex_separation.vertex_separation(G); vse
2

The vertex separation of a circuit is 1:

sage: from sage.graphs.graph_decompositions import vertex_separation
sage: G = digraphs.Circuit(6)
sage: vs, L = vertex_separation.vertex_separation_MILP(G); vs
1

sage.graphs.graph_decompositions.vertex_separation.vertex_separation_exp(_G_, _verbose=False_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.vertex_separation_exp "Permalink to this definition")

Return an optimal ordering of the vertices and its cost for vertex-separation.

INPUT:

-   `G` – a Graph or a DiGraph
    
-   `verbose` – boolean (default: `False`); whether to display information on the computations
    

OUTPUT:

A pair `(cost, ordering)` representing the optimal ordering of the vertices and its cost.

Note

Because of its current implementation, this algorithm only works on graphs on less than 32 vertices. This can be changed to 54 if necessary, but 32 vertices already require 4GB of memory.

EXAMPLES:

The vertex separation of a circuit is equal to 1:

sage: from sage.graphs.graph_decompositions.vertex_separation import vertex_separation_exp
sage: g = digraphs.Circuit(6)
sage: vertex_separation_exp(g)
(1, [0, 1, 2, 3, 4, 5])

sage.graphs.graph_decompositions.vertex_separation.width_of_path_decomposition(_G_, _L_)[](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#sage.graphs.graph_decompositions.vertex_separation.width_of_path_decomposition "Permalink to this definition")

Return the width of the path decomposition induced by the linear ordering L of the vertices of G.

If G is an instance of [`Graph`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph.html#module-sage.graphs.graph "sage.graphs.graph"), this function returns the width pwL(G) of the path decomposition induced by the linear ordering L of the vertices of G. If G is a [`DiGraph`](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/digraph.html#module-sage.graphs.digraph "sage.graphs.digraph"), it returns instead the width vsL(G) of the directed path decomposition induced by the linear ordering L of the vertices of G, where

vsL(G)=max0≤i<|V|−1|N+(L[:i])∖L[:i]|pwL(G)=max0≤i<|V|−1|N(L[:i])∖L[:i]|

INPUT:

-   `G` – a Graph or a DiGraph
    
-   `L` – a linear ordering of the vertices of `G`
    

EXAMPLES:

Path decomposition of a cycle:

sage: from sage.graphs.graph_decompositions import vertex_separation
sage: G = graphs.CycleGraph(6)
sage: L = G.vertices(sort=True)
sage: vertex_separation.width_of_path_decomposition(G, L)
2

Directed path decomposition of a circuit:

sage: from sage.graphs.graph_decompositions import vertex_separation
sage: G = digraphs.Circuit(6)
sage: L = G.vertices(sort=True)
sage: vertex_separation.width_of_path_decomposition(G, L)
1

[

Next

Rank Decompositions of graphs

](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/rankwidth.html)[

Previous

Tree decompositions



](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/tree_decomposition.html)

Copyright © 2005--2022, The Sage Development Team

Made with [Sphinx](https://www.sphinx-doc.org/) and [@pradyunsg](https://pradyunsg.me/)'s [Furo](https://github.com/pradyunsg/furo)

CONTENTS

-   -   [Exponential algorithm for vertex separation](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#exponential-algorithm-for-vertex-separation)
    -   [MILP formulation for the vertex separation](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#milp-formulation-for-the-vertex-separation)
    -   [Branch and Bound algorithm for the vertex separation](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#branch-and-bound-algorithm-for-the-vertex-separation)
    -   [Authors](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#authors)
    -   [Methods](https://doc.sagemath.org/html/en/reference/graphs/sage/graphs/graph_decompositions/vertex_separation.html#methods)