The Motivation for Dataflow
=

Let's talk about Dataflow programming.

In most languages you'd have assignment operators like this:

~~~
a = 5
b = a + 2
log(b) // 7
~~~

`b` is set to the value of `a + 2` at a single point in time- when the assignment line is executed. If we then follow that up with

~~~
a = 2
log(b) // 7
~~~

`b` retains the value that it was initially assigned.

Imagine that instead we want `b` to always be `a + 2` for whatever value `a` is. For example, imagine `b` is a variable that holds the pixel layout coordinate of an object relative to `a`, and that always needs to be `2` pixels greater.

Let's use the `=` operator to denote that kind of relationship instead. Wouldn't it be great to have the following:

~~~
a = 5
b = a + 2
a = 2
log(b) // 4
~~~

We can freely update `a` as much as we like, and we'd know that the value of `b` is always `a + 2`.

--

Perhaps this isnt a big deal if we only have two variables. But what if we have more? What if the value of `a` was determined elsewhere?

~~~
i = 2
j = 4
a = fibonacci(i+j)
b = a + 2
~~~

The value of `b` is now dependent on `a`, which itself is dependent on `i` and `j`. Wouldn't it be nice for assignment to work the way we described? Any change could propogate to any variable as needed.

That's the idea behind Dataflow programming. Variables are not assigned values so much as they are assigned functions, that may or may not depend on other variables.

~~~
i = function() { return 2 }
j = function() { return 4 }
a = function() { return fibonacci(i + j) }
b = function() { return a + 2 }
~~~
