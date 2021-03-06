---
layout: post
title:  "A Detailed Look at Backpropagation in Feedforward Neural Networks"
date:   2018-10-30
categories: [deep-learning]
tags: [deep-learning]
mathjax: true
---

The last few years have shown an enormous rise in the use of neural networks for supervised-learning tasks. This growth has been driven by multiple factors - exponentially more labeled data to train with, faster and cheaper GPUs (graphics processing units) that parallelize linear algebra operations used extensively by deep learning, as well as a better understanding of the process of neural network training.

At the same time, the core training algorithm used to train neural networks is still **backpropagation** and **gradient descent**. While there are many excellent frameworks like <a href="https://www.tensorflow.org/">TensorFlow</a> and <a href="https://pytorch.org/">PyTorch</a> that take care of the details for the modern machine learning practitioner, it is crucial to understand what they do under the hood. The first step in that journey is understanding what backpropagation actually is.

# Global View of the Training Process

One can view a neural network as a black box that maps certain input vectors $\vec{x}$ to output vectors $\vec{y}$. More formally, the neural network is a function, $f$:

$$\vec{y} = f(\vec{x})$$

$f$ depends on several underlying parameters, also known as weights, denoted by $\vec{w}$:

$$\vec{y} = f(\vec{x}; \vec{w})$$

The situation isn't unlike linear regression where the output $y$:

$$y = w_1 x_1 + w_2 x_2 + \ldots + w_n x_n = \vec{w}.\vec{x}$$

is a function of the input $\vec{x}$ with some parameters $\vec{w}$.

Time for some pedantry - technically both $\vec{x}$ and $\vec{w}$ are inputs to $f$ and this distinction between the "inputs" $\vec{x}$ and the "weights" $\vec{w}$ seems silly. The separation actually encodes the beautiful idea that $f$ actually denotes a *family* of functions, one for each particular $\vec{w}$ and the central duty of any machine learning algorithm is to pick one function out of the family so that it best describes/understands our data. One can make this more explicit by writing $f_{\vec{w}}(\vec{x})$ instead of $f(\vec{x}; \vec{w})$ but the latter is easier to write notationally.

The central problem then is the discovery of the correct $\vec{w}$. What does that even mean? Well, given a dataset with input vectors and the corresponding outputs (labels), one uses $f$ with random weights $\vec{w}$ to make predictions, measures the deviation between the predictions and labels and tweaks the weights to minimize the deviation.

As an example, one commonly used measure of deviation is **mean-squared error** which is especially useful for regression problems. Another one commonly used is **cross-entropy** (or **negative-log-likelihood**) which is used for classification problems. There are many more and you can/should write your own depending on the problem you are solving. For simplicity, we'll use mean-squared error below but the discussion is minimally changed if one uses a different error metric.

The mean-squared error measures the deviation between the label $y$ and the prediction $\hat{y}$ as:

$$error = \frac{(\hat{y}-y)^2}{2}$$

Dividing by 2 is purely a convention that will become clear later on. If the label and prediction agree, this error is 0 and the more they disagree, the higher the error. For a dataset, one would just average the errors:

$$C = \frac{1}{n} \Sigma_{i=0}^{n} \frac{(\hat{y}\_i-y_i)^2}{2}$$

where we introduced the symbol $C$ which stands for **cost**. The terms **cost**, **loss**, **error**, **deviance** are often used interchangeably but we'll stick to cost from now on. $n$ is the number of examples in the dataset and the symbol $\Sigma$ (capital "Sigma") denotes a sum over all the errors.

Since the predictions, $\hat{y}\_i = f(\vec{x}\_i; \vec{w})$ are functions of $\vec{w}$, $C$ actually depends on $\vec{w}$ as well:

$$C[\vec{w}] = \frac{1}{n} \Sigma_{i=0}^{n} \frac{(f(\vec{x}\_i; \vec{w})-y_i)^2}{2}$$

where we made $C[\vec{w}]$ denotes $C$'s dependence on $\vec{w}$ .

We would now pick some random $\vec{w}$, make predictions $f(\vec{x}\_{i}; \vec{w})$ and compute the cost $C[\vec{w}]$. Our next task is to tweak $\vec{w}$ and repeat the procedure so that $C[\vec{w}]$ decreases. Our end goal is to minimize the cost, $C[\vec{w}]$ and the set of weights $\vec{w}$ that would do that would define our final model.

The big question here is two-fold:
* How should we choose the initial weights, $\vec{w}^{(0)}$ (the "0" denotes "initial")?

* Once we compute $C[\vec{w}^{(t)}]$ with a given $\vec{w}^{(t)}$, how should we choose the next $\vec{w}^{(t+1)}$ so that **in the long run**, we decrease $C[\vec{w}]$?

There's some new notation above so let's take a moment to clearly define what we mean:

Think of the process of updating the weights $\vec{w}$ as a process that occurs once every second. At time $t=0$, we start with a randomly generated $\vec{w}^{(0)}$. At time $t$, the weights will be $\vec{w}^{(t)}$. We want a rule to go from $\vec{w}^{(t)}$ to $\vec{w}^{(t+1)}$, i.e. from time-step $t$ to time-step $t+1$.

One way to minimize $C[\vec{w}]$ is a "greedy" approach. Let's look at a simple example that is one-dimensional i.e. there's only one element in $\vec{w}$ called $w$, which is a real number:

{% include image.html url="/assets/backprop/gradientdescent.svg" description="Fig 1. A simple cost function in one variable i.e. with one weight" %}

This is a nice situation where there is exactly one minimum at $w\_{\*}$. Let's suppose, we are at $w\_{R}$ ($R$ denotes "right" and $L$ denotes "left"). We know we need to move to the left or in the negative direction. We also know the slope of the cost curve is positive ("pointing up") at $w\_R$. On the other hand, suppose we are at $w_L$. We need to move to the right or in the positive direction while the slope is negative ("pointing down") at $w_L$.

In other words:
* When the slope of the cost function is positive, we need to move the weight in the negative direction i.e. decrease the weight.
* When the slope is negative, we need to move the weight in the positive direction i.e. increase the weight.

Mathematically,

$$w^{(t+1)} = w^{(t)} - \text{(something)} \text{(sign of slope)}$$

where $\text{something}$ is a positive number (so it won't change the sign of the term) which signifies the magnitude of the change in $w^{(t)}$.

When the slope is positive, we get:

$w^{(t+1)} = w^{(t)} - \text{(positive)} \text{(positive)} = w^{(t)} - \text{positive}$

i.e. $w^{(t+1)} < w^{(t)}$ so we moved in the negative direction.

When the slope is negative, we get:

$w^{(t+1)} = w^{(t)} - \text{(positive)} \text{(negative)} = w^{(t)} + \text{positive}$

i.e. $w^{(t+1)} > w^{(t)}$ so we moved in the positive direction.

We still need to decide what $\text{something}$ is. It's usually taken to be proportional to the magnitude of the slope:

$$w^{(t+1)} = w^{(t)} - \eta \mid{\frac{dC[w^{(t)}]}{dw}}\mid \text{(sign of slope)}$$

where $\eta$ is a constant of proportionality called the **learning rate**, $\mid\frac{dC[w^{(t)}]}{dw}\mid$ is the absolute value of the slope (or derivative) at the point $w^{(t)}$. We don't need to separate out the magnitude of the slope and the sign of the slope and we can simply write:

$$w^{(t+1)} = w^{(t)} - \eta \frac{dC[w^{(t)}]}{dw} \label{graddesc}$$

This generalizes easily to a cost function depending on multiple weights $\vec{w}$. We just compute the derivative of the cost with respect to each element of $\vec{w}$ and update the weights according to equation $\ref{graddesc}$. In particular, if $\vec{w} = (w_1, w_2, \ldots, w_N)$ are the $N$ weights, we compute the partial derivatives, $\frac{\partial C}{\partial w_i}$ for each $i$ and update each weight as follows:

$$w_i^{(t+1)} = w_i^{(t)} - \eta \frac{\partial C[\vec{w}^{(t)}]}{\partial w_i} \label{multidimgd1}$$

This is usually written more succinctly as:

$$\vec{w}^{(t+1)} = \vec{w}^{(t)} - \eta \nabla C[\vec{w}^{(t)}] \label{multidimgd2}$$

but ignore the extra notation here for now. The two equations $\ref{multidimgd1}$ and $\ref{multidimgd2}$ are completely equivalent.

**The main takeaway from the above discussion is that we *really really* care about the derivatives of the cost with respect to the weights since we need them to update the weights to minimize the cost and to hopefully get a well-performing model. If we can find an efficient way to do so, it would make it possible to train neural networks on non-trivial datasets. That efficient way is backpropagation.**


In the discussion below, we'll assume mean-squared error and exactly one data point. Both of these assumptions are straightforward to remove.

Some other assumptions/prerequisites/notes before we start:

* You'll need some familiarity with matrices and matrix multiplication as well as differentiation from calculus (but no integration at all).
* Ideally, get a few sheets of paper, a pen and a quiet space and work through the calculations as you go along. Writing something out cements the material far more than just reading it.
* Unfortunately I don't know how to show the details without mathematics. Please don't be turned off by unusual symbols - they are just strokes on a piece of paper or pixels on a screen. There is often a debate about the importance of mathematical content in machine learning and deep learning. While it is true that one doesn't need to know the mathematical details to apply many of these techniques (at least at a basic level) and that many papers use mathematics to obscure instead of illuminate concepts, mathematics gives a precise and beautiful understanding of what these algorithms are doing. It is still our most direct probe into complex systems and most of all, it is fun.
* What you'll hopefully take away is that after all the fog clears, the simple act of calculating derivatives for this problem results in simple, iterative equations that let us train neural networks very efficiently.
* There'll be a follow-up entry on *implementing* backpropagation from scratch and tweaks to gradient descent.
* Lastly, backpropagation is probably deeply flawed. There are some big questions here - 1) do animal brains actually learn via a mechanism like backpropagation, 2) are there alternatives that lead to better solutions in far shorter amount of time and with small amounts of data, 3) what is the exact nature of the so-called cost landscape i.e. the behavior of the cost as a function of the weights. As you read the article below, I urge you to be bold and think of alternatives to backpropagation.


## Backpropagation I - linear activations

We will be working with a very simple network architecture in this section. The architecture is shown in figure 2 below:

{% include image.html url="/assets/backprop/nn_1.svg" description="Fig 2. A simple feedforward linear neural network" %} 

There is an input node taking a number $x_0$, two internal nodes with values $x_1$ and $x_2$ respectively and an output node with value $x_3$ (also denoted as $\hat{y}$, as before).

There are three weights: $w_{01}$, $w_{12}$, and $w_{23}$ respectively. You should read $w_{ij}$ as the weight "transforming the value at node i to the value at node j".

