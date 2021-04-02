---
title:  "Hamiltonian Monte Carlo - A Brief Sample"
layout: single
author_profile: true 
excerpt: "A small implementation of Hamilitonian Monte Carlo sampling using automatic differentiation in Python"
header:
  overlay_image: /assets/images/HMC7.png
  overlay_filter: rgba(140,140,140, 0.5)
  caption: "HMC samples of a probability distribution"
classes: wide
---

# Problem Statement - an optimisation viewpoint

Suppose we have some function that we wish to optimise, i.e locate the minimum of this function. 

One method would be to choose an initial point and take a random step in any location, if the function decreases then accept this point, and if the function evaluation is higher than before reject this sample. 

Intuitively this seems like an incredibly inefficient way of optimising a function, and mathematically it can be proven as well. 
As the number of dimensions of this optimisation problem increases, then the probability of stepping in a decreasing direction decreases exponentially. 

So how do we get around this? Well one of the first classes of optimisation that is taught is _gradient based_ optimisation. Taking the gradient of the function, and taking a step of a certain size in the opposite direction (_downhill_).

Now going one step further, we can also add _momentum_ to our gradient based optimisation. This allows our trajectory to continue moving in promising directions as it carries over a velocity term to subsequent iterations, it is completely analagous to a ball rolling down a hill complete with a friction term that ensures that momentum slowly decreases and the 'ball' comes to a stop.

# What has this got to do with sampling probability distributions?

Well it turns out sampling a probability distribution is surprisingly similar to the problem above, except _instead of locating the single best point, we look to find the most representative set of points that describe the distribution_. For example, high valued regions of the probability distrubution should intuitively have more 'points' sampled, as it is more likely that sampling this distribution, you will end up here. 

<p align="center">
<img src="/assets/images/sample.png" width="90%">
</p>

The _Metropolis Hastings_ algorithm is equivalent choosing an initial point, sampling in a random direction, and with a certain probability related to it's value, accepting or rejecting this sample. 
Statistically it allows for a representative sample, however it runs into dimensionality issues as mentioned above.

## This is where Hamiltonian Monte Carlo sampling comes in.

Instead of taking a random step, we just let a hypothetical ball roll down the probability distribution (which is upside down). We're moving towards areas of high probability, but because we don't induce any friction our ball just keeps rolling around. This motion is as a result of what is called _Hamiltonian mechanics_, hence the name. If the evaluation of the mechanics results in an error, i.e. numerical errors meaning the new sample has _gained_ total energy, then the sample is rejected.

_How do we induce randomness_? After each sample we can set the momentum to a random value, and thus we roll in a different direction. 

This allows us to gain a very good sample of a distribution, and because gradient information is incorporated, scales with dimension extremely well. 

The two hyper-parameters in HMC sampling are<img src="https://render.githubusercontent.com/render/math?math=\epsilon">, and <img src="https://render.githubusercontent.com/render/math?math=L">. The first describing the 'length' of sub-steps between samples through which we propogate Hamiltonian mechanics, and the second descibing how many of these steps we take before we induce a random momentum and complete an individual sample. 

Their influence and a visualisation of HMC sampling is shown below:

<p align="center">
<img src="/assets/images/HMC1.png" width="40%">
<img src="/assets/images/HMC2.png" width="40%">
</p>
Increasing L results in longer evaluations of Hamiltonian mechanics, of course less efficient as more gradient evaluations have to be taken per sample, but setting L too low will result behaviour too similar to the Metropolis Hastings algorithm and all the benefits will be lost: 

<p align="center">
<img src="/assets/images/HMC1.png" width="40%">
<img src="/assets/images/HMC12.png" width="40%">
</p>

Once we have decided on the right parameters, the number of samples can be increased and a representative set of samples from the probability distribution can be obtained.

<p align="center">
<img src="/assets/images/HMC4.png" width="80%">
</p>
<p align="center">
<img src="/assets/images/HMC5.png" width="80%">
</p>


<p align="center">
<img src="/assets/images/HMC6.png" width="80%">
</p>
<p align="center">
<img src="/assets/images/HMC7.png" width="80%">
</p>
<p align="center">
<img src="/assets/images/HMC8.png" width="80%">
</p>
<p align="center">
<img src="/assets/images/HMC9.png" width="80%">
</p>

<p align="center">
<img src="/assets/images/HMC10.png" width="80%">
</p>
<p align="center">
<img src="/assets/images/HMC11.png" width="80%">
</p>


# Implementation 

The above figures were produced with in Python using the Autograd library for automatic differention of the distribution. The code can be found on [Github](https://github.com/tomrsavage/hamiltonianmontecarlo). 