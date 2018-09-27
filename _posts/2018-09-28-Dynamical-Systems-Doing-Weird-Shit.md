---
layout: post
title: "Dynamical Systems Doing Weird Shit"
date: 2018-09-28
---

Work in Progress...

So this another set of notes for an IEEE journal club presentation. This time I have picked [Dynamical Systems That Sort Lists, Diagonalize Matrices, and Solve Linear Programming Problems](https://ieeexplore.ieee.org/document/194420). I think this paper is super kawaii! I'm not sure if it is a practical result as it stands but it is a really interesting idea. The main idea is to approximately solve a combinatorial optimization problems by solving a flow on a restricted continuous space that will have the "answer" as the final state at $t=\infty$. I came across this paper looking into graph isomorphism, so I will use that as a lead in example.

## Matching Graphs

Say you have two graphs $G$ and $H$ and they have no edge attributes apart from an arbitrary index. However, they do have the same number of edges and the same number of nodes. Hell they even have the same degree at every node. How can you tell if these two graphs actually are isomorphic? That is, the node indices have just been shuffled around but they are actually the same graph? Well if you have the corresponding adjacency matrices $A$ (of graph $G$)and $B$ (of graph $H$) then if there exists a permutation matrix $P$ such that

$$ P'AP = B $$

Then they are isomorphic. This seems like a disgusting optimisation problem though to search through permutations. It is in actually in the weird limbo land of being in NP but not known if it is in P or NP-Complete (a bit like factoring). So the idea is this, maybe if we go up a level from Permutation Matrices to Orthogonal Matrices we might be able to use our old friend calculus to help us out, since Permutation Matrices are a subset of Orthogonal Matrices. So let's solve the following minimization problem on Orthogonal Matrices

$$ J = \lVert \phi'A\phi - B \rVert_{F}^{2} + \lambda\lVert \phi'\phi - I \rVert^{2}_{F} $$

where $\phi$ is an Orthogonal Matrix, $\lambda$ is a Lagrange Multiplier to make sure that this is not a lie and $I$ is the identity matrix. The optimal value should be $0$ and $\phi$ should be a Permuation Matrix if we hit the bottom, unless we hit a funny bottom like if we have isospectral graphs that aren't isomorphic. 