More precisely, forward propagation or inference or prediction in this architecture is:

$x_1 = w_{01} x_0$

$x_2 = w_{12} x_1$

$\hat{y} \equiv x_3 = w_{23} x_2$

We can substitude the values iteratively to get:

$x_3 = w_{23} x_2 = w_{23} w_{12} x_1 = w_{23} w_{12} w_{01} x_0$

In other words, given an input $x_0$, our function $f$ will predict $x_3$ defined above.

There is something silly going on here. Why would we have all these weights when we can define a new weight, say $w_{c} \equiv w_{23} w_{12} w_{01}$ (the "c" stands for combined) and define the following architecture going straight from the input to the output

{% include image.html url="/assets/backprop/nn_1_short.svg" description="Fig 3. Simplified neural network with combined weights" %}

$x_3 = w_c x_0$

You are absolutely right if you made that observation and it's a very important point. Just combining these "linear" nodes doesn't do anything. We need to add non-linearities to be able to learn arbitrarily complicated functions $f$. But for now, bear with me since this sections lays the groundwork for the next section where we introduce non-linearities into the network architecture.

Going back to our network, to execute gradient descent, we need to calculate all the derivatives of the cost function with respect to the weights.

If we only had one data point $x_0$ with our prediction $x_3$ and the actual value $y$, the cost would be

$C[\vec{w}] = C[w_{01}, w_{12}, w_{23}] = \frac{(x_3 - y)^2}{2}$

Expanding, we get

$C[\vec{w}] = \frac{1}{2} (w_{23} w_{12} w_{01} x_0 - y)^2$

We can now calculate the derivatives by using some basic calculus:

$\frac{\partial C}{\partial w_{23}} = \frac{1}{2} 2 (x_3 - y) \frac{\partial x_3}{\partial w_{23}}\\
= (x_3 - y) w_{12} w_{01} x_0\\
$

$\frac{\partial C}{\partial w_{12}} = (x_3-y) w_{23} w_{01} x_0$

$\frac{\partial C}{\partial w_{01}} = (x_3-y) w_{23} w_{12} x_0$

We see a couple of patterns here:

* There is one derivative for each weight.
* The factor of $\frac{1}{2}$ was useful because the derivative "pulls down" a factor of 2 from $(x_3-y)^2$ and the factors cancel out.
* Each derivative is proportional to $(x_3 - y)$ or the deviation between the prediction and the target/label. If the deviation is 0, then all the derivatives are 0 and there are no corrections to the weights during gradient descent, as should be the case.
* One can think of two "chains" - a forward chain and a backward chain.
	* Forward chains look like:
		* $x_0$
		* $w_{01} x_0$ (same as $x_1$)
		* $w_{12} w_{01} x_0$ (same as $x_2$)
		* $w_{23} w_{12} w_{01} x_0$ (same as $x_3$)
	* Backward chains look like:
		* $(x_3 - y)$
		* $w_{23} (x_3 - y)$
		* $w_{12} w_{23} (x_3 - y)$
		* $w_{01} w_{12} w_{23} (x_3 - y)$
	* Both forward and backward chains show up in the derivatives.

In other words, forward chains are what one would get if one walks from the left to the right of the network and multiplies the factors together. Backward chains are what one would get if one walked from the right to the left.

To make the above point clearer, let's rewrite the derivatives with the weights in order from left to right and any **missing weight is colored in red**.

$\frac{\partial C}{\partial w_{23}} = (x_3 - y) {\color{red} {w_{23}}} w_{12} w_{01} x_0\\
$

$\frac{\partial C}{\partial w_{12}} = (x_3-y) w_{23} {\color{red} {w_{12}}} w_{01} x_0$

$\frac{\partial C}{\partial w_{01}} = (x_3-y) w_{23} w_{12} {\color{red} {w_{01}}} x_0$


Let's introduce some more notation for the backward chains. We will use the Greek symbol "delta", $\delta$ since it stands for the English "d" for "difference" or "deviance" and is a conventional symbol used for $x_3-y$ which measures the error.

Define:

$\delta_0 = x_3-y$

$\delta_1 = w_{23} (x_3 - y) = {\color{green} {w_{23}\delta_0}}$

$\delta_2 = w_{12} w_{23} (x_3 - y) = {\color{green} {w_{12}\delta_1}}$

$\delta_3 = w_{01} w_{12} w_{23} (x_3 - y) = {\color{green} {w_{01} \delta_2}}$

Using these new symbols, we can write the derivatives in a very simple manner:

$\frac{\partial C}{\partial w_{23}} = \underbrace{(x_3 - y)}\_{\delta_0} {\color{red} {w_{23}}} \underbrace{w_{12} w_{01} x_0}\_{x_2} = \delta_0 x_2\\
$

$\frac{\partial C}{\partial w_{12}} = \underbrace{(x_3-y) w_{23}}\_{\delta_1} {\color{red} {w_{12}}} \underbrace{w_{01} x_0}\_{x_1} = \delta_1 x_1$

$\frac{\partial C}{\partial w_{01}} = \underbrace{(x_3-y) w_{23} w_{12}}\_{\delta_2} {\color{red} {w_{01}}} \underbrace{x_0}\_{x_0} = \delta_2 x_0$

In other words, we always get the combination $\delta_{A} x_{B}$ where $A+B=2$ and $B$ (subscript of $x$) is the source of the weight with respect to which we are taking the derivative. So,

$\frac{\partial C}{\partial w_{i,i+1}} = \delta_{2-i} x_i$

There is no magic about the "2". It is the number of hidden layers i.e. the number of nodes excluding the input and the output nodes. 

The main advantage here is that as one makes predictions, one has to calculate the forward chains - $x_1, x_2, x_3$ and once one calculates the deviation $(x_3 - y)$, calculating the backward chains is just an iterative multiplication by the weights but going in the reverse direction. So in two passes through the network, one can calculate all the derivatives.

To get a sense of why this is so promising, imagine we knew nothing about backpropagation and someone handed us the neural network in Fig 2. To calculate the derivatives, we could use finite differences. For example,

$$\frac{\partial C[w_{01}, w_{12}, w_{23}]}{\partial w_{01}} \approx \frac{C[w_{01} + \epsilon, w_{12}, w_{23}] - C[w_{01}-\epsilon, w_{12}, w_{23}]}{2\epsilon} $$

where $\epsilon$ is some pre-decided small number ($\approx$ stands for "approximately equal to"). This basically exploits the definition of a derivative:

$$\frac{df(x)}{dx} = \lim_{\epsilon\rightarrow 0} \frac{f(x+\epsilon) - f(x)}{\epsilon}$$

If $\epsilon$ is small enough, we'll have a reasonably accurate estimate of the derivative. Since we are numerically (as opposed to analytically) computing the derivative, $\frac{f(x+\epsilon) - f(x-\epsilon)}{2\epsilon}$ is better behaved but let's not worry about that. The main point is that to numerically evaluate the *approximation* of the derivative, we have to do forward propagation *twice* - once to calculate $C[w_{01} + \epsilon, w_{12}, w_{23}]$ and once to calculate $C[w_{01} - \epsilon, w_{12}, w_{23}]$. All this for just *one* damn derivative. If we have $N$ weights, we'll end up doing two forward propagations for each one to do a total of **$2N$** forward propagation passes!!!! and only get approximate derivatives! Compare that to the one forward and one backward pass we did with our iterative equation above to get the derivative without any numerical approximation errors.

So far so good but this is still just a linear neural network with nothing interesting going on. Let's add non-linearities into the mix and hope that this story repeats.

## Backpropagation II - non-linear activations

We will still maintain the same overall architecture. The new twist is an extra operation at each node.

A **linear** function $g(\vec{x})$ is any function with the following property:

$g(a\vec{x} + b\vec{y}) = ag(\vec{x}) + bg(\vec{y})$

where $a, b$ are constant real numbers and $\vec{x}, \vec{y}$ are $n$-dimensional vectors.

The simplest example of a linear function is $f(x) = \alpha x$. Then, 

$f(ax+by) = \alpha (ax+by) = a (\alpha x) + b (\alpha y) = a f(x) + b f(y)$.

An example of a function that is not linear (**non-linear**) is $f(x) = \sqrt{x}$. A simple counterexample suffices - $f(4) = 2, f(9) = 3$ but $f(4+9) = \sqrt{13} \neq f(4)+f(9)=5$.

Linear functions can be very useful and the whole subject of **linear algebra** studies mathematical objects called vector spaces and linear functions between them. But most real systems in the world are not linear. Think of height as a function of age - it doesn't increase by the same amount each year but instead stabilizes and even decreases with age. We want our neural network to be able to learn arbitrary non-linear functions. To enable this, we need to sprinkle some non-linear functions at various points throughout our architecture.

{% include image.html url="/assets/backprop/nn_2.svg" description="Fig 4. Feedforward neural network with non-linear functions $\sigma_i$ inserted." %}

In figure 4, each box represents one of our original nodes split into two operations. At each node, we now define two numbers:

$p_i$ is the value **before** the non-linear function is applied (p is for "pre")

$q_i$ is the value **after** the non-linear function is applied (q since it comes after p)

and $i$ denotes which layer/node we are talking about. The non-linear function, also commonly known as the **activation function** or just **activation** is denoted by $\sigma_i$. This can potentially be different at every single layer/node.

Forward propagation is now modified with every original forward propagation equation split into two:

$p_0 = x_0 \text(input) \rightarrow q_0 = \sigma_0(p_0)$

$p_1 = w_{01} q_0 \rightarrow q_1 = \sigma_1(p_1)$

$p_2 = w_{12} q_1 \rightarrow q_2 = \sigma_2(p_2)$

$p_3 = w_{23} q_2 \rightarrow q_3 = \sigma_3(p_3) \text(output)$

The input $x_0$ is now denoted by $p_0$ for notational consistency. $p_0$ is now fed to the activation function $\sigma_0$ to get $q_0$. $q_0$ is now the input to the second layer. This process repeats till we get $q_3$ at the end which is the output of the model.

As a special case, consider $\sigma_i(x) = id(x) = x$ where $id$ denotes the identity function that maps every input to itself - $id(x) = x$. In that case, $p_i = q_i$ and we get our old linear neural network back.

To be more explicit about the output $q_3$'s dependence on the weights, we can combine the equations:

$q_3 = \sigma_3(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0))))$

As before, we compute the cost function to compare the output to the label:

$C[\vec{w}] = C[w_{01}, w_{12}, w_{23}] = \frac{(q_3-y)^2}{2}$

or more explicitly:

$C[\vec{w}] = \frac{(\sigma_3(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0))))-y)^2}{2}$

Let's take a step back and realize that we really haven't done anything very different. All we did was add 4 activations to our neural network, compute the output and evaluate the cost to see how well we did. As before what we really care about are the derivatives of the cost with respect to the weights so we can do gradient descent.

