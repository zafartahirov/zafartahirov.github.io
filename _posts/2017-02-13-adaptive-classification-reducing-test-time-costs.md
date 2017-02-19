---
layout: post
title: "Adaptive Classification and More"
modified: 2017-02-13 17:16:02 -0500
tags: [machine learning, classifier, python, svm]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
hidden: false
---

We use machine learning almost everywhere now, and a lot of research is focused on improving the accuracy, improving the time it takes us to train the networks, improving the complexity, etc... I would like to contribute a little to a slightly different problem: how do make the test time better?

Suppose you are controlling a robot using a machine learning algorithm, and would like to increase the battery life. If you don't care about the training time, and accuracy is important, but not when your battery is dying. So how do we go about it?

We have recently released a pwork on hardware of the [adaptive classifiers](http://people.bu.edu/joshi/files/Takhirov-ISLPED-2016.pdf), but let us go step-by-step.

You can download the iPython Notebook that I used to generate the images [here](/downloads/adaptive/adaptive.ipynb).

<!-- more -->

## "Hardness" of a problem

Let us define "hardness" of a problem as "how easy is given problem" meaning if I have several classifiers, how complex of a classifier do I need to identify the label correctly. 

### Quick example

As an example, let us look at the [UCI Penbase](http://archive.ics.uci.edu/ml/datasets/Pen-Based+Recognition+of+Handwritten+Digits) dataset.

{% comment %}
{% highlight python linenos %}
import scipy.io as sio

data_raw = sio.loadmat('pendigits.mat')
xtrain, ytrain = data_raw['xtrain'].T, data_raw['ytrain'][0]
xtest, ytest = data_raw['xtest'].T, data_raw['ytest'][0]

###

from sklearn.linear_model import LogisticRegression
from sklearn.linear_model import Ridge
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import PolynomialFeatures

linear = LogisticRegression()
linear.fit(xtrain, ytrain)

# polynomial = make_pipeline(PolynomialFeatures(degree=2), LogisticRegression())
xtrain_poly = PolynomialFeatures(degree=2).fit_transform(xtrain)
polynomial = LogisticRegression()

polynomial = LogisticRegression()
polynomial.fit(xtrain_poly, ytrain)

y_linear = linear.predict(xtrain)
y_poly = polynomial.predict(xtrain_poly)

y_classifier = -np.ones(ytrain.shape)
y_classifier[y_linear != ytrain] = 1

x_lin_wrong = xtrain[y_linear != ytrain]
y_lin_wrong = y_linear[y_linear != ytrain]
fig, ax = plt.subplots(5,5)
for idx in xrange(5):
  for jdx in xrange(5):
    num = np.random.randint(0, len(x_lin_wrong))
    x = x_lin_wrong[num][::2]
    y = x_lin_wrong[num][1::2]
    ax[idx][jdx].plot(x, y)
    ax[idx][jdx].grid('off')
    ax[idx][jdx].set_axis_off()
    ax[idx][jdx].set_title(
      'T:' + str(ytrain[y_linear != ytrain][num]) + 
      '/P:' + str(y_poly[y_linear != ytrain][num]) + 
      '/L:' + str(y_linear[y_linear != ytrain][num]),
      color='b', y=0.9
    )
plt.show()
{% endhighlight %}
{% endcomment %}

In the images below there are two numbers `T:a/P:b/L:c`. `T` is the true label for the image, `P` is the prediction of the second degree polynomial, and `L` is the result of a linear classifier. That means every time we see problems like these, the linear classifier will fail.

![Examples hard for Linear](/images/adaptive/hard_linear.png)

As a first order, let's say that the "hardness" of the problem is defined as a distance from the decision boundary[^1]. So all of the above examples would be considered "hard".

### Simple example

To visualize how the idea of "hardness" would create decision boundaries, let us consider a simpler (2D) example:

![True Dataset](/images/adaptive/synthetic_true.png)

The above synthetic dataset was generated using the following `True Label` generator

$$
\begin{align}
  g(x_i, y_i) &= 
    \begin{cases}
      -1 \mbox{ (bluish)}, &\mbox{if } f(x_i, y_i) < 0 \\
      +1 \mbox{ (yellowish)}, &\mbox{otherwise}
    \end{cases},\\
    f(x, y) &= .7x^3 -.3x^2 + .9x + y - .1
\end{align}
$$

Now let's assume that we are running on a tight energy budget, and decide to use a cheap [Logistic Regression](http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html), and find out that the new equation and the plots for the `Predicted Labels` should be 

$$
\begin{align}
  g'(x_i, y_i) &= 
    \begin{cases}
      -1 \mbox{ (bluish)}, &\mbox{if } f'(x_i, y_i) < 0 \\
      +1 \mbox{ (yellowish)}, &\mbox{otherwise}
    \end{cases},\\
    f'(x, y) &= .744x + .0187y - 0.124
\end{align}
$$

![Logistic Dataset](/images/adaptive/synthetic_logit.png)

Notice that some "yellowish" labels bleed into the blue region and vice versa. The accuracy on the training data for the classifier is `94%`.

To separate the hard data points from the easy ones, we will compute the `hard_bias` as follows[^2]:

$$
\begin{align}
  H_{bias} &= \max{\left(\left|y'_i - y_i\right| \cdot \mathbb{1}_{g(x_i, y_i) \ne g'(x_i, y_i)}\right)}, \\
  y'_i &= \frac{-.744x - .124}{.0187}
\end{align}
$$

where $$\mathbb{1}_{z}$$ is an indicator function. Now we can specify the boundaries for the `Hardness Pseudo-Labels`:

$$
\begin{align}
  h(x_i, y_i) &= 
    \begin{cases}
      +1 \mbox{ (hard, red)}, &\mbox{if } |f'(x_i, y_i)| \le H_{bias} \\
      -1 \mbox{ (easy, blue)}, &\mbox{otherwise}
    \end{cases}
\end{align}
$$

![Hardness](/images/adaptive/synthetic_hardness.png)

If we analyze the test and train synthetic data, we find out that only a fraction of all the data points require complex classification.

![Ratio](/images/adaptive/synthetic_ratio.png)

### Chooser Function

in order to choose if we want to use a more expensive classifier or not, we use a "chooser" function -- another logistic regression function which would predict the "hardness" of the problem.



---

[^1]: Note that this is not really true. There are cases where the problem is very far from decision boundary, but is still misclassified, and vice-versa
[^2]: This `bias` is only for illustration purposes, in reality the "hardness" is subject to a little more complex decision making process. In particular $$\min_{g \in \mathbb{G}} \sum_i\sum_j{L_j(x_i, y_i)\mathbb{1}_{g(x_i)=j}}$$ is an appropriate function to train a "chooser" function with respect to loss $$L$$
