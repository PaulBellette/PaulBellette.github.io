---
layout: post
title: "Set Covering as a Linear Integer Program"
date: 2020-03-22
---

So, long time and no posts. I have a half written thing that I have been going back and forth on with no real commitment but I feel inspired to write this one. This is a slight departure from my normal all caps rants into a more gentle introduction to some basic ideas. This post is coming from me giving a far too quick explanation of how you can set up the Set Cover Problem as a Linear Integer Program at the end of a lecture I gave for the UQ Master of Data Science Math7501 course. I felt like I didn't give a super clear overview as I was rushing, so I aim to rectify that here with this post.

## The Set Cover Problem

The example I gave in the lecture as an example of the set cover problem was for planning a trip around Australia to try to see various animals with minimal stops. This seems so two weeks ago. The idea of travelling anywhere looks to be off the cards for a while now. So lets go for a more topical example. Lets say you are going to be stuck at home for a while with lots of time on your hands and there a bunch of classic movies that you want to see, that you have heard are really good. We can represent these movies as a set, say 

$$ U = \{\textrm{Paris Texas}, \textrm{Rope}, \textrm{Waltz with Bashir}, \textrm{Hunger}, \textrm{火垂るの墓},  \textrm{Blue Velvet}, \textrm{The Proposition}, \textrm{Mother}\} $$

Now also say you have a bunch of streaming services that you would need to sign up for to be able to watch these movies. Now the catch is that (as always) not one of the streaming services has all the movies that you want to watch. So say for example,

$$
\begin{align}
& \textrm{Netflix} =  \{\textrm{Paris Texas}, \textrm{Rope},  \textrm{火垂るの墓} \}\\
& \textrm{Prime} =  \{\textrm{Rope}, \textrm{Hunger}, \textrm{The Proposition}, \textrm{Blue Velvet}\}\\
& \textrm{Stan} =  \{\textrm{The Proposition}, \textrm{Mother}\}\\
& \textrm{Fetch} =  \{\textrm{Rope},  \textrm{Waltz with Bashir}, \textrm{Blue Velvet}, \textrm{Mother}\}\\
& \end{align}
$$

So the set cover problem can be set up as a decision problem or as an optimization problem. The decision problem is "can I sign up for $k$ streaming services such that I can see all the movies that I want to watch". The optimization version ask "what is the minimal number of streaming services I can sign up for such that I can watch all the movies that I want to see?". Now (hopefully if I set it up right) you can see that each stream service is a proper subset of $U$ and the union of the streaming services, which we will call $S$, is equal to $U$. So straight away you can see that a naive strategy would be enumerate all the subsets of $S$ and find the smallest one where the union of the subsets covers $U$. The problem is that this will involve $2^N$ operations, where $N$ is the number of subsets (4 streaming services in this case, which isn't that big for this particular example but can get very large for real problems making naive search practically impossible).

The idea behind the next part is to show how we can use the tools of linear algebra to represent this problem, which allows us to make use of powerful Linear Integer Programming software to greatly speed up the search for the optimal solution.

## The Set Cover Problem as Linear Algebra

To start with we can turn our subsets $S$ into a covering matrix. The idea here is that each row of the matrix will be a movie from $U$ and each column will be each streaming service. What we do is simply put a $0$ if the streaming service does not have the movie or a $1$ if the streaming service does have the movie. So keeping the order I have written for the set of movies $U$ and the set of streaming services $S$ we get the following matrix (which I really hope is actually correct given above).

$$ A = \begin{bmatrix}1 & 0 & 0 & 0\\1 & 1 & 0 & 1 \\ 0 & 0 & 0 & 1 \\0 & 1 & 0 & 0 \\1 & 0 & 0 & 0\\0 & 1 & 0 & 1\\0 & 1 & 1 & 0\\0 & 0 & 1 & 1 \end{bmatrix} $$

Now if we want to evaluate which movies a particular streaming service will supply we can simply multiply this matrix by a vector $x$ that has all zeros except for the element corresponding to the column of the coverage matrix for that particular streaming service (i.e. the first is Netflix, the second is Prime, etc). For example the "Netflix Vector" is

$$x = [1,0,0,0]$$

and if we take the Matrix by Vector product we get

$$ y = Ax = [1, 1, 0, 0, 1, 0, 0, 0] $$

i.e. Paris Texas, Rope and 火垂るの墓, which is what we expect. We can interpret this solution vector $y$ as the count of the number of times we have covered this particular movie with the streaming services selected in $x$. So to solve the covering problem we are constrained to have the elements of the vector $x$ as either zero or one and we need to have every element of $y$ greater or equal than one (so we can watch every movie in $U$). 

Now the last part is working out how do we evaluate the objective, i.e. we want to select the minimum number of streaming services. To do this we want to sum our $x$ vector. This can be simply achieved by taking the inner product of $x$ with a vector of all $1$'s. This has the advantage that we can generalize our problem if say we want to model the costs of the different services, we could just change the $1$'s for the price of each service.

So to summarize we can turn the set cover problem into a Linear Algebra problem, provided we restrict the elements of $x$ to be either $0$ or $1$. This is where the "integer" in the linear integer program comes from. To summarize the above as a set of equations, we want to

$$
\begin{align*}
& \textrm{Minimize    } c.x\\
& \textrm{Subject to    }  Ax \geq b\\
& \textrm{where}\\
& x_{i} \in \{0,1\} \textrm{ for } i = 1,...,N\\
& c_{j} = 1 \textrm{ for } j=1,...,M\\
& b_{i} = 1 \textrm{ for } i=1,...,N\\
\end{align*}
$$

So why do we bother to set it up in this form? Well we want to be able to use powerful Linear Integer Programming software libraries that have been developed. These libraries use sophisticated exact and heuristic algorithms to speed up the search for the optimal vector $x$. A famous algorithm of this type is [Branch and Bound](https://en.wikipedia.org/wiki/Branch_and_bound) which has an Australian connection via the idea's genesis in the work of the mathematician [Alison Harcourt](https://en.wikipedia.org/wiki/Alison_Harcourt). The two most notable libraries for solving Integer Linear Programs are CPLEX and Gurobi (which are unfortunately closed source so I won't bother linking to commercial websites for these). There are open source alternatives such as the c++ library [COIN-OR](https://github.com/coin-or/Clp), which I think (maybe) is what Mathematica uses underneath to solve these problems.

Challenge: Can you find the optimal combination of streaming services?

## Anything Else

We are not always interested in exact solutions to these types of problems, greedy algorithms can work very well in practice. Another fun idea is that we can also relax the integer program to a linear program and interpret the real valued vector elements obtained as probabilities and use a randomized algorithm to search for good candidate solutions. I hope that you can see that this problem has many useful applications to real problems (as stated in the lecture I have been encountering this problem at my day job in the context of solving rostering problems with constraints around who can do which job).