Using the chain rule from calculus, we can explicitly compute the derivatives (and write all the terms explicitly for clarity):

$\frac{\partial C}{\partial w_{23}} = \underline{(q_3-y)} \space \underline{\sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0))))} \space \underline{\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))}$

$\frac{\partial C}{\partial w_{12}} = \underline{(q_3-y)} \space \underline{\sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0))))} \space \underline{w_{23}}\space \underline{\sigma_2'(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))} \space \underline{\sigma_1(w_{01}\sigma_0(p_0))}$

$\frac{\partial C}{\partial w_{01}} = \underline{(q_3-y)}\space \underline{\sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0))))} \space \underline{w_{23}} \space \underline{\sigma_2'(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))} \space \underline{w_{12}} \space \underline{\sigma_1'(w_{01}\sigma_0(p_0))}\space\underline{\sigma_0(p_0)} $

Here $\sigma'(x)$ is short-hand notation for $\frac{d\sigma}{dx}$ to prevent the notation from getting heavy. The underlines are to delineate various terms that show up.

This is promising! We can already do some sanity checks and make a few observations:

Sanity checks:
* All the derivatives are proportional to $(q_3-y)$. In other words, if your prediction is exactly equal to the label, there are no derivatives and hence no gradient descent to do which is precisely what one would expect.
* If we replace all the activations by the identity function, $id(x) = x$ with $id'(x) = 1$, then we can replace the derivatives with 1, all the $q_i = p_i = x_i$ and we recover the derivatives for Case I without the activation functions.

Some observations:
* We still get both forward chains and backward chains:
	* Forward chains now look like:
		* ${\color{blue} {p_0}}$
		* $\sigma_0(p_0) = {\color{blue} {q_0}}$
		* $w_{01} \sigma_0(p_0) = {\color{blue} {p_1}}$
		* $\sigma_1(w_{01} \sigma_0(p_0)) = {\color{blue} {q_1}}$
		* $w_{12} \sigma_1(w_{01} \sigma_0(p_0)) = {\color{blue} {p_2}}$
		* $\sigma_2(w_{12} \sigma_1(w_{01} \sigma_0(p_0))) = {\color{blue} {q_2}}$
		* $w_{23} \sigma_2(w_{12} \sigma_1(w_{01} \sigma_0(p_0))) = {\color{blue} {p_3}}$
		* $\sigma_3(w_{23} \sigma_2(w_{12} \sigma_1(w_{01} \sigma_0(p_0)))) = {\color{blue} {q_3}}$
		* These are just the terms forward propagation generates.
	* Backward chains:
		* $(q_3-y)$
		* $(q_3 - y) \sigma_3'(p_3)$
		* $(q_3 - y) \sigma_3'(p_3) w_{23}$
		* $(q_3 - y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2)$
		* $(q_3 - y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2) w_{12}$
		* $(q_3 - y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2) w_{12} \sigma_1'(p_1)$
		* $(q_3 - y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2) w_{12} \sigma_1'(p_1) w_{01}$
		* $(q_3 - y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2) w_{12} \sigma_1'(p_1) w_{01} \sigma_0'(p_0)$
		* These now have derivatives and if $\sigma_i(x) = id(x)$, we can replace the derivatives by 1 and recover the backward chains from Case I.

As before, let's rewrite the derivatives with missing terms highlighted in ${\color{red} {red}}$.

$\frac{\partial C}{\partial w_{23}} = (q_3-y) \sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))) {\color{red} {w_{23}}} \sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))$

$\frac{\partial C}{\partial w_{12}} = (q_3-y) \sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))) w_{23} \sigma_2'(w_{12}\sigma_1(w_{01}\sigma_0(p_0))) {\color{red} {w_{12}}} \sigma_1(w_{01}\sigma_0(p_0))$

$\frac{\partial C}{\partial w_{01}} = (q_3-y) \sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))) w_{23} \sigma_2'(w_{12}\sigma_1(w_{01}\sigma_0(p_0))) w_{12} \sigma_1'(w_{01}\sigma_0(p_0)) {\color{red} {w_{01}}} \sigma_0(p_0) $

Define new symbols for the backward chains:

$\delta_0 = (q_3-y) \sigma_3'(p_3)$

$\delta_1 = (q_3-y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2) = {\color{green} {\delta_0 w_{23} \sigma'(p_2)}}$

$\delta_2 = (q_3-y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2) w_{12} \sigma_1'(p_1) = {\color{ green} {\delta_1 w_{12} \sigma'(p_1)}}$

$\delta_3 = (q_3 - y) \sigma_3'(p_3) w_{23} \sigma_2'(p_2) w_{12} \sigma_1'(p_1) w_{01} \sigma_0'(p_0) = {\color{green} {\delta_2 w_{01} \sigma'(p_0)}}$

Using these, we can rewrite the derivatives:

$\frac{\partial C}{\partial w_{23}} = \underbrace{(q_3-y) \sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0))))}\_{\delta_0} \space {\color{red} {w_{23}}} \space \underbrace{\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))}\_{q_2} = \delta_0 q_2$

$\frac{\partial C}{\partial w_{12}} = \underbrace{(q_3-y) \sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))) w_{23} \sigma_2'(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))}\_{\delta_1} \space {\color{red} {w_{12}}} \space \underbrace{\sigma_1(w_{01}\sigma_0(p_0))}\_{q_1} = \delta_1 q_1$

$\frac{\partial C}{\partial w_{01}} = \underbrace{(q_3-y) \sigma_3'(w_{23}\sigma_2(w_{12}\sigma_1(w_{01}\sigma_0(p_0)))) w_{23} \sigma_2'(w_{12}\sigma_1(w_{01}\sigma_0(p_0))) w_{12} \sigma_1'(w_{01}\sigma_0(p_0))}\_{\delta_2} \space {\color{red} {w_{01}}} \space \underbrace{q_0}\_{q_0} = \delta_2 q_0$

As before, we get the same pattern:

$\frac{\partial C}{\partial w_{i,i+1}} = \delta_{2-i} q_i$

which is gratifying. As before, during the forward pass, we incrementally calculate $q_1, q_2, q_3$ and then we iteratively do a backward pass and calculate $\delta_0, \delta_1, \delta_2$.

## Backpropagation III - linear activations + multi-node layers

In practice, neural networks with one node per layer are not very helpful. What we really want is to put multiple nodes at each layer to get the classic feedforward neural network shown below. For now, as in Section I, we won't include non-linear activations.

{% include image.html url="/assets/backprop/nn_3.svg" description="Fig 5. Multi-node linear feedforward network." %}

Forward propagation in this architecture is:

$x_1 = W_{01} x_0$

$x_2 = W_{12} x_1$

$x_3 = W_{23} x_2$

where the $W_{ij}$ are matrices of weights. We used $w_{ij}$ to refer to an individual weight, like the ones in sections I and II and we'll use $W_{ij}$ to refer to the **weight matrix** that takes us from layer $i$ to layer $j$. $x_i$ are now vectors and ideally should be written as $\vec{x}\_i$ but we'll omit the vector sign.

To be more explicit, 

$x_0 = \begin{bmatrix}
x_1^{(0)} \\\
x_2^{(0)} \\\
\vdots \\\
x_n^{(0)}
\end{bmatrix}$

and,

$W_{01} = \begin{bmatrix}
w_{11}^{(01)} & w_{12}^{(01)} & \ldots & w_{1n}^{(01)} \\\
w_{21}^{(01)} & w_{22}^{(01)} & \ldots & w_{2n}^{(01)} \\\
\vdots \\\
w_{m1}^{(01)} & w_{m2}^{(01)} & \ldots & w_{mn}^{(01)} \\\
\end{bmatrix}$

and $x_1 = \underbrace{\begin{bmatrix}
x_1^{(1)} \\\
x_2^{(1)} \\\
\vdots \\\
x_m^{(1)}
\end{bmatrix}}\_{(m,1)}
= \underbrace{\begin{bmatrix}
w_{11}^{(01)} & w_{12}^{(01)} & \ldots & w_{1n}^{(01)} \\\
w_{21}^{(01)} & w_{22}^{(01)} & \ldots & w_{2n}^{(01)} \\\
\vdots \\\
w_{m1}^{(01)} & w_{m2}^{(01)} & \ldots & w_{mn}^{(01)} \\\
\end{bmatrix}}\_{(m,n)}
\underbrace{\begin{bmatrix}
x_1^{(0)} \\\
x_2^{(0)} \\\
\vdots \\\
x_n^{(0)}
\end{bmatrix}}\_{(n,1)}
=\underbrace{\begin{bmatrix}
w_{11}^{(01)} x_1^{(0)}  + w_{12}^{(01)} x_2^{(0)} + \ldots + w_{1n}^{(01)} x_n^{(0)} \\\
w_{21}^{(01)} x_1^{(0)}  + w_{22}^{(01)} x_2^{(0)} + \ldots + w_{2n}^{(01)} x_n^{(0)} \\\
\vdots \\\
w_{m1}^{(01)} x_1^{(0)}  + w_{m2}^{(01)} x_2^{(0)} + \ldots + w_{mn}^{(01)} x_n^{(0)} \\\
\end{bmatrix}}\_{(m,1)}$

The superscripts denote the object a number belongs to. So, $x_{k}^{(0)}$ is the $k$th element of $x_0$ and $w_{ij}^{(01)}$ is the element in the $i$th row and $j$th column of $W_{01}$. The **dimensions** of the various objects are highlighted under the objects. $(m,n)$ means an object has $m$ rows and $n$ columns. We will often write $\text{dim}(A) = (m,n)$ to denote the dimension of object $A$.

We can combine these equations to write:

$x_3 = W_{23}W_{12}W_{01}x_0$

As in section I, there's still the same silliness going on. Why not define $W_c = W_{23}W_{12}W_{01}$ which is just another matrix and do gradient descent on the elements of $W_c$ directly. As before though, we are preparing to introduce non-linear activations in the next section.

In principle, we haven't done anything radically new. We just need to compute a cost and then find the derivatives with respect to each individual weight. Recall that we were using the mean-squared error metric as a cost function. The only difference is that now the output itself might be a vector:

$y = (y_1, y_2, \ldots, y_n)$

i.e. there are $n$ labels and the output vector $x_3$ also has $n$ dimensions. So the cost would just be a sum of mean-squared errors for every element in $y$ and $x_3$:

$C = \frac{1}{2} [(x_1^{(3)}-y_1)^2 + (x_{2}^{(3)}-y_2)^2 + \ldots + (x_{n}^{(3)}-y_n)^2]$

where $x_3 = (x_{1}^{(3)}, x_{2}^{(3)}, \ldots, x_{n}^{(3)})$

A more concise way of writing this is as follows:

$C[W_{01}, W_{12}, W_{23}] = \frac{(x_3-y)^T(x_3-y)}{2}$

where $x^T$ denotes the transpose of a vector. So,

$x = \begin{bmatrix} 
x_1 \\\
x_2 \\\
\vdots \\\
x_n \end{bmatrix} \implies x^T = [x_1, x_2, \ldots, x_n]
$

More generally, given a matrix $A$ with elements $a_{ij}$, the transpose of a matrix, denoted by $A^T$ has elements where the rows and columns are flipped. So

$(A^T)\_{ij} = a_{ji}$

Note that the indices on the right-hand side are flipped. An example will make this clear:

$A = \begin{bmatrix}
a_{11} & a_{12} & a_{13} \\\
a_{21} & a_{22} & a_{23} \\\
\end{bmatrix}
\implies
A^T = \begin{bmatrix}
a_{11} & a_{21}\\\
a_{12} & a_{22}\\\
a_{13} & a_{23}\\\
\end{bmatrix}
$

So, the $ij$-th element of $A^T$ is the $ji$-th element of A. In other words, $A^T$ takes every row of $A$ and makes it into a column. Moreover, transposing a matrix changes its dimensions. If $\text{dim}(A) = (m,n)$ then $\text{dim}(A^T) = (n,m)$.

Going back to the cost function:

$C = \frac{(x_3-y)^T(x_3-y)}{2} = \frac{1}{2} [x^{(3)}\_1-y_1, x^{(3)}\_2-y_2, \ldots, x^{(3)}\_n-y_n] \begin{bmatrix}
x^{(3)}\_1-y_1 \\\
x^{(3)}\_2-y_2 \\\
\vdots \\\
x^{(3)}\_n-y_n \\\
\end{bmatrix}$

$\implies C = \frac{1}{2} [(x_1^{(3)}-y_1)^2 + (x_{2}^{(3)}-y_2)^2 + \ldots + (x_{n}^{(3)}-y_n)^2]$

showing that the first form is just a more concise way of writing our original cost function.

Expanding, we get:

$C = \frac{(x_3-y)^T(x_3-y)}{2} = \frac{1}{2}\[x_3^Tx_3 - x_3^Ty - y^Tx_3 + y^Ty\]$

The only term that doesn't depend on the weights matrices is $y^Ty$ and is a constant once the dataset is fixed (i.e. the labels are fixed). So we can neglect this term from here on since it'll never contribute to our derivatives.

Also, $x_3^Ty = y^Tx_3$ since they are both just dot products between the same pair of vectors. More explicitly, if $x_3 = (a_1 a_2 \ldots a_n)$ and $y = (b_1 b_2 \ldots b_n)$, then

$x_3^Ty = [a_1 a_2 \ldots a_n]
\begin{bmatrix}
b_1 \\\
b_2 \\\
\vdots \\\
b_n
\end{bmatrix}
= a_1 b_1 + a_2 b_2 + \ldots a_n b_n$

and 

$y^T x_3 = [b_1 b_2 \ldots b_n]
\begin{bmatrix}
a_1 \\\
a_2 \\\
\vdots \\\
a_n
\end{bmatrix}
= b_1 a_1 + b_2 a_2 + \ldots b_n a_n$

The two values are the same.

So, we can rewrite the cost 

$$C[W] = \frac{1}{2}[x_3^Tx_3 - 2 y^Tx_3] = \frac{x_3^Ty}{2} - y^Tx_3 \label{multidimcost}$$

To reiterate:
* "$=$" is being misused here since we completely dropped the term $y^Ty$ BUT since we are only using $C$ to find the derivatives for gradient descent and the dropped term doesn't contribute, it doesn't matter. If it makes you more comfortable, you could define a new cost $C' = C - y^Ty$ and since minimizing a function $f$ is equivalent to minimizing $f + \text{constant}$, minimizing $C'$ and $C$ is equivalent in the sense that they will result in the same set of minimizing weights.
* We combined $x_3^Ty$ and $y^T x_3$ since they are equal (hence the factor of 2).

Good progress! We are minimizing equation $\ref{multidimcost}$. We can compute the derivative with respect to every matrix element $w^{(ij)}\_{ab}$ of every matrix $W_{ij}$ and do gradient descent on each one:

$w_{ab}^{(ij), (t+1)} = w_{ab}^{(ij), (t)} - \eta \frac{\partial C}{\partial w_{ab}^{(ij), (t+1)}}$

We are using the same notation as in sections I and II. In $w_{ab}^{(ij), (t+1)}$, the $t$ refers to the step in gradient descent, $(ij)$ refers to the matrix $W_{ij}$ that the weight comes from and $ab$ refers to the matrix element, i.e. row $a$ and column $b$. This horrible tragedy of notational burden is 1) very annoying, 2) absolutely devoid of any insight. Sure we can compute this mess and maybe even elegantly but unlike sections I and II, there seem to be no nice backward chains here. To prepare a nice meal, one has to sometimes do a lot of "prep" i.e. preparation of ingredients and "pre-processing" them. Using mathematics to understand and gain insights is no different. So we'll take a nice de-tour to introduce the idea of **matrix derivatives**.

