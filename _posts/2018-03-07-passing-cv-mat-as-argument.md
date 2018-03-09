---
layout: post
title: "Passing cv::Mat as argument"
modified: 2018-03-09 12:56:46 -0800
tags: [c++, OpenCV]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: true
---

Often times when we pass `cv::Mat`, we forget one important thing: `OpenCV` matrix does not respect the `const` modifier.
In this post I would like to discuss what happens when `cv::Mat` is passed around.

<!-- more -->

## What is `cv::Mat`?

## Changing the header of `cv::Mat`
Consider the following examples[^1], where we want to change what the datastructure is.
Note that we are trying to modify the HEADER of the `cv::Mat`, hence the behavior.

---

### Passing a copy -- `cv::Mat`.
This is totally OK, but the scoping rules apply -- only local variable will be changed, and no changes will happen outside of the function.

```c++
void Foo(cv::Mat out) {
    out = cv::Mat::ones(4, 4, CV_32F);  // OK, but only local changes happen.
}
```

---

### Passing a reference -- `cv::Mat&`.
This is totally fine, and the changes will propagate to outside of the function.

```c++
void Bar(cv::Mat& out) {
    out = cv::Mat::ones(4, 4, CV_32F);  // OK, Changes happen everywhere.
}
```

---

### Passing a `const` -- `const cv::Mat`.
This is NOT OK, even for local changes.

```c++
void Baz(const cv::Mat out) {
    out = cv::Mat::ones(4, 4, CV_32F);  // Error!!! Cannot change the header!
}
```

---

### Passing a `const` reference -- `const cv::Mat&`.
This is NOT OK!

```c++
void Qux(const cv::Mat& out) {
    out = cv::Mat::ones(4, 4, CV_32F);  // Error!!! Cannot change the header!
}
```

---

From the above examples, we see that `cv::Mat` guarantees that the header of the data structure will respect the `const` modifiers. However, if we look at [the header for the `core.hpp`](https://github.com/opencv/opencv/blob/2.4.13/modules/core/include/opencv2/core/core.hpp#L1980) we notice that the data in the structure `cv::Mat` is stored as a raw pointer:

{%highlight c++ %}
int dims;
//! the number of rows and columns or (-1, -1) when the matrix has more than 2 dimensions
int rows, cols;
//! pointer to the data
uchar* data;
{%endhighlight %}

Because of that, when passing the `cv::Mat` one has to be extra careful.
Because the way the data structure is implemented, passing it around as `const` or `const &` has slightly different behavior from what is expected.

---

### Changing the data in the `cv::Mat`

Now take a look at the functions below, no matter what is being passed, the modifications are allowed -- `cv::Mat` doesn't respect the const modifiers![^2]


```c++
void Foo(cv::Mat out) {
    out.data[0] = 5;  // OK globally!
}

void Bar(cv::Mat& out) {
    out.data[0] = 5;  // OK globally!
}

void Baz(const cv::Mat out) {
    out.data[0] = 5;  // OK globally!
}

void Qux(const cv::Mat& out) {
    out.data[0] = 5;  // OK globally!
}
```

---

### What about constant methods?

One might say that the examples above are artificial, and anyone writing code, will see it immediately.
However, there are cases when the changes will happen in places where you won't expect it.

```c++
struct Foo {
    void bar(const cv::Mat& out) const {
        cv::Mat baz = out(cv::Rect(0, 0, 10, 10));
        cv::cvtColor(baz, baz, cv::COLOR_BGR2RGB);
        // Although we don't modify the `out`, and `baz` is only a subset,
        // it is still changed!!!
    }
}
// ...
int main() {
    // ...
    Foo foo;
    foo.bar(image); // The colorspace of `image` is flipped now!!!
    // ...
}
```

## Conclusion

When handling the `cv::Mat` -- be very careful, and treat it as a smart pointer :).

[^1]: https://stackoverflow.com/a/23486280/3606192
[^2]: Technically it does -- `cv::Mat` is never changed -- only the data it's pointing to.