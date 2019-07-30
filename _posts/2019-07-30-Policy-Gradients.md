---
layout: post
title: "Policy Gradients - the REINFORCE Algorithm"
date: 2019-07-30
---

Riverrun, past Eve and Adam's, from swerve of shore to bend of bay, brings us by a commodius vicus of recirculation back to yet another set of notes for an IEEE journal club presentation [^1]. I have been spending a lot of time thinking and working on stochastic optimisation problems lately and I thought it would be interesting to have a look at an early paper on what has come to be called the REINFORCE Algorithm or "Policy Gradients" in the Reinforcement Learning literature with the explicitly named [Simple Statistical Gradient-Following Algorithms for Connectionist Reinforcement Learning](https://link.springer.com/content/pdf/10.1007%2FBF00992696.pdf). So at first glance this appears to be yet another rebranding of gradient ascent in machine learning, but it is a little bit more interesting than that. The crux of the algorithm actually turns out to be a really useful trick that has been rediscovered in multiple fields (in fact the divergence of fields is also independently interesting as Control Theory is Reinforcement Learning is Stochastic Optimisation is Simulation Optimisation, the only difference is dogma). It is also another great example of a theory result in Neural Networks that took a long time before the practical applications became feasible. Now you can build an agent to play pong from pixels with no knowledge of the game apart from defining a reward function and a control space (notes on implementation details in the video are [here](http://karpathy.github.io/2016/05/31/rl/)).

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=YOW8m2YGtRg" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>  

I won't wax lyrical about the algorithm, the paper discusses it well enough and there are already a million cv padding, bloodless, didactic blog posts about Policy Gradients on the internet. I will just talk a bit about the problem setup because I don't think that is super clear in the paper and say a little about the magic trick which makes it really cool.

## The Problem Setup

This is the central problem of control theory. It is zen like in its subtlety and profundity and as such also appears to contain zero information content. The universe has deigned that we can see some vector of states, $x$, and we have acess to some vector of control variables, $u$. Now the whole universe is distilled into the joint distribution over control variates and states, $p(x,u)$. Now the universe has also blessed us with a reward function that returns a scalar that depends on our particular place in state space, $f(x)$. Now the problem we are required to solve (or at least approximate) is

$$ u_{best} = \operatorname*{argmax}_u E(f(x)) $$

where the expectation is over the marginal distribution of states for a fixed value of the controls, i.e.

$$ E(f(x)) = \int_{-\infty}^{\infty}f(x)p_{u}(x)dx $$

That is it. If you can solve the above optimization problem, job is jobbed and we can all go home. But unfortunately optimization is really hard. If the distribution has blobbed out the details enough, maybe just doing gradient ascent to a local optima of the expected reward is good enough. It will always at least be better than where you started, $u_{0}$, i.e.

$$ u_{i+1} = u_{i} + \gamma\nabla_{u}E(f(x)) $$

where $\gamma$ is a magic step size. Now where this cunning plan all goes to shit is that the size of the control and state vectors are probably very large. Therefore any chance of actually doing the integral above over states is hopelessly doomed. A naive approach might be to simply take a Monte Carlo estimate of the expectation and then do finite differences for the gradient for steps in the control vector. This will work but is going to be rather slow for large control vectors (as you need to take a step in every direction and then do enough Monte Carlo samples to get a good estimate of the expectation). If only there were a better way...

## The Trick

So now comes the crux. If the distribution has nice properties we can take the gradient inside the expectation to derive a direct Monte Carlo estimator for the gradient (cough, cough, dominated convergence theorem, something, something), i.e.

$$ \nabla_{u}\int_{-\infty}^{\infty}f(x)p_{u}(x)dx = \int_{-\infty}^{\infty}f(x)\nabla_{u}p_{u}(x)dx $$

Now if we use the following identity in the above

$$ \nabla_{u}p_{u}(x) = \nabla_{u}log(p_{u}(x))p_{u}(x) $$

we can get

$$ \nabla_{u}E(f(x)) = \int_{-\infty}^{\infty}f(x)\nabla_{u}log(p_{u}(x))p_{u}(x)dx $$

Thus if we can draw samples from the marginal distribution, we can get the following Monte Carlo estimator

$$ \nabla_{u}E(f(x)) \approx \Sigma_{i=1}^{n}f(x_i)\nabla_{u}log(p_{u}(x_i))/n $$

Which turns out to be an efficient estimator for the gradient. This useful little fact has been independently derived a few times over the place and is called the "Likelihood Ratio Trick" or the "Score Function Trick".

## Enough Crapping On, What is this Paper About?

So, I will go through more on the night, but in the paper's version of this problem the "reinforcement signal" is the expected reward and what I have called control variables, they have called the weights of the neural network (which is a non-parametric representation of the distribution over states). I will say that my nomenclature is probably totally stupid (in my head you control the system by setting the weights, not stochastically firing the pong paddles). Also as this is intended to be an online method a good number of Monte Carlo samples in the gradient estimator is precisely one, i.e.

$$ \nabla_{u}E(f(x)) \approx f(x_i)\nabla_{u}log(p_{u}(x_i)) $$

So I guess you can sort of think of it as a greedy method where you take whatever samples you can get and update the weights to make the network do what you want right away. Also an offset to the reward is introduced, which you can think of a type of importance sampling for the gradient estimator. Simple. And it seems to sort of work.

## So What is Wrong with It?

You can probably guess. You are trying to learn a lot and you are just doing Gradient Ascent with a crappy gradient estimator so it can take a lot of samples to get a good solution. The usual issues with vanilla Gradient Ascent, i.e. how do you pick a step size (especially if a line search isn't an option), maybe I should be doing something like conjugate gradients, etc. Also, because it is greedy (and uses one sample!), you can easily end up in a really bad part of parameter space and never be able to recover. The extensions basically try to address these issues with suitably uninformative names (e.g. Actor-Critic Methods where the reward function is approximated so you can optimize against that instead and Proximal Policy Optimization where the gradient step is controlled in a trust region type deal via KL divergence).

## Anything Else You Would Like to Rant About?

I'll start with my usual rant about neural networks needing more priors. It is cool to learn everything from scratch but it doesn't practically make much sense if you can decompose the problem a bit before hand to save yourself heaps of samples. Maybe decouple the estimation and control parts of the problem and use pre trained models to improve the sample efficiency. In particular the terrible sample efficiency and the possibility of wandering into bad regions of state space during training basically make this method a no go for a real robot (without modification).
It really should be noted that the score function trick is a super useful way of estimating difficult gradients in lots of other domains too, not just for this type of optimisation problem. There is nothing explicitly "neural networky" about this method, you can apply it to other ML methods too as long as you can estimate the gradient of the log of the probability and draw samples. It is also interesting to think that Stochastic Gradient Descent in supervised learning can be interpreted as a REINFORCE algorithm (i.e. you adapt the weights based on a mini batch). It isn't particularly helpful to think this way, just interesting...

[^1]: a way a lone a last a loved a long the. One of the saddest experiences of my life to date was spending a long, long time trying to read Finnegans Wake. It was an epic struggle. Then one day my bookmark fell out and I was never able to find out where I was up to ever again.