### Aside: Matrix derivatives

Instead of writing the gradient descent update equation for each weight, we would like to write it for each weight matrix:

$W_{ij}^{(t+1)} = W_{ij}^{(t)} - \eta \frac{\delta C}{\delta W_{ij}^{(t)}}$

where we have introduced $\frac{\delta C}{\delta W_{ij}^{(t)}}$, the derivative of the cost with respect to a matrix! For the above equation to be well-defined, $\frac{\delta C}{\delta W_{ij}^{(t)}}$ would need to have dimensions of $W_{ij}$ and would be defined as:

$\frac{\delta C}{\delta W_{ij}} = \begin{bmatrix}
\frac{\partial C}{\partial w_{11}^{(ij)}} & \frac{\partial C}{\partial w_{12}^{(ij)}} & \ldots & \frac{\partial C}{\partial w_{1n}^{(ij)}} \\\
\frac{\partial C}{\partial w_{21}^{(ij)}} & \frac{\partial C}{\partial w_{22}^{(ij)}} & \ldots & \frac{\partial C}{\partial w_{2n}^{(ij)}} \\\
\vdots \\\
\frac{\partial C}{\partial w_{m1}^{(ij)}} & \frac{\partial C}{\partial w_{m2}^{(ij)}} & \ldots & \frac{\partial C}{\partial w_{mn}^{(ij)}} \\\
\end{bmatrix}$

Our end goal is to deduce what rules such matrix derivatives would follow. To do so, we'll have to get our hands dirty but only once. Once the rules are derived, we can forget all the details and blindly differentiate expressions with matrices.

Let's start with one of the terms in the cost function.

#### Cost linear in weights

Let's start with a simple cost function:

$C[A] = y^TAx$

where $x, y$ are vectors and $A$ is a matrix. More precisely,

$\text{dim}(x) = (n,1)$

$\text{dim}(A) = (m,n)$

$\text{dim}(y) = (m,1)$

Why are the dimensions important? Because the product $O_1O_2$ is only defined when

$\text{number of columns of } O_1 = \text{number of rows of } O_2$

and the dimensions of $O_1O_2$ will be:

$(\text{number of rows of }O_1, \text{number of columns of }O_2)$

A more concise way of writing this is:

$\text{dim}(O_1) = (n_1, k)$

$\text{dim}(O_2) = (k, m_2)$

$\implies \text{dim}(O_1O_2) = (n_1, m_2)$

For our case,

$C[A] = \underbrace{y^T}\_{(1,m)}\underbrace{A}\_{(m,n)}\underbrace{x}\_{(n,1)}$

and $\text{dim}(C[A]) = (1,1)$ i.e. it's just a number which is what we expected to get for the cost.

Our notation for the cost:

$C[A]$

betrays our intention to keep $x, y$ fixed and minimize $C$ as a function of the elements of $A$. We will still use gradient descent which requires that we compute the derivatives of $C$ with respect to the elements of $A$.

$A = \begin{bmatrix}
	a_{11} & a_{12} & \ldots & a_{1n} \\\
	a_{21} & a_{22} & \ldots & a_{2n} \\\
	\vdots \\\
	a_{m1} & a_{m2} & \ldots & a_{mn} \\\
\end{bmatrix}$

Once we have the derivatives:

$\frac{\partial C}{\partial a_{ij}}$

we can update the elements of $A$:

$a_{ij}^{(t+1)} = a_{ij}^{(t)} - \eta \frac{\partial C}{\partial a_{ij}^{(t)}}$

Instead let's combine the derivatives in a matrix:

$\frac{\delta C}{\delta A} \equiv \begin{bmatrix}
	\frac{\partial C}{\partial a_{11}} & \frac{\partial C}{\partial a_{12}} & \ldots & \frac{\partial C}{a_{1n}} \\\
	\frac{\partial C}{\partial a_{21}} & \frac{\partial C}{\partial a_{22}} & \ldots & \frac{\partial C}{a_{2n}} \\\
	\vdots \\\
	\frac{\partial C}{\partial a_{m1}} & \frac{\partial C}{\partial a_{m2}} & \ldots & \frac{\partial C}{a_{mn}} \\\
\end{bmatrix}$

where $\text{dim}(A) = \text{dim}(\frac{\delta C}{\delta A}) = (m,n)$

We can then write:

$A^{(t+1)} = A^{(t)} - \eta \frac{\delta C}{\delta A^{(t)}}$

i.e. update the whole matrix in one go! Please note that the superscript $t$ is for the time-step NOT tranpose. For tranpose, we always use a *capital* T.

We also know that $C$ is linear in the elements of $A$ (more on this below) and so the derivatives should not depend on $A$ - just like the derivative of the linear function, $f(x) = ax + b$ with respect to x, $\frac{df}{dx} = a$ doesn't depend on $x$. So, $\frac{\delta C}{\delta A}$ can only depend on $x,y$ and the only way to construct a matrix of dimension $(m,n)$ from $x$ and $y$ is 

$\underbrace{y}\_{(m,1)}\underbrace{x^T}\_{(1,n)} = \begin{bmatrix}
y_1 \\\
y_2 \\\
\vdots \\\
y_m
\end{bmatrix}
\begin{bmatrix}
x_1 & x_2 & \ldots & x_n \\\
\end{bmatrix} = \begin{bmatrix}
y_1 x_1 & y_1 x_2 & \ldots y_1 x_n \\\
y_2 x_1 & y_2 x_2 & \ldots y_2 x_n \\\
\vdots \\\
y_m x_1 & y_m x_2 & \ldots y_md x_n \\\
\end{bmatrix}$

Maybe all this is just too general and hand-wavy. After all, couldn't we multiply $yx^T$ by a constant and still get something with dimension $(m,n)$. That's true! So, let's compute the derivative matrix explicitly to convince ourselves.

