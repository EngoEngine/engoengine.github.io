---
layout: tutorial
title: Camera Movement
permalink: /tutorials/03-camera-movement
number: 3
---

In this tutorial, we will enlarge our world view by moving around using the `CameraSystem`. 

> #### Recap
> Remember what we did in [the last tutorial](/tutorials/02-first-system)? <br>
> We created a `CityBuildingSystem` which allows you to build *Cities* anywhere whenever you press **F1**. 

> #### Final Code
> The final code **for tutorial 3** is available 
> [here at GitHub](https://github.com/EngoEngine/TrafficManager/tree/03-camera-movement). 

### The CameraSystem
The `CameraSystem` is unlike other systems we've seen (`CityBuildingSystem`, `RenderSystem`, `MouseSystem`). Instead,
this system is given to each `Scene`, automatically. It allows you to 
change the location from which you're looking onto the game. As this is a 2D-game engine, we're only allowing you to 
change the X-axis, Y-axis and Z-axis (viewing distance). The angle itself isn't changeable. 

> #### Note
> You may not add the `CameraSystem` manually to the `World`. This happens automatically. 

There are a few ways to use this `CameraSystem`, and each one will be discussed separately:
* By adding the `engo.KeyboardScroller`, you can move around using predefined keys;
* By adding the `engo.EdgeScroller`, you can move around by moving your cursor close to the edges of the screen;
* By adding the `engo.MouseZoomer`, you can zoom in/out by using the scroll-wheel on your mouse;
* By sending a `CameraMessage` through the `Mailbox`. What the Mailbox is, will also be explained. 

#### KeyboardScroller
The `KeyboardScroller` is another System - one that listens for keyboard input and moves the screen accordingly. 

Let's add one in the `Setup` function of our game:
{% highlight go %}
// Setup is called before the main loop starts. It allows you to add entities and systems to your Scene.
func (*myScene) Setup(world *ecs.World) {
	engo.SetBackground(color.White)

	world.AddSystem(&engo.MouseSystem{})
	world.AddSystem(&engo.RenderSystem{})
	world.AddSystem(engo.NewKeyboardScroller(400, engo.W, engo.D, engo.S, engo.A))

	world.AddSystem(&systems.CityBuildingSystem{})
}

{% endhighlight %}

The magic number `400` here, is the speed at which the camera should move. It's mostly a relative number, meaning:
at zoom level 1x (so everything is at 100%), it will move 400 units per second. You can change this to any value 
you feel fits to your game. 

You can now move around using the WASD-keys. If we also want to allow the use of the arrow keys, we can use the
`BindKeyboard` method, available on the `KeyboardSystem`:

{% highlight go %}
// Setup is called before the main loop starts. It allows you to add entities and systems to your Scene.
func (*myScene) Setup(world *ecs.World) {
	engo.SetBackground(color.White)

	world.AddSystem(&engo.MouseSystem{})
	world.AddSystem(&engo.RenderSystem{})
	
	kbs := engo.NewKeyboardScroller(400, engo.W, engo.D, engo.S, engo.A)
	kbs.BindKeyboard(engo.ArrowUp, engo.ArrowRight, engo.ArrowDown, engo.ArrowLeft)
	world.AddSystem(kbs)

	world.AddSystem(&systems.CityBuildingSystem{})
}
{% endhighlight %}


#### EdgeScroller
The `EdgeScroller` is just like the `KeyboardScroller`, a plug-and-play system which we can easily add to our game. 

Its usage is pretty much the same as the `KeyboardScroller`:
{% highlight go %}
world.AddSystem(&engo.EdgeScroller{400, 20})
{% endhighlight %}

It uses the same scroll-speed as the KeyboardScroller (`400`) in this case. The magic number `20` here, is the number
of pixels the cursor has to be near the edge of the screen, in order to count as being "near the edge". The higher the 
number, the sooner this system will decide to move the camera around. 

#### ScrollZoomer
As with the other two, the `ScrollZoomer` is a `System` we can add to our game. 

{% highlight go %}
world.AddSystem(&engo.MouseZoomer{-0.125})
{% endhighlight %}

The magic number `-0.125` is the zoom speed. The fact that this number is negative, indicates that scrolling down, 
means zooming out.  

#### Using CameraMessages and the Mailbox
The `Mailbox` in `engo` is a place where systems can `Listen` for certain messages types, and other systems can send
such messages. In this specific scenario, the `CameraSystem` is constantly listening for `CameraMessage` messages to
arrive, and moves the camera around accordingly. The `KeyboardScroller`, `EdgeScroller` and `ZoomScroller` all make
use of this: they all send `CameraMessage` messages. 

> Each `Scene` has one **Mailbox**, one **World** and one **CameraSystem**. The mailbox of the current `Scene` can
> always be accessed via `engo.Mailbox`. The `World` will be given via the `Setup` and `New` methods, and the 
> `CameraSystem` can be invoked by using `CameraMessage` messages. 

Let's look at a few `CameraMessage` messages, shall we? The comments should speak for themselves

{% highlight go %}
// A message which states: move 100 units to 
// the bottom on the Y-axis. 
Mailbox.Dispatch(CameraMessage{
    Axis:        YAxis, 
    Value:       100, 
    Incremental: true,
})

// A message which states: go to Y-value 100
Mailbox.Dispatch(CameraMessage{
    Axis:        YAxis, 
    Value:       100, 
    Incremental: false,
})

// A message which states: zoom out a bit
Mailbox.Dispatch(CameraMessage{
    Axis:        ZAxis, 
    Value:       -1, 
    Incremental: true,
})

// A message which states: zoom in a bit
Mailbox.Dispatch(CameraMessage{
    Axis:        ZAxis, 
    Value:       2, 
    Incremental: true,
})

// A message which states: let's view everything at 100% 
// zoom (default)
Mailbox.Dispatch(CameraMessage{
    Axis:        ZAxis, 
    Value:       1, 
    Incremental: false,
})

// A message which states: move 100 units to the right
Mailbox.Dispatch(CameraMessage{
    Axis:        XAxis, 
    Value:       100, 
    Incremental: true,
})
{% endhighlight %}

What if we wanted to listen for messages?

{% highlight go %}
Mailbox.Listen("CameraMessage", func(msg Message) {
    // Do things with msg
    // If you want to access other values than it's
    // type, you'll want to use type-assertions
    
    // e.g.
    // camMsg, ok := msg.(CameraMessage)
    // if !ok {
    //    return
    // }
})
{% endhighlight %}

> Note that we're using the string `"CameraMessage"` here. A lot of interfaces in `engo` have a `Type() string` method. 
> These will always be used to uniquely define a type (to avoid using the slow `reflect`). This basically means: *call
> this function as well, whenever you receive a message which says it is of type `CameraMessage`.* 

### World Bounds
You may have noticed you cannot zoom-out indefinitely, and that you cannot move to the left/right/top/bottom as much
as you would've liked. This is because `engo` attempts to find out which "world bounds" would suit the game. This 
usually means it looks at the window size, and uses that as a guideline. However, you will most likely always want to
set `engo.WorldBounds` to something suitable for your game. This is usually the size of your map, so people can't 
move away from it. `WorldBounds` requires you to define two points: the upper-left corner (`WorldBounds.Min`), and
the lower-right corner (`WorldBounds.Max`). 

> We won't be setting it to anything now, because we have no "map" at the moment. 

