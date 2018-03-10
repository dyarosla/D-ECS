Propogation in Dataflow
==

You could imagine that it would be wasteful to always propogate a change. For example:

~~~
a = 0
b = complex(a)

for(i in 0...100) {
   a = random(100)
}

log(b)
~~~

We can see that `b` is dependent on `a` but requires computing via some aptly named `complex` function. 

* Aside: *Let us assume that `complex(i)` is deterministic. That is, running it multiple times with the same value for `i` yields the same result.*

However, the value of `b` is not *used* until the `log` statement. If we naively broadcasted that we need to recompute `b` on every change to `a`, we'd call `complex(a)` 100 times, bogging down our system.

Instead, our ideal assignment operator would only compute the value of a variable when the value of that variable is requested. Whether that is to `log` something, or use the variable to `draw` something onto the screen; anywhere where the value is *displayed*, whether that be to the end user or some other application.

So in the following case, nothing would be computed until the log statement.

~~~
a = 0
b = a + 2
c = a + b

log(c)
~~~

The call to `log(c)` would request the value of `c` which would request the value of `a + b`. This would in turn request the values of `a` and `b`, which would in turn force `b` to compute the value of `a + 2`. Once `b` is computed, `c` can finally be evaluated. 

The call to `log(c)` would *unwrap* `c`.

This brings us to one more thought. What if we had

~~~
a = complex(10)
b = a + 1

for(i in 0...100) {
   log(b)
}
~~~

We don't want to *fully unwrap* `b` 100 times. That would require us to recompute the value of `a` 100 times (again, assuming `complex(i)` is deterministic).

Our ideal assignment operator would both only *unwrap* variables when values are requested *and* cache computations that we've already done.

In the above example, `log(b)` would compute the value of `a` once during the 0th iteration of the loop, as well as cache the result of `a + 1`. Each subsequent iteration of the loop would simply log the same cached value for `b`.

We can conclude that for efficient dataflow propogation, we want assignment that both only computes values when they are needed, as well as caches results of computations between retrievals.

Next up: [The Motivation for ECS](https://github.com/dyarosla/dataflow/blob/master/ecs_motivation.md)

[All Topics](https://github.com/dyarosla/D-ECS)