$C[A] = \begin{bmatrix}
	y_1 & y_2 & \ldots & y_m \\\
\end{bmatrix}
\begin{bmatrix}
	a_{11} & a_{12} & \ldots & a_{1n} \\\
	a_{21} & a_{22} & \ldots & a_{2n} \\\
	\vdots \\\
	a_{m1} & a_{m2} & \ldots & a_{mn} \\\
\end{bmatrix}
\begin{bmatrix}
	x_1 \\\
	x_2 \\\
	\vdots \\\
	x_n
\end{bmatrix}$

$C[A] = \begin{bmatrix}
	y_1 & y_2 & \ldots & y_m \\\
\end{bmatrix}
\begin{bmatrix}
	a_{11} x_1 + a_{12} x_2 + \ldots + a_{1n} x_n \\\
	a_{21} x_1 + a_{22} x_2 + \ldots + a_{2n} x_n \\\
	\ldots \\\
	a_{m1} x_1 + a_{m2} x_2 + \ldots + a_{mn} x_n \\\
\end{bmatrix} \\\ = (y_1 a_{11} x_1 + y_1 a_{12} x_2 + \ldots y_1 a_{1n} x_n) + (y_2 a_{21} x_1 + y_2 a_{22} x_2 + \ldots y_2 a_{2n} x_n) + \ldots + (y_m a_{m1} x_1 + y_m a_{m2} x_2 + \ldots y_m a_{mn} x_n)$

If we look closely at the last line, all the terms are of the form $y_i a_{ij} x_j$ (which is exactly how one writes matrix multiplication). So, we could write this as:

$C[A] = \Sigma_{i=1}^{m}\Sigma_{j=1}^{n} y_i a_{ij} x_j$

We also introduce the so-called Einstein (yes, the same Einstein you are thinking about) notation here now. We drop the summation sign, $\Sigma$ and write:

$C[A] = y_i a_{ij} x_j$

with the convention that any index that repeats twice is to be summed over. Since i appears twice - once with $y$ and once in $a_{ij}$ and j appears twice - once with $a_{ij}$ and once with $x_j$, they both get summed over the appropriate range. This way we don't have to write the summation sign each way.

To be clear, $y_i a_{ij} x_j$ is the same as $\Sigma_{i=1}^{m}\Sigma_{j=1}^{n} y_i a_{ij} x_j$ using the Einstein notation. Also, it doesn't matter what we call the repeated index so:

$y_i a_{ij} x_j = y_{bob} a_{bob,nancy} x_{nancy}$

It doesn't matter at all what we can the indices. All that matters is that repeated indices get summed over.

Great! so we computed an explicit form of $C$ and now we want derivatives with respect to $a_{kl}$ where $k,l$ are just indices denoting row k and column l. 

$\frac{\partial C}{\partial a_{kl}} = \frac{\partial}{\partial a_{kl}} [y_i a_{ij} x_j]$

We define:

$\delta_{a,b} = \begin{cases}
1, \text{if } a=b \\\
0, \text{otherwise} \\\
\end{cases}$

Then, 

$\frac{\partial C}{\partial a_{kl}} = y_i x_j \frac{\partial a_{ij}}{\partial a_{kl}}$

since $y_i, x_j$ don't depend on $a_{kl}$.

Now, 

$\frac{\partial a_{ij}}{\partial a_{kl}} = \begin{cases}
1, \text{if } i=k, j=l \\\
0, \text{otherwise}
\end{cases}$

Another way of writing this is:

$\frac{\partial a_{ij}}{\partial a_{kl}} = \delta_{i,k}\delta_{j,l}$

So,

$\frac{\partial C}{\partial a_{kl}} = y_i x_j \frac{\partial a_{ij}}{\partial a_{kl}} = y_i x_j \delta_{i,k}\delta_{j,l}$

But since repeated indices are summed over, when $i=k$ and when $j=l$, we get:

$\frac{\partial C}{\partial a_{kl}} = y_k x_l$

which is exactly the $(k,l)$ element of $yx^T$. So we just showed through explicit calculation that:

$$\boxed{C = y^T A x \implies \frac{\delta C}{\delta A} = y x^T}$$

the same result we got earlier by looking at various dimensions.

This can be used in gradient descent as:

$A^{(t+1)} = A^{(t)} - \eta \frac{\delta C}{\delta A^{(t)}} = A^{(t)} - \eta y x^T$

Now (anticipating future use), what if 

$C[A, B] = y^T A B x$

is our cost function. Is there an easy way to calculate $\frac{\delta C}{\delta A}$ and $\frac{\delta C}{\delta B}$? You bet there is!

Let's start with $\frac{\delta C}{\delta A}$.

We can define $x' = Bx$ to get $C = y^T A x'$:

