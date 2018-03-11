#Copy Systems

It is often the case that we want some Systems to simply copy data between Components they operate on. For example, a `PhysicsRect` Component may require the `x` and `y` properties of a `Location` component.

~~~
Location {
    var x;
    var y;
}

PhysicsRect {
    var x;
    var y;
    var rect;
    var mass;
}
~~~

A System that copies between these two Components would operate on `(Location, PhysicsRect)` nodes.

~~~
LocationPhysicsRectNode {
    var location:Location;
    var physicsRect:PhysicsRect;
}
~~~

In a regular ECS environment, a Copy System would likely iterate over all such nodes each frame, constantly updating the values. Not only is this a lot of work, it's a lot of wasted computation. 

~~~
CopyLocationToPhysicsSystem {
    var nodes = [];
    function register(node){
        nodes.push(node);
    }
    function update(){
        for(node in nodes){
            node.physicsRect.x = node.location.x;
            node.physicsRect.y = node.location.y;
        }
    }
}
~~~


Moreover, the order for when this System is added to our program is important relative to the order of when other systems are added. Specifically, this System must update after the `LocationSystem` updates the `x`,`y` values, and before the `PhysicsSystem` operates on the copied values.

### D-ECS for Copy Systems
In D-ECS, our System that registers a each node could simply assign the relation between the respective values of each Component.

~~~
CopyLocationToPhysicsSystem {
    function register(node){
        node.physicsRect.x = 
            function() { return node.location.x }
        node.physicsRect.y = 
            function() { return node.location.y }
    }
}
~~~

Note that we don't need an `update()` method for this System anymore. The System solely defines Component relationships at registration, and lets the Dataflow assignment handle the computation at a later time.

Because we have no `update()` method to call each frame, we have no more wasted computations each frame. 
Due to how Dataflow propogation works, our program would only copy over the `x` and `y` values between the `Location` and `PhysicsRect` Components when the values for `PhysicsRect.x` and `PhysicsRect.y` are requested: whether that be in the `PhysicsSystem` or somewhere further down the line that depends on the `PhysicsRect` data. And even then, it will only copy the values over for the times when `x` and `y` in `Location` have changed- otherwise Dataflow propogation guarantees we simply use the cached values from our most recent copy if there have not been any changes since our last copy.

And because we don't have an `update()` method anymore, it does not matter when we register this System with respect to the other Systems. We no longer have to worry about whether we add it before or after we add the `LocationSystem` and `PhysicsRectSystem` to our program.

[All Topics](https://github.com/dyarosla/D-ECS)
