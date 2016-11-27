---
layout: post
title: "Python Running Time Experiments"
categories: [python]
description:
tags: [python, runtime, algorithms]
comments: true
share: true
image:
  feature: http://i.imgur.com/C44iJeR.jpg
  credit: Ex Machina
---

I have just (re)started the [MIT OCW 6.006](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/),
and during the discussion of different
algorithms decided to check what are real run-times for different commonly used
methods on my machine. Continue reading to see the results...

<!-- more -->

The original code is located [here](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/readings/python-cost-model/timing.py).
There are minor changes to the code, and the modified version and the results are located
on [GitHub](https://github.com/zafartahirov/MOOCs/tree/master/MIT_ocw/6.006F11/JUNK).

Let me know if you find any inconsistencies!

## Methodology
All experiments are run on `MacBook Pro (Retina, 13-inch, Late 2013)` on
`2.4 GHz Intel Core i5` with `8 GB 1600 MHz DDR3` using `Python 2.7.12`.

For every test, `timeit` module was used to find the run time of function under
test. The parameters for the test are generated given some bounds, and the
result is fit to the best guess.
Difference in the results is caused by background processes
([source](https://docs.python.org/2/library/timeit.html#timeit.Timer.repeat)).

Here is an example of a run
{% highlight python %}
print "Test List-1: create an empty list"
spec_string = "1<=n<=10"
growth_factor = 2
print "Spec_string: ",spec_string, "by factors of", growth_factor
var_list, param_list = make_param_list(spec_string,growth_factor)
# f_list = ("n","1")
f_list = ("1",)
run_times = []
trials = 1000
for D in param_list:
  t = timeit.Timer("x = list()")
  run_times.append(t.timeit(trials)*1e6/float(trials))
fit(var_list,param_list,run_times,f_list)
{% endhighlight %}

The raw and formatted results:
{% highlight bash %}
Test List-1: create an empty list
Spec_string:  1<=n<=10 by factors of 2
var_list ['n']
Function list: ('1',)
run times:
n =      1 : 0.125885 microseconds
n =      2 : 0.125885 microseconds
n =      4 : 0.124931 microseconds
n =      8 : 0.124931 microseconds
Coefficients as interpolated from data:
 0.125405*1
(measuring time in microseconds)
Sum of squares of residuals: [  5.78285384e-05]
RMS error = 0.38 percent
{% endhighlight %}

| Operation | Description       | Asymptotic  | Real ($$\mu$$s) | Error (%) |
|:----------|:------------------|:------------|:----------------|:----------|
| `list()`  | Create empty list | $$O(1)$$    | $$0.13$$        | 0.38      |

## Results

$$n$$ is the variable number of elements - could be number of digits in a number
number of elements in an array, etc., while $$ k = n / 1000 $$

**Real values are rounded!!!**

### Integer

| Operation   | Description           | Asymptotic  | Real ($$\mu$$s)           | Error (%) |
|:------------|:----------------------|:------------|:--------------------------|:----------|
| `int('1'*n)`| String to Integer[^1] | $$O(n^2)$$  | $$0.2 - 2.7k + 7.2k^2 $$  | 1.9       |
| `repr(2**n)`| Integer to String[^1] | $$O(n^2)$$  | $$0.8 - 0.5k + 1.6k^2 $$  | 1.7       |
| `'%x'%x`    | Int to Hex            | $$O(n)$$    | $$0.1 - 1.0k $$           | 12        |
| `x+y`       | Addition/Subtraction  | $$O(n)$$    | $$0.1 - 0.04k $$          | 12        |
| `x*y`       | Multiplication        | $$O(n^{\log3})$$  | $$0.1 - 1.6k^{\log3} $$  | 17   |
| `x/y`       | Division (Remainder)  | $$O(n^2)$$  | $$0.8 - 2.0k^2 $$         | 5.8       |
| `pow(x,y,z` | Modular Exponentiation| $$O(n^3)$$  | $$ 1071.8 + 3107.9k^3 $$  | 1.7       |
| `2**n`      | Power of two          | $$O(1)$$    | $$ 0.02 $$                | 0.41      |

[^1]: I was expecting the conversion to be linear, but it is quadratic. After a brief search found [this SO answer](http://stackoverflow.com/a/1845764/3606192) that mentions Python conversion being $$O(n^2)$$

### String[^2]

| Operation       | Description         | Asymptotic  | Real ($$\mu$$s)   | Error (%) |
|:----------------|:--------------------|:------------|:------------------|:----------|
| `s[i]`          | Extract byte        | $$O(1)$$    | $$0.06$$          | 1.9       |
| `s+t`           | Concatenate         | $$O(n)$$    | $$0.04 + 75.2n$$  | 34        |
| `s[:n/2]`       | Extract sub string  | $$O(n)$$    | $$0.17 + 20.3n$$  | 25        |
| `s.translate()` | Translate string    | $$O(n)$$    | $$0.55 + 1.1k$$   | 5.1       |

[^2]: String operations have high error, and the real time varies a lot during benchmarking.

### List

| Operation       | Description         | Asymptotic | Real ($$\mu$$s)  | Error (%) |
|:----------------|:--------------------|:-----------|:-----------------|:----------|
| `list()`        | Create empty list   | $$O(1)$$   | $$0.13$$             | 0.65      |
| `L[i]`          | List lookup         | $$O(1)$$   | $$0.04$$             | 3.2       |
| `L.append(x)`   | Append in the end   | $$O(1)$$   | $$0.12$$             | 22        |
| `L.pop()`       | Pop the last        | $$O(1)$$   | $$0.24$$             | 17        |
| `L+M`           | Concatenate         | $$O(n)$$   | $$0.36 + 11.1k $$    | 9.2       |
| `L[:n/2]`       | Extract slice       | $$O(n)$$   | $$ 0.26 + 2.7k$$     | 5         |
| `L[:]`          | Copy                | $$O(n)$$   | $$-0.33 + 5.3k$$     | 2.5       |
| `L[:n/2] = M`   | Slice assignment    | $$O(n)$$   | $$-0.19 + 5.0k$$     | 1.7       |
| `del L[0]`      | Delete first        | $$O(n)$$   | $$-0.15 + 0.3k$$     | 27        |
| `L.reverse()`   | Reverse             | $$O(n)$$   | $$-0.15 + 0.4k$$     | 9.7       |
| `L.sort()`      | Sort array          | $$O(n\log{n})$$| $$-3.29 + 17.2k + 1.6k\log{k}$$ | 7.5 |

### Dict

| Operation       | Description         | Asymptotic | Real ($$\mu$$s)  | Error (%) |
|:----------------|:--------------------|:-----------|:-----------------|:----------|
| `dict()`        | Create empty dict   | $$O(1)$$   | $$0.15$$         | 0         |
| `D[i]`          | Dictionary lookup   | $$O(n)$$   | $$0.05 + 0.2n$$  | 2         |
| `D.copy()`      | Dictionary copy     | $$O(n)$$   | $$-19.5 + 37.9k$$| 23        |
| `D.items()`     | List items          | $$O(n\log{n})$$ | 29.2 - 188.9k + 191k  | 8.1 |