$C = y^T A \underbrace{B x}\_{x'} = y^T A x'$

We know $\frac{\delta C}{\delta A} = y x'^T$ from our earlier result and we can just replace $x'$ to get:

$\frac{\delta C}{\delta A} = y x'^T = y (Bx)^T = y x^T B^T$ using the fact $(AB)^T = B^TA^T$.

On to $\frac{\delta C}{\delta B}$. We can use a similar trick.

$C = y^T A B x = (A^Ty)^T B x$ since $(A^Ty)^T = y^T A$

Let's define $y' = A^T y$:

$C = {\underbrace{(A^Ty)}\_{y'}}^T B x = y'^T B x$

From our previous result:

$\frac{\delta C}{\delta B} = y' x^T = A^T y x^T$

To summarize:

$$\boxed{C = y^T A B x \implies \frac{\delta C}{\delta A}=y x^T B^T, \frac{\delta C}{\delta B} = A^T y x^T}$$

#### Cost quadratic in weights

The other term in our neural network cost function is a quadratic cost function:

$C = \frac{1}{2} x^TA^TAx$

We have two $A$ matrices multiplying the terms hence it's quadratic in the weights/elements of $A$.

The dimensions are:

$\text{dim}(x) = (n,1) \implies dim(x^T) = (1,n)$

$\text{dim}(A) = (m,n) \implies dim(A^T) = (n,m)$

Can we still guess what $\frac{\delta C}{\delta A}$ should be from the dimensions alone?

We expect $\text{dim}(A) = \text{dim}(\frac{\delta C}{\delta A}) = (m,n)$ and also since $C$ is quadratic in $A$, we expect the derivative to be linear in $A$.

Let's take a few guesses:

$\frac{\delta C}{\delta A} = A (x^Tx)$ which works dimensionally since $x^Tx$ is just a number.

$\frac{\delta C}{\delta A} = A (xx^T)$ which works dimensionally since $xx^T$ has dimension $(n,n)$ so we still get something linear in $A$ and with dimension $(m,n)$.

Technically, $\frac{\delta C}{\delta A} = A (xx^T) (xx^T)$ also works. But if we follow our intuition from calculus, $C$ is quadratic in $x$ and the $x$ terms just come along for the ride as constants. Taking derivatives can't change its order. So the final answer also needs to be quadratic in $x$ which rules out $A (xx^T) (xx^T)$ or $A (xx^T)^n$ for $n>1$.

Let's see if we can convince ourselves by doing an explicit calculation. We'll happly use our new index notation to cut through the calculation:

$C = \frac{1}{2} x^TA^TAx = \frac{1}{2} x_i (A^T)\_{ij} (A)\_{jk} x_k = \frac{1}{2} x_i a_{ji} a_{jk} x_k$

where as before repeated indices mean an implicit sum. Now, using the chain rule:

$\frac{\partial C}{\partial a_{cd}} = \frac{1}{2} [x_i \frac{\partial a_{ji}}{\partial a_{cd}} a_{jk} x_k + x_i a_{ji} \frac{\partial a_{jk}}{\partial a_{cd}} x_k]$

$\frac{\partial C}{\partial a_{cd}} = \frac{1}{2} [x_i \delta_{j,c}\delta_{i,d} a_{jk} x_k + x_i a_{ji} \delta_{j,c}\delta_{k,d} x_k] = \frac{1}{2} [x_d a_{ck} x_k + x_i a_{ci}x_d]$

These two terms are exactly the same:

$x_d a_{ck} x_k = x_d (Ax)\_{c}$

$x_i a_{ci} x_d = x_d a_{ci} x_i = x_d (Ax)\_{c}$

and add up to kill the factor of $\frac{1}{2}$.

So,

$\frac{\partial C}{\partial a_{cd}} = (Ax)\_{c} x_d = (Axx^T)\_{cd}$

In other words, we just showed that:

$$\boxed{C = \frac{1}{2}x^TA^TAx \implies \frac{\delta C}{\delta A} = Ax x^T}$$

We still have one more calculation to do that will be crucial for doing back-propagation on our multi-node neural network.

Suppose,

$C = \frac{1}{2} x^T B^T A^T A B x$


Calculating $\frac{\delta C}{\delta A}$ is easy given what we just calculated and we just need to replace $x \rightarrow B x$. So,

$\frac{\delta C}{\delta A} = (ABx) (Bx)^T = (ABx)x^TB^T$

But what about $\frac{\delta C}{\delta B}$? $B$ is sandwiched between $A$ and $x$ and can't be factored away. If we define $D = A^T A$ then we have

$C = \frac{1}{2} x^T B^T D B x$

Again we'll guess our solution base on dimensions and then explicitly compute it.

We are given the sizes:

$\text{dim}(x) = (n,1) \implies dim(x^T) = (1,n)$

$\text{dim}(B) = (m, n) \implies dim(B^T) = (n, m)$

$\text{dim}(A) = (l, m) \implies dim(A^T) = (m,l)$

We also expect $\text{dim}(\frac{\delta C}{\delta B}) = \text{dim}(B) = (m,n)$. As before, the derivative should be linear in $B$, quadratic in $A$ and $x$ since they are for all practical purposes, constants for us.

So, let's see what modular pieces we have to work with:

Quadratic in $x$:

$\text{dim}(x^Tx) = (1,1)$

$\text{dim}(xx^T) = (n,n)$

Quadratic in $A$:

$\text{dim}(A A^T) = (l,l)$

$\text{dim}(A^T A) = (m,m)$

Linear in $B$:

$\text{dim}(B) = (m,n)$

So we can multiply $B$ on the right by something that is $(1,1)$ i.e. $x^Tx$ or $(n,n)$ i.e. $xx^T$ and on the left by something that is $(m,m)$ i.e. $A^TA$ and still maintain the dimensionality of $B$.

Our guess is:

$\frac{\delta C}{\delta B} = (A^T A) B \begin{cases} 
xx^T \\\
x^Tx \\\
\end{cases}$

We also know that if we replace $D = A^T A$ by the $(m,m)$ identity matrix, we recover our previous example $C = \frac{1}{2} x^T B^T B x$ which gave us $\frac{\delta C}{\delta B} = B (xx^T)$ so we know $xx^T$ is the wrong choice to make.

To summarize: 

$$\boxed{C = \frac{1}{2} x^T B^T A^T A B x \implies \frac{\delta C}{\delta A} = (ABx) (Bx)^T, \frac{\delta C}{\delta B} = (A^T A) B (xx^T)}$$

Of course, let's prove this by doing the explicit calculation using our powerful index notation:

We defined $D = A^TA$ which is a symmetric matrix i.e. $D^T = (A^TA)^T = A^T A = D$ or in terms of elements of $D$, $d_{ij} = d_{ji}$.

$C = \frac{1}{2} x^T B^T D B x = \frac{1}{2} x_i (B^T)\_{ij} (D)\_{jk} (B)\_{kl} x_l = \frac{1}{2} x_i b_{ji} d_{jk} b_{kl} x_l$

Then,

$[\frac{\delta C}{\delta B}]\_{cd} = \frac{1}{2} [x_i \frac{\partial b_{ji}}{\partial b_{cd}} d_{jk} b_{kl} x_l + x_i b_{ji} d_{jk} \frac{\partial b_{kl}}{\partial b_{cd}} x_l]$

The derivatives above can only be $1$ when the indices match and otherwise they are $0$:

$\frac{\partial b_{kl}}{\partial b_{cd}} = \delta_{k,c} \delta_{l,d}$

So,

$[\frac{\delta C}{\delta B}]\_{cd} = \frac{1}{2} [x_i \delta_{j,c}\delta_{i,d} d_{jk} b_{kl} x_l + x_i b_{ji} d_{jk} \delta_{k,c}\delta_{l,d} x_l]$

All repeated indices are summed over and the $\delta$s pick out the correct index. As an example:

$\delta_{a,b} x_b = \Sigma_{b=0}^{n} \delta_{a,b} x_b = \underbrace{\Sigma_{b\neq a} \underbrace{\delta_{a,b}}\_{= 0} x_b + \underbrace{\delta_{a,a}}\_{= 1} x_a}\_{\text{Separating terms where the index is a and not a}} = x_a$

In other words if you see

$\delta_{a,b} x_b$

read it as "wherever you see a $b$, replace it with an $a$ and remove the deltas"

and if you see 

$\delta(a,b)\delta(c,d) x_b y_d$

read it as "wherever you see a $b$, replace it with $a$ and wherever you see $d$, replace it with $c$ and remove the deltas".

Using this, we get

$[\frac{\delta C}{\delta B}]\_{cd} = \frac{1}{2} [x_d d_{ck} b_{kl} x_l + x_i b_{ji} d_{jc} x_d]$

These are basically the same terms:

$[\frac{\delta C}{\delta B}]\_{cd} = \frac{1}{2} x_d [d_{ck} b_{kl} x_l + d_{jc} b_{ji} x_i]$

where we have just rearranged the factors in the second term and factored out $x_d$. Recall that $D$ was symmetric, i.e. $d_{jc} = d_{cj}$. Then we get

$[\frac{\delta C}{\delta B}]\_{cd} = \frac{1}{2} x_d [d_{ck} b_{kl} x_l + d_{cj} b_{ji} x_i]$

So the two terms are exactly the same since the only non-repeated index is $c$. In other words

$[\frac{\delta C}{\delta B}]\_{cd} = x_d d_{ck} b_{kl} x_l = (DBx)\_{c}x_d = (DBxx^T)\_{cd}$

confirming our suspicion that:

$\frac{\delta C}{\delta B}] = (DBxx^T) = (A^TA)B(xx^T)$

That's it! I promise that's the end of index manipulation exercises for this section. We'll now collect all our results and use them to show that we still get backward chains as before.

$$\boxed{C = y^TAx \implies \frac{\delta C}{\delta A} = yx^T}\label{linearnoactA}$$

$$\boxed{C = y^TABx \implies \frac{\delta C}{\delta A} = yx^TB^T, \frac{\delta C}{\delta B} = A^Tyx^T}\label{linearnoactAB}$$

$$\boxed{C = \frac{1}{2} x^TA^TAx \implies \frac{\delta C}{\delta A} = A(xx^T)}\label{quadraticnoactA}$$

$$\boxed{C = \frac{1}{2} x^TB^TA^TABx \implies \frac{\delta C}{\delta A} = AB(xx^T)B^T, \frac{\delta C}{\delta B} = (A^TA) B (xx^T)}\label{quadraticnoactAB}$$

At this stage, we could declare victory because we have learned how to differentiate expression that occur in the cost for the feedforward neural network without worrying about indices. But, what we really want is rules that we can understand easily. We care mostly about equations $\ref{linearnoactAB}$ and $\ref{quadraticnoactAB}$. Why? Because generally our neural networks will have multiple layers with a matrix ($A, B$) for each layer-to-layer transition. 

Let's focus on equation $\ref{linearnoactAB}$:

$$C = y^TABx \implies \frac{\delta C}{\delta A} = yx^TB^T, \frac{\delta C}{\delta B} = A^Tyx^T$$ and on the $\frac{\delta C}{\delta A}$ term first.

$\frac{\delta C}{\delta A} = \frac{\delta}{\delta A}(y^TABx) = \frac{\delta}{\delta A} (\underbrace{y^T}\_{\text{constant with respect to A}} A \underbrace{Bx}\_{\text{constant with respect to A}}) = yx^TB^T = (y^T)^T (Bx)^T$

Now, if we were dealing with usual derivatives ($a,b$ are constants below):

$\frac{d(a x b)}{dx} = \frac{d}{dx} (\underbrace{a}\_{\text{constant with respect to x}} x \underbrace{b}\_{\text{constant with respect to x}}) = a b$

So, the derivative $\frac{d}{dx}$ simply let's $a$ and $b$ pass through.

But it seems for our matrix derivative, the constants $y^T$ and $Bx$ pass through after being transposed. In other words, the derivative $\frac{\delta}{\delta A}$ will transpose any constant vector to give:

$\frac{\delta}{\delta A}(y^TABx) = (y^T)^T \frac{\delta}{\delta A}(A Bx) = y \underbrace{\frac{\delta A}{\delta A}}\_{=1} (Bx)^T = y (Bx)^T = yx^TB^T$

Also, note that unlike regular numbers where we can re-order the terms, $ab = ba$, this is generally not true for vectors and matrices $AB \neq BA$ and is sometimes not even defined given $\text{dim}(A)$ and $\text{dim}(B)$.

We haven't proven this rules works in general so let's see if it works for $\frac{\delta C}{\delta B}$. If we transposed and passed through every constant vector and matrix, we would get:

$\frac{\delta C}{\delta B} = \frac{\delta}{\delta B}(y^TABx) = (y^TA)^T\frac{\delta B}{\delta B}x^T = A^Tyx^T$ which is exactly what $\ref{linearnoactAB}$ says!

So the central rules seems to be:

* To differentiate an expression linear in the matrix we want to differentiate with respect to, pass through all constants but transpose them.

We haven't proven this rule works in general but we'll constantly test this intuition.
### End of Aside on Matrix derivatives

It's time to get back to our neural network and put all this together. To recap, our forward propagation was defined as:

$x_1 = W_{01} x_0$

$x_2 = W_{12} x_1$

$x_3 = W_{23} x_2$

or if we combine the equations:

$x_3 = W_{23}W_{12}W_{01}x_0$

and the cost is:

$$C[W_{01}, W_{12}, W_{23}] = \frac{1}{2}[x_3^Tx_3 - 2 y^Tx_3] = \frac{x_3^Tx_3}{2} - y^Tx_3$$

Plugging in the expression for $x_3$, we get

$$C[W_{01}, W_{12}, W_{23}] = \frac{1}{2}x_0^TW_{01}^TW_{12}^TW_{23}^TW_{23}W_{12}W_{01}x_0 - y^TW_{23}W_{12}W_{01}x_0$$

We can now use our catalog of matrix derivatives to calculate the 3 derivatives needed for gradient descent: $\frac{\delta C}{\delta W_{01}}, \frac{\delta C}{\delta W_{12}}, \frac{\delta C}{\delta W_{23}}$

$\frac{\delta C}{\delta W_{01}}$:

Let's define $D \equiv W_{23}W_{12}$ to get:

$C = \frac{1}{2}x_0^TW_{01}^T\underbrace{W_{12}^TW_{23}^T}\_{D^T}\underbrace{W_{23}W_{12}}\_{D}W_{01}x_0 - y^T\underbrace{W_{23}W_{12}}\_{D}W_{01}x_0$

$C = \frac{1}{2} x_0^T W_{01}^T D^T D W_{01} x_0 - y^T D W_{01} x_0$

Then, using identities $\ref{quadraticnoactAB}$ AND $\ref{linearnoactAB}$:

$\frac{\delta C}{\delta W_{01}} = \frac{\delta}{\delta W_{01}} \frac{1}{2} x_0^T W_{01}^T D^T D W_{01} x_0 - \frac{\delta}{\delta W_{01}} y^T D W_{01} x_0 = (D^TD)W_{01}(x_0x_0^T) - D^T y x_0^T$

So,

$\frac{\delta C}{\delta W_{01}} = W_{12}^TW_{23}^T(W_{23}W_{12}W_{01}x_0)x_0^T - W_{12}^TW_{23}^Tyx_0^T$

But, $W_{23}W_{12}W_{01}x_0$ is precisely $x_3$, the result of forward propagation. So, we get a very nice result:

$$\boxed{\frac{\delta C}{\delta W_{01}} = W_{12}^TW_{23}^T(x_3-y)x_0^T}$$

Note that the combination $x_3-y$ shows up again and if the prediction is exactly correct i.e. if $x_3 = y$, then the derivative is 0 and there's no correction via gradient descent, as one would expect.

$\frac{\delta C}{\delta W_{12}}$:

Define $u \equiv W_{01}x_0$ to get:

$C = \frac{1}{2} \underbrace{x_0^TW_{01}^T}\_{u^T} W_{12}^TW_{23}^TW_{23}W_{12} \underbrace{W_{01}x_0}\_{u} - y^TW_{23}W_{12}\underbrace{W_{01}x_0}\_{u}$

$C = \frac{1}{2}u^TW_{12}^TW_{23}^TW_{23}W_{12}u - y^TW_{23}W_{12}u$

Using identities $\ref{quadraticnoactAB}$ AND $\ref{linearnoactAB}$, we get:

$\frac{\delta C}{\delta W_{01}} = W_{23}^TW_{23}W_{12}uu^T - W_{23}^Tyu^T$

Replacing $u = W_{01}x_0$,

$$\frac{\delta C}{\delta W_{12}} = W_{23}^TW_{23}W_{12}W_{01}x_0x_0^TW_{01}^T - W_{23}^Tyx_0^TW_{01}^T = W_{23}^Tx_3x_1^T - W_{23}^Tyx_1^T = W_{23}^T(x_3-y)x_1^T$$

So, 

$$\boxed{\frac{\delta C}{\delta W_{12}} = W_{23}^T(x_3-y)x_1^T}$$

$\frac{\delta C}{\delta W_{23}}$:

Define $D \equiv W_{12}W_{01}$ to get:

$C = \frac{1}{2}x_0^T \underbrace{W_{01}^TW_{12}^T}\_{D^T} W_{23}^TW_{23}\underbrace{W_{12}W_{01}}\_{D}x_0 - y^TW_{23} \underbrace{W_{12}W_{01}}\_{D} x_0$

$C = \frac{1}{2}x_0^TD^TW_{23}^TW_{23}Dx_0 - y^TW_{23}Dx_0$

Using identities $\ref{quadraticnoactAB}$ AND $\ref{linearnoactAB}$, we get:

$\frac{\delta C}{\delta W_{23}} = W_{23}D(x_0x_0^T)D^T - y^Tx_0^TD^T$

Replacing $D = W_{12}W_{01}$:

$\frac{\delta C}{\delta W_{23}} = W_{23}W_{12}W_{01}x_0x_0^TW_{01}^TW_{12}^T - y^Tx_0^TW_{01}^TW_{12}^T = (x_3-y)x_2^T$

In summary:

$$\boxed{\frac{\delta C}{\delta W_{01}} = W_{12}^TW_{23}^T(x_3-y)x_0^T}$$

$$\boxed{\frac{\delta C}{\delta W_{12}} = W_{23}^T(x_3-y)x_1^T}$$

$$\boxed{\frac{\delta C}{\delta W_{23}} = (x_3-y)x_2^T}$$


Presto!!! We again see forward and backward chains.

Forward chains:
* ${\color{blue} x_0}$
* $W_{01} x_0 = {\color{blue} x_1}$
* $W_{12} W_{01} x_0 = {\color{blue} x_2}$
* $W_{23} W_{12} W_{01} x_0 = {\color{blue} x_3}$

Backward chains:
* $x_3-y \equiv {\color{red} {\Delta_0}}$
* $W_{23}^T(x_3-y) \equiv {\color{red} {\Delta_1}}$
* $W_{12}^TW_{23}^T(x_3-y) \equiv {\color{red} {\Delta_2}}$
* $W_{01}^TW_{12}^TW_{23}^T(x_3-y) \equiv {\color{red} {\Delta_3}}$

where we now use capital deltas $\Delta$ instead of small deltas $\delta$, to signify that the backward chains are matrices.

As before, we can succinctly write the derivatives as:

$\frac{\delta C}{\delta W_{i,i+1}} = \Delta_{2-i} x_i^T$

In this notation, this is essentially the same as the results from sections I and II except for the fact that $x_i$ is now a vector and $\Delta_i$ is a matrix.

## Backpropagation IV (In Progress)- non-linear activations + multi-node layers

Finally, the action begins! We can now start building up the full backpropagation for a realistic feedforward neural network with multiple layers, each with multiple nodes and non-linear activations.

We'll use notation similar to section II.

Forward propagation is:

$\begin{array}{ccc}
p_0 = x_0 & q_0 = \sigma_0(x_0) \\\
p_1 = W_{01} q_0 & q_1 = \sigma_1(p_1) \\\
p_2 = W_{12} q_1 & q_2 = \sigma_2(p_2) \\\
p_3 = W_{23} q_2 & q_3 = \sigma_3(p_3) \\\
\end{array}$

where $W_{ij}$ are matrices and $p_i, q_i$ are vectors and all the dimensions are such that matrix multiplications are well-defined. $\sigma_i$ are activation functions and they act on vectors element-wise:

$\sigma \begin{bmatrix}
x_1 \\\
x_2 \\\
\vdots \\\
x_n
\end{bmatrix}
= \begin{bmatrix}
\sigma(x_1) \\\
\sigma(x_2) \\\
\vdots \\\
\sigma(x_n) \\\
\end{bmatrix}$

To summarize:
* $p_i$ and $q_i$ are always vectors.
* $p_i$ is the "pre-activation" ("p" for "pre") input to a layer. 
* $q_i$ is the "post-activation" ("q" since it comes after "p") output of a layer.
* $W_{ij}$ is a matrix that always takes $q_i \rightarrow p_j$.
* $\sigma_i$ always takes $p_i \rightarrow q_i$.

The last two rules pop out of our equations and while it's just notation, it serves as a powerful guide to ensure that we are not making mistakes. At any point in the calculation if you see the combination $W_{ij} p_i$, something is probably wrong. If we see $W_{ij}p_k$ where $k\neq i,j$, something is probably wrong.

As before, we'll use the mean-squared cost which is:

$C[W_{01}, W_{12}, W_{23}] = \frac{1}{2} (q_3-y)^2$

For the purposes of optimization, we can expand:

$C = \frac{1}{2} (q_3^Tq_3 - 2y^Tq_3)$

so we get a term quadratic in $q_3$ ($\frac{1}{2} q_3^Tq_3$) and a term linear in $q_3$ ($y^Tq_3$). As in section III, we drop the constant (independent of $q_3$ and thus the weights) term $\frac{1}{2} y^Ty$.

If we combined all the forward propagation equations, we get:

$q_3 = \sigma_3(W_{23}\sigma_2(W_{12}\sigma_1(W_{01}\sigma_0(p_0))))$

We need matrix derivatives but with a twist introduced by the activation functions. Since we already had some practice in Section III, let's dive straight in:

### Aside: Matrix derivatives

#### Cost linear in weights

Let's start with a simpler cost that mimics the term linear in $q_3$:

$C[A] = y^T \sigma(A x)$

where the dimensions are:

$\text{dim}(x) = (n,1)$

$\text{dim}(y) = (m,1)$

$\text{dim}(A) = (m,n)$

In other words:

$C[A] = \underbrace{y^T}\_{(1,m)} \sigma(\underbrace{A}\_{(m,n)} \underbrace{x}\_{(n,1)})$

We want to compute $\frac{\delta C}{\delta A}$.

We can try guessing what this should be based on the dimensions of the matrix. 

* $\frac{\delta C}{\delta A}$ should be linear in $x$ and $y$ 
* It should also depend on $\sigma'(Ax)$ where 

$\sigma'(Ax) = \sigma'\begin{bmatrix}
a_{11}x_1 + a_{12}x_2 + \ldots + a_{1n}x_n \\\
a_{21}x_1 + a_{22}x_2 + \ldots + a_{2n}x_n \\\
\vdots \\\
a_{m1}x_1 + a_{m2}x_2 + \ldots + a_{mn}x_n \\\
\end{bmatrix}
=\begin{bmatrix}
\sigma'(a_{11}x_1 + a_{12}x_2 + \ldots + a_{1n}x_n) \\\
\sigma'(a_{21}x_1 + a_{22}x_2 + \ldots + a_{2n}x_n) \\\
\vdots \\\
\sigma'(a_{m1}x_1 + a_{m2}x_2 + \ldots + a_{mn}x_n) \\\
\end{bmatrix}$
with $\sigma'(x)$ denoting $\frac{d\sigma}{dx}$ ns a more compact notation.
* $\text{dim}(\frac{\delta C}{\delta A}) = \text{dim}(A) = (m,n)$

* Any guess we come up with should reduce to the special case $\frac{\delta C}{\delta A} = yx^T$ when $\sigma = id$, the identity function, $id(x) = x$, which is the result we derived in Section III.

Let's look at how we can combine these terms to get something with dimensions = $(m,n)$:

* $y x^T$
* $A$ but this shouldn't show up directly if we follow the intuition that the derivative of a linear function $f(x) = ax$ with respect to $a$ should be independent of $a$.
* We can replace $y$ with $\sigma'(Ax)$ since $\text{dim}(Ax) = \text{dim}(\sigma(Ax)) = \text{dim}(\sigma'(Ax)) = \text{dim}(y) = (m,1)$ i.e. $Ax$ has the same dimensions as $y$ and acting with element-wise functions, whether $\sigma$ or $\sigma'$ doesn't change dimensions. So another option is $\sigma'(Ax) x^T$.

How do we combine both $yx^T$ and $\sigma'(Ax) x^T$ since both terms have the correct dimensions $(m,n)$ required for $\frac{\delta C}{\delta A}$ and we need the derivative to be linear in $y$ (requiring the first term) and it needs to contain $\sigma'(Ax)$ (requiring the second term).

Since we are unsure, let's explicitly calculate the derivative:

$C = y^T \sigma(Ax) = (y_1 y_2 \ldots y_m) \begin{bmatrix}
\sigma(a_{11}x_1 + a_{12}x_2 + \ldots + a_{1n}x_n) \\\
\sigma(a_{21}x_1 + a_{22}x_2 + \ldots + a_{2n}x_n) \\\
\vdots \\\
\sigma(a_{m1}x_1 + a_{m2}x_2 + \ldots + a_{mn}x_n) \\\
\end{bmatrix}$

$C = y_1 \sigma(a_{11}x_1 + a_{12}x_2 + \ldots + a_{1n}x_n) + \\\ y_2 \sigma(a_{21}x_1 + a_{22}x_2 + \ldots + a_{2n}x_n) + \\\ \ldots + y_m \sigma(a_{m1}x_1 + a_{m2}x_2 + \ldots + a_{mn}x_n)$.

Differentiating with respect to $a_{kl}$, we get

$\frac{\partial C}{\partial a_{kl}} = y_k \sigma'(a_{k1}x_1 + a_{k2} x_2 + \ldots a_{kn} x_n) x_l = y_k [\sigma'(Ax)]\_{k} x_l$

