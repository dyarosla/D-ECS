Runtime Dependencies
==

ECS presents a few unique challenges due to how it decouples Components and Systems.

One potential problem you can run into is that, during testing, some Components won't seem to "do" anything, caused by accidentally forgetting to register a System that acts on that Component type.

Another potential, and more serious problem, is that because multiple Systems can act on several shared Components, it can be difficult to debug where data values for Components come from. As Component data is public, it can be modified by any number of Systems, and figuring out the order of things can become fairly complex.

Both of the above problems exist in "regular" ECS due to the nature of ECS. ECS lets us more freely compose behaviour between objects at the expense of encapsulation. Technically, the problem is due to making Components' data public, and by doing so, exposing writeability from any System.

At this point one might jump to the conclusion that we should make Components data variables private instead, adopting *setters* and *getters* that wrap Components` variables. Systems could then use those accessor methods instead. In reality, this simply adds a layer of complexity over the same interaction. We've simply changed the problem of directly modifying variables from any System, to the problem of modifying variables through an intermediary *setter* from any System. This doesn't actually solve anything.


### Dataflow Variable Dependencies

Dataflow variables provide a solution to both of the above problems due to their nature.

We've already seen that Dataflow variables are essentially functions that may or may not depend on other variables.

Let's take the example below:

~~~
a = function(){ return 2 }
b = function(){ return a + 1 }
c = function(){ return a + b }
~~~

Behind the scenes Dataflow variables must store a dependency graph of some sort to propogate changes. When we assign the above functions to the variables `a`,`b` and `c`, we're consequently stating that the value of `b` depends on the value of `a`, the value of `c` depends on `a` and `b`, and `a` does not have any variable dependencies.

We summarize the above in the following chart:

~~~
Dependencies:
a <- []
b <- [a]
c <- [a,b]
~~~

### D-ECS for Debugging Runtime Dependencies

If we use Dataflow variables in our ECS' Components, two things happen.

First, Systems that act on Components must assign value to Component's data through assignment to the Components' Dataflow variables. Those assignments will take the form of functions that may or may not rely on other variables, whether those variables are part of the System itself or variables that are part of other Components.

This in turn allows Component variables to be queried at runtime to see where their values are being assigned.


Below, we look at an example query for the dependencies of a `Transform` Component of an Entity with id `10`:


~~~
> ecs.getDependencies(entity:10, component:Transform)

10.Transform
---------
x : 100      <- [10.Position.x]
y : 100      <- [10.Position.y]
scaleX : 1   <- [5.Screen.resolution]
scaleY : 1   <- [10.Transform.scaleX]
~~~


We can see that `x` and `y` are dependant on the `Position` Component of the same Entity, the `scaleX` value is dependant on the `5` Entity's `Screen` Component's `resolution` variable, and lastly the `Transform.scaleY` value is dependent on the same `Transform` Component's `scaleX` variable.

We can take this a step further and log the function assigned to each variable.

~~~
> ecs.getValues(entity:10, component:Transform)

10.Transform
---------
x : 100      = function(){ return position.x }
y : 100      = function(){ return position.y }
scaleX : 1   = function(){ return screen.resolution == HD ? 1 : 2 }
scaleY : 1   = function(){ return scaleX }
~~~

This makes runtime debugging both of our original ECS problems much simpler.

If a Component seems to "do nothing", say for instance, it's transformation on screen seems to not update, our query could return:

~~~
> ecs.getDependencies(entity:10, component:Transform)

Transform
---------
x : 0      <- []
y : 0      <- []
scaleX : 1 <- [5.Screen.resolution]
scaleY : 1 <- [10.Transform.scaleX]
~~~

Here, `x` and `y` have no dependencies, and their values are `0`. We may expect that their values should be dependent on the `Position` Component values, and can go ahead and check whether we have registered a System that acts on `<Position,Transform>` Nodes.

We can similarly solve the second problem of figuring out which Systems affect which values by looking at the dependencies of a Component at runtime, and making changes if need be based on that information.

Say that in the above example we instead got the following result:

~~~
> ecs.getDependencies(entity:10, component:Transform)

10.Transform
---------
x : 0        <- [10.Position.x]
y : 0        <- [10.Position.y]
scaleX : 1   <- [5.Screen.resolution]
scaleY : 1   <- [10.Transform.scaleX]
~~~

Clearly we've correctly hooked up the `Position` to the `Transform` Components, but we're still getting the non-expected value of `0` for `x` and `y`. We would go ahead and explore the `Position` component at this point.

~~~
> ecs.getDependencies(entity:10, component:Position)

10.Position
---------
x : 0        <- []
y : 0        <- []

> ecs.getValues(entity:10, component:Position)

10.Position
---------
x : 0        = null
y : 0        = null
~~~

We see that the `Position` values aren't themselves being set as we expect them. We would go ahead and fix that up in a System that acts on `Position` Components.

In this way, we've seen that adopting Dataflow variables in our ECS allows us to more easily debug value propogation between Components and Systems, and mitigate the pain points that come with making Components' data public.

[All Topics](https://github.com/dyarosla/D-ECS)
