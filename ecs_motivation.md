The Motivation for ECS
==

Entity Component Systems, or ECS for short, is a programming paradigm that eschews a lot of typically object-oriented principles (things like subclasses and data encapsulation) for a design that is more "data-driven".

Much like OOP, ECS is not standardized and comes in several *flavors*. But like OOP, it has several tenants that it is often defined by. 

The basic idea behind ECS is that we define all of our code in terms of Components, Entities, and Systems.

### Components
A Component is a bundle of data (whether it is defined as a `class`, `struct`, or `typedef`). For example, we may have a `Location` component.

~~~
Location {
    var x;
    var y;
}
~~~

We could also have a `Texture` component.

~~~
Texture {
    var fileName;
}
~~~

Most importantly, a Component only defines data, a "dumb data object" if you will. It does not define any methods for that data. As such, these variables are often `public`.

### Entities

An Entity is a bundle of Components. More precisely, it is an instance of a bundle of Components.

In ECS, we create an Entity and then assign it Components. Entities are often represented by an integer `id` that maps to a set of components.

~~~
var e = uniqueID();
addComponent(e, new Location(x:5, y:10));
addComponent(e, new Texture(fileName:"test.png"));
~~~

Like Components, Entities also do not have any methods.

### Systems

So far, we've seen that Components only hold data, and Entities only hold Components. Neither one actually defines any methods to work with the data.

Systems are where the "work" happens.

A System is a set of methods that operates on a Component or a set of Components. A System defines the way that data found in Components is transformed.

When we add a Component to an Entity, it will be registered with all Systems that operate on it. 

For example, we could have a `TextureSystem` that operates on `Texture` Components. 

If we add a `Texture` Component to an Entity, that `Texture` Component will be registered with the `TextureSystem`, at which point the `TextureSystem` could load the image defined by the `Texture`'s `fileName` property.

Note that a System operating on a Component is equivalent to a System operating on the Entity who owns that Component. Because Entities simply store bundles of Components, they themselves are never directly operated on.

The most interesting part of ECS are Systems that operate on sets of Components that belong to the same Entity (sometimes called Nodes).

For example, we could have a `RenderSystem` that operated on the Entities that owned the set of Components `<Location, Texture>`. Every frame, the `RenderSystem` could render the image defined by the `Texture` Component at the screen location defined by the `Location` Component.

### Why is this powerful?

By separating the definition of our Entities from their methods, we end up defining Entities purely by their set of Components. Thus, Entities are modular and easily open for extensibility and we can add new functionality to an Entity simply by adding new Components to it.

How those Entities operate, or interact with one another, is defined solely in the Systems that operate on the Entities' Components. Systems' sole responsibility is to define how the Components they operate on work. Because they operate on separate sets of Components, it lends them to becoming highly decoupled from other Systems, and more easily testable on their own.

The power of ECS is that it provides a framework to create modular, testable, decoupled, and ultimately, versatile programs.

[All Topics](https://github.com/dyarosla/D-ECS)