where $[\sigma'(Ax)]\_{k}$ denotes the $k$th element of $\sigma'(Ax)$

Contrast this to what we got in Section III without activation functions. There the same exact derivative gave us $y_k x_l$. The fact that we see a term $y_k [\sigma'(Ax)]\_{k}$ tells us that we need to define an element-wise multiplication operation:

$$x \odot y$$

that takes two vectors of the same dimensions and just multiplies the corresponding elements together:

$$x \odot y = \begin{bmatrix}
x_1 \\\
x_2 \\\
\ldots \\\
x_n
\end{bmatrix} \odot
\begin{bmatrix}
y_1 \\\
y_2 \\\
\ldots \\\
y_n
\end{bmatrix} = \begin{bmatrix}
x_1 y_1 \\\
x_2 y_2 \\\
\ldots \\\
x_n y_n
\end{bmatrix}$$

It looks like a weird operation but just pops out of our calculation. Using this notation, we just showed

$$\frac{\partial C}{\partial a_{kl}} = [y \odot\sigma'(Ax)]_k x_l$$

and 

$$\boxed{\frac{\delta C}{\delta A} = [y \odot \sigma'(Ax)] x^T}$$

which combines both the terms we guessed we needed, $yx^T$ and $\sigma'(Ax)x^T$ using a new operation.

Time for a sanity check. If there was no activation function or more formally, if the activation function was the identity function $\sigma(x) = id(x) = x$, then we expect to recover $yx^T$. Let's see if this is the case.

If $\sigma(x) = id(x) = x$, then $\sigma'(x) = id'(x) = 1$, no matter what $x$ is. This would give:

$\sigma'(Ax) = \begin{bmatrix}
1 \\\
1 \\\
\vdots \\\
1 \\\
\end{bmatrix}$

and $y \odot \sigma'(Ax) = y \odot \begin{bmatrix}
1 \\\
1 \\\
\vdots \\\
1 \\\
\end{bmatrix} = \begin{bmatrix}
y_1\*1 \\\
y_2\*1 \\\
\vdots \\\
y_m \*1 \\\
\end{bmatrix} = y$

So, we would get:

$$\frac{\delta C}{\delta A} = [y \odot \sigma'(Ax)] x^T = y x^T$$ 

which is what we expected.

We need to now develop some intuition to see what these derivatives would look like if the matrix $A$ was nested the way forward propagation nests weight matrices. In other words, what if 

$$C = y^T \sigma_2(A\sigma_1(B x))$$

We now have two derivatives: $\frac{\delta C}{\delta A}, \frac{\delta C}{\delta B}$.

$\frac{\delta C}{\delta A}$:

This is the easier one. Define $q = \sigma_1(Bx)$ to give:

$C = y^T \sigma_2(A\underbrace{\sigma_1(B x)}\_{q}) = y^T \sigma_2(Aq)$

From our previous calculation, we know

$\frac{\delta C}{\delta A} = [y \odot \sigma_2'(Aq)]q^T = \boxed{[y \odot \sigma_2'(A\sigma_1(Bx))] \sigma_1(Bx)^T}$

Recall from section III that we guessed a rule that a matrix derivative transposes every constant vector it sees. The same phenomenon occurs here too. Let's see it step by step:

$\frac{\delta C}{\delta A} = \frac{\delta}{\delta A} (y^T \sigma_2(A\sigma_1(B x)))$

$\frac{\delta C}{\delta A} = y \frac{\delta}{\delta A}(\sigma_2(A\sigma_1(B x)))$

Now, using the chain rule, we get $\sigma_2'$ and a preceding $\odot$:

$\frac{\delta C}{\delta A} = y \odot \sigma_2'(A\sigma_1(B x)) \frac{\delta}{\delta A}(A\sigma_1(B x))$

Since, $\sigma_1(Bx)$ is a constant with respect to $A$, it has to get transposed.

$\frac{\delta C}{\delta A} = y \odot \sigma_2'(A\sigma_1(B x)) \frac{\delta}{\delta A}(A) (\sigma_1(B x))^T$

Using $\frac{\delta A}{\delta A} = 1$, we get our final answer:

$\frac{\delta C}{\delta A} = [y \odot \sigma_2'(A\sigma_1(B x)] \sigma_1(B x)^T$

That's great! Now we only need to follow these rules and not worry about the details of what goes on underneath.

Let's see if we can apply these rules to guess what $\frac{\delta C}{\delta B}$ should look like and then we'll explicitly compute it:

**Using imputed rules**:

$\frac{\delta C}{\delta B} = \frac{\delta}{\delta B} (y^T \sigma_2(A\sigma_1(B x)))$

Tranpose "constant" vector $y$:

$\frac{\delta C}{\delta B} = y \frac{\delta}{\delta B} (\sigma_2(A\sigma_1(B x)))$

Apply chain rule i.e. differentiate the function and put $\odot$ before it:

$\frac{\delta C}{\delta B} = [y \odot \sigma_2'(A\sigma_1(Bx))] \frac{\delta}{\delta B} (A\sigma_1(Bx))$

Tranpose constant matrix $A$:

$\frac{\delta C}{\delta B} = [y \odot \sigma_2'(A\sigma_1(Bx))] A^T \frac{\delta}{\delta B} (\sigma_1(Bx))$

Use chain rule again:

$\frac{\delta C}{\delta B} = [y \odot \sigma_2'(A\sigma_1(Bx))] A^T \odot \sigma_1'(Bx) \frac{\delta}{\delta B} (B x)$

Transpose "constant" vector $x$:

$\frac{\delta C}{\delta B} = [y \odot \sigma_2'(A\sigma_1(Bx))] A^T \odot \sigma_1'(Bx) \frac{\delta}{\delta B} (B) x^T$

Use $\frac{\delta B}{\delta B} = 1$,

$\frac{\delta C}{\delta B} = [y \odot \sigma_2'(A\sigma_1(Bx))] A^T \odot \sigma_1'(Bx) x^T$

The big question now is if this is correct?

Let's check if the dimensions work out.

$$C = \underbrace{y^T}_{(1,k)} \sigma_2(\underbrace{A}_{(k,m)}\sigma_1(\underbrace{B}_{(m,n)} \underbrace{x}_{(n,1)}))$$

So $\text{dim}(y) = \text{dim}(\sigma_2'(A\sigma_1(Bx))) = (k,1)$ and $y \odot \sigma_2'(A\sigma_1(Bx))$ is well-defined.

This term is then multiplied by $A^T$ on the right i.e.:

$[y \odot \sigma_2'(A\sigma_1(Bx))] A^T$

Oops! this is not well-defined! $\text{dim}(A^T) = (m,k)$ so we have a multiplication of the form $(k,1)(m,k)$. 

Let's see if we can salvage this (don't worry, we'll confirm all our guesses with explicit calculations below). While $(k,1)(m,k)$ is not well-defined, $(m,k)(k,1)$ is. So, maybe the rule is that the $A^T$ should be tacked on in front of everything to its left i.e.

$\frac{\delta C}{\delta B} = A^T [y \odot \sigma_2'(A\sigma_1(Bx))] \frac{\delta}{\delta B} (\sigma_1(Bx))$

Does this weird rule make sense? Of course! Imagine we had

$\frac{\delta }{\delta A} (y^T B^T A x)$

We can view this in two ways. The first one is to treat one term at a time:

$\frac{\delta }{\delta A} (y^T B^T A x) = y \frac{\delta }{\delta A} (B^T A x) = y B \frac{\delta }{\delta A} (Ax) = y B x^T$

The second one is to realize that $y^T B^T$ is a constant term and can be grouped together:

$\frac{\delta }{\delta A} (y^T B^T A x) = (y^T B^T)^T \frac{\delta }{\delta A} (A x) = Byx^T$

Now we see the problem. When we treat one term at a time, we are transposing constant terms but $(AB)^T \neq A^T B^T$. Instead it is $B^T A^T$. So, every time we see a constant term, we need to transpose it AND tack it on the left of everything.

$\frac{\delta }{\delta A} (y^T B^T A x) = y \frac{\delta }{\delta A} (B^T A x) = {\color{red} {B y}} \frac{\delta }{\delta A} (Ax) = y B x^T$

Going back to our original calculation:

$\frac{\delta C}{\delta B} = \frac{\delta}{\delta B} (y^T \sigma_2(A\sigma_1(B x)))$

Tranpose "constant" vector $y$:

$\frac{\delta C}{\delta B} = y \frac{\delta}{\delta B} (\sigma_2(A\sigma_1(B x)))$

Apply chain rule i.e. differentiate the function and put $\odot$ before it:

$\frac{\delta C}{\delta B} = [y \odot \sigma_2'(A\sigma_1(Bx))] \frac{\delta}{\delta B} (A\sigma_1(Bx))$

Tranpose constant matrix $A$ AND put it to the left of the previous term:

$\frac{\delta C}{\delta B} = A^T [y \odot \sigma_2'(A\sigma_1(Bx))] \frac{\delta}{\delta B} (\sigma_1(Bx))$

Use chain rule again:

$\frac{\delta C}{\delta B} = A^T [y \odot \sigma_2'(A\sigma_1(Bx))] \odot \sigma_1'(Bx) \frac{\delta}{\delta B} (B x)$

Transpose "constant" vector $x$:

$\frac{\delta C}{\delta B} = A^T [y \odot \sigma_2'(A\sigma_1(Bx))] \odot \sigma_1'(Bx) \frac{\delta}{\delta B} (B) x^T$

Use $\frac{\delta B}{\delta B} = 1$,

$$\boxed{\frac{\delta C}{\delta B} = \{[A^T [y \odot \sigma_2'(A\sigma_1(Bx))] \odot \sigma_1'(Bx)\} x^T}$$

Now all the dimensions are well-defined:

$\text{dim}[A^T [y \odot \sigma_2'(A\sigma_1(Bx))]] = (m,1)$

$\text{dim} [\sigma_1'(Bx)] = (m,1)$

So, $\text{dim} [A^T [y \odot \sigma_2'(A\sigma_1(Bx))] \odot \sigma_1'(Bx)] = (m,1)$

and $\text{dim}(x^T) = (1,n)$ 

to give:

$\text{dim}(\frac{\delta C}{\delta B}) = (m,n) = \text{dim}(B)$!

Let's also check that if $\sigma_1 = id$, we basically recover the previous case using $\sigma'(x) = 1$ and $\odot$ing with a vector of $1$s gives the original vector back:

$\frac{\delta C}{\delta B} = A^T [y \odot \sigma_2'(A\sigma_1(Bx))] x^T = A^T [y \odot \sigma_2'(ABx)] x^T$

If $A = I$, the identity matrix (diagonals are $1$ and everything else is $0$), we get:

$\frac{\delta C}{\delta B} = y \odot \sigma_2'(Bx)] x^T$

as expected.

We have another (ugly) way of confirming this calculation and while it's not pretty, we have to confirm as many ways as we can:

**Explicit calculation**

$C = y^T \sigma_2(A\sigma_1(B x))$

#### Cost quadratic in weights

Let's start with a simple quadratic cost:

$C = \frac{1}{2} \sigma(x^TA^T)\sigma(Ax)$

We have done most of the hard work above. Let's now compute the derivative using both our imputed rules and explicitly to ensure they match.

**Imputed Rules**

$\frac{\delta C}{\delta A} = \frac{\delta}{\delta A} [\frac{1}{2} \sigma(x^TA^T)\sigma(Ax)]$



$\frac{\delta C}{\delta A} = \frac{1}{2} [\frac{\delta \sigma(x^TA^T)}{\delta A}\sigma(Ax) + \sigma(x^TA^T)\frac{\delta \sigma(xA)}{\delta A}]$

The two derivatives are equal since the terms are just transposed. We can combine them and get rid of the factor of $\frac{1}{2}$:

$\frac{\delta C}{\delta A} = \sigma(x^TA^T)\frac{\delta \sigma(xA)}{\delta A}$


### End of Aside on Matrix derivatives

