---
layout: post
title: "not_notMNIST Dataset Generation"
modified: 2017-01-19 22:44:15 -0800
tags: [machine learning, notMNIST, classification]
image:
  feature: letters.png
  credit: not_notMNIST GitHub
  creditlink: https://github.com/zafartahirov/not_notMNIST
comments: 
share: 
---

If you have ever worked with classification algorithms, you are **definitely** familiar with the [MNIST](http://yann.lecun.com/exdb/mnist/) dataset. 
If you are a little more involved with machine learning and especially classification, you have heard of the [notMNIST](http://yaroslavvb.blogspot.com/2011/09/notmnist-dataset.html) as well.

However, if you always wanted to have your own dataset, but didn't know how to use it -- this post is for you!

<!-- more -->
**not_notMNIST** is a dataset generator! Everything it needs is an alphabet and whole bunch of fonts. The main advantage is that you can use it to generate a really big dataset with some unicode characters, and train your classifier on that. *If you believe that the dataset you generate is worth spreading, share it* :)

## How to use the generated data?

[Once you generate the data](#how-to-generate-the-data), you can just load the `pickle` file either per character, or for all of the generated data.

The `pickle` file is stored under `$DIR/$WIDTHx$WIDTH.pickle`, where `$DIR` is the output directory (by default `./28x28/`), and `$WIDTH` is the width of image (defaults to `28`). Data has the following format:

- `data['labels']` - Targets
- `data['images']` - Predictors (square images)

```python
# -*- coding: utf-8 -*-

import pickle
import numpy as np
import matplotlib.pyplot as plt

with open('Demo/Japanese/100x100/100x100.pickle', 'rb') as f:
  data = pickle.load(f)

labels = data['labels']
images = data['images']

num_points = len(labels)

f, ax = plt.subplots(2,2)
for i in range(2):
  for j in range(2):
    idx = np.random.randint(num_points)
    ax[i,j].imshow(images[idx], cmap='Greys_r')
plt.show()
```

## How to generate the data

You need several things installed first:

- ImageMagick
- Python 2.7+
  - numpy
  - scipy
  - pickle

The installation for the prerequisites would depend on your OS, and is outside of the scope of the current post :)

### List of arguments

{% highlight raw %}
-a <string>, --alphabet <string>
  What alphabet to generate. Every character needs to be unique
  Defaults to [a-zA-Z0-9] characters
  Is overridden by --af or --alphabetfile
-af <file name>, --alphabetfile <file name>
  Open the alphabet from <file name>
  Is overridden by -a or --alphabet

-d <dir name>, --directory <dir name>
  Where to save the generated images
  Defaults to a new directory with the current dimensions as a name

-e <font name>, --exclude <font name>
  Exclude a font. Can be stacked
-ef <file name>, --excludefile <file name>
  Exclude all fonts from the file

-f <font name>, --font <font name>
  Font names to generate images for (could be location of a font)
-ff <file name>, --fontfile <file name>
  File with font names to load in a list
-fd <font dir>, --fontdir <font dir>
  Directory with the fonts you want to use. The supported extensions
  are 'ttf,ttc,otf'. You can modify it below in the code

-h, --help
  Print this help and exit

-w <number>, --width <number>
  Image width (and height). A square image is generated.
{% endhighlight %}

To learn how to use it, let's go through some examples. You can see the Demo files [on GitHub](https://github.com/zafartahirov/not_notMNIST)

### Example 1

{% highlight bash %}
./not_notMNIST
{% endhighlight %}

This is a default run. It will take all possible fonts that it can find, and it will try to generate 28x28 images for every alpha-numeric character. The results will be stored in the `28x28` directory.

### Example 2

{% highlight bash %}
./not_notMNIST -a AbzZ -f Arial -w 100
{% endhighlight %}

This one uses 3 new arguments:

- `-a AbzZ` tells the scripts to generate data for the letters `A`, `b`, `z`, and `Z`
- `-f Arial` We want only a single font, and it is `Arial`
- `-w 100` Images should be of size `100x100`

### Example 3
{% highlight bash %}
./not_notMNIST \
  -w 28 \
  -d Demo/Japanese/28x28 \
  -af Demo/Japanese/japanese.alphabet \
  -ff Demo/Japanese/japanese.fonts
{% endhighlight %}

This one is more complex

- `-d` specifies the output directory to be `Demo/Japanese/28x28`
- `-af` tells the script that there is a file with an alphabet `Demo/Japanese/japanese.alphabet`
- `-ff` specifies a file with a list of fonts `Demo/Japanese/japanese.fonts`.

If we new where the fonts are located, we could have used `-fd` parameter to use all the fonts in there.

### Example 4

Suppose that you want to use all the fonts in the list except 1 (or whatever number). Use `-e` argument (or `-ef`):

{% highlight bash %}
./not_notMNIST -a AbzZ -e Arial -w 100
{% endhighlight %}

would use all installed fonts EXCEPT `Arial`

## Call for Contributions

I have written this tool for my own work in ~a night. I would appreciate if you could submit any [Issues](https://github.com/zafartahirov/not_notMNIST/issues) or [PRs](https://github.com/zafartahirov/not_notMNIST/issues).

This work is still in progress...
