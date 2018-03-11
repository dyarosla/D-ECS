Assemblages in ECS
==

From the previous article, we've defined an Entity to be an instance of a bundle of Components. That means that whenever we create a new Entity, we have to add to it all the Components we want it to be defined by one by one.

However, there are certain times when we come to find patterns of Components we want Entities to have, that define specific kinds of Entities. 

As a hypothetical example, we might find that "UI Elements" are Entities which require the Components `<Layout, TouchTracker, Renderable>` to have all the base functionality we expect from a "UI Element". A "UI Element" in this case can be considered a *blueprint* for certain types of Entities.

This *blueprint* has taken on the term Assemblage in ECS literature. You can think of the relationship between Assemblages to Entities as akin to the relationship between Classes to instances of those classes.

An Assemblage simply defines a bundle of Component classes.

~~~
UIElement {
    Transform,
    TouchTracker,
    Renderable
}

ecs.assemble(entity, UIElement);
~~~

The neat thing about Assemblages is that they can stack. If we have another Assemblage that defines another "blueprint", we can add it to the Entity as well. They inherently allow the concept of "multiple-inheritance" for Entities; that is, an Entity can be both a `UIElement` and a `GameObject` if it so chooses. And of course, more functionality can be added onto our Entity at any time with plain old `addComponent` calls as well.

[All Topics](https://github.com/dyarosla/D-ECS)
