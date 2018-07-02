---
layout: post
title: "Beating the baseline model"
modified: 2018-05-29 14:08:46 -0700
tags: [ml]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
published: false
---

Now that you have your baseline ML model, you want to make it better?
Well, there are some tricks that you can employ.
Let's look through them :)

<!-- more -->

At this stage your goal is to achieve _statistical power_: develop a small model that is capable of beating a dumb baseline.
By a "dumb" model I mean the model that you cooked up really quickly just to see if anything works: you confirmed that (1) the outputs can be predicted given the inputs, and (2) you have enough _informative_ data to learn the input/output relationship.

**Most of the material here is from Fran&ccedil;ois Chollet's "Deep Learning with Python."**

## Baseline model

Before discussing some modifications, let us get some  baseline model, and see if we can beat random guessing :).
We will be using the MNIST dataset for our experiments.

{% digraph Baseline model %}
rankdir=LR
splines=line
nodesep=.05;
node[label="", style=solid, shape=circle, fixedsize=true];
edge[minlen=1]

subgraph cluster_input {
    color = white;
    node [color=blue4];
    label = "Input";
    in1[label="pix 1"]
    in2[label="pix 2"]
    indots[label="..." pin=true]
    in3[label="pix\n784"]
    subgraph inputs {
        node [shape=point, label=""]
        i1 i2 idots i3
    }
    i1 -> in1
    i2 -> in2
    idots -> indots
    i3 -> in3
}

subgraph cluster_hidden {
    color=white;
    node [color=red2 label="w1&Sigma;"];
    label = "Hidden (N nodes)";
    x11 x12 xdots[label="..." pin=true] x13
    subgraph activation {
        node [label="&sigma;1"]
        a11 a12 adots[label="..." pin=true] a13
    }
}

subgraph cluster_output {
    color=white;
    node [color=seagreen2, label="w2&Sigma;"];
    label="Output";
    O11 O12 O1dots[label="..." pin=true] O13
    subgraph activation {
        node [label="&sigma;2"]
        O21 O22 O2dots[label="..." pin=true] O23
    }
    subgraph outputs {
        node [shape=point, label=""]
        o1 o2 odots o3
    }
    O21 -> o1 [label="1"]
    O22 -> o2 [label="2"]
    O2dots -> odots [label="..."]
    O23 -> o3 [label="10"]
}

in1 -> {x11, x12, x13}
in2 -> {x11, x12, x13}
in3 -> {x11, x12, x13}

x11 -> a11 -> {O11, O12, O13}
x12 -> a12 -> {O11, O12, O13}
x13 -> a13 -> {O11, O12, O13}
xdots -> adots

O11 -> O21
O12 -> O22
O13 -> O23
O1dots -> O2dots
{% enddigraph %}

No need to explain the details of the _fully connected_: whole bunch of pixels come in, sum up and activate twice, and return as labels... Easy! :)

$$w?\Sigma$$ represents the weighted sum, and $$\sigma1$$ and $$\sigma2$$ represent the activation functions. If you like math more than diagrams (you poor soul), the above diagram can be written as

$$
\begin{align}
\text{Output}_{[1\times10]} =
\sigma_2\left(
    \sigma_1\left(
        \text{Pixels}_{[1\times784]}\cdot W_{1[784\times N]}
    \right) \cdot W_{2[N\times 10]}
\right)
\end{align}
$$

Here is the model for the above graph (+ some housekeeping).
Don't ask why I chose those parameter -- it was completely random:
{% gist 1a91fee57380940c73f57452214cbbbe %}

And, while we are at it, here is how I run it:
{% gist 6555f515d472a1af93960599ff5078cf%}

The results are pretty good, as shown in the report below

Training...
{% highlight nolang %}
Train on 48000 samples, validate on 12000 samples
...
Epoch 15/20
48000/48000 [==============================] - 1s 18us/step - loss: 0.0025 - acc: 0.9943 - val_loss: 0.0060 - val_acc: 0.9766
Epoch 16/20
48000/48000 [==============================] - 1s 17us/step - loss: 0.0024 - acc: 0.9944 - val_loss: 0.0059 - val_acc: 0.9770
Epoch 17/20
48000/48000 [==============================] - 1s 17us/step - loss: 0.0022 - acc: 0.9948 - val_loss: 0.0058 - val_acc: 0.9783
Epoch 18/20
48000/48000 [==============================] - 1s 19us/step - loss: 0.0021 - acc: 0.9949 - val_loss: 0.0060 - val_acc: 0.9769
Epoch 19/20
48000/48000 [==============================] - 1s 16us/step - loss: 0.0020 - acc: 0.9953 - val_loss: 0.0059 - val_acc: 0.9777
Epoch 20/20
48000/48000 [==============================] - 1s 16us/step - loss: 0.0020 - acc: 0.9958 - val_loss: 0.0059 - val_acc: 0.9779
{% endhighlight %}

Testing...
{% highlight nolang %}
10000/10000 [==============================] - 0s 20us/step
test_acc: 0.9768
test_loss: 0.005966137247346342
{% endhighlight %}

![Simple Model for MNIST]({{ "/images/2018-05-29-beating-the-baseline-model/simple_mnist.png" }})

## Step 1. Change your model.
The first step is to change the model to something that has more statistical power.
The previous model doesn't really improve on our accuracy at all :'(.
Given that the input is an image -- why not putting a couple of convolutional layers in there? Checkout the new model below