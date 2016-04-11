---
layout: tutorial
title: Our First System
permalink: /tutorials/02-first-system
number: 2
---

In this tutorial, we will create our first `System` in `engo`. We will also be using basic input methods such as the
**mouse and keyboard**. 

> #### Recap
> Remember what we did in [the last tutorial](/tutorials/01-hello-world)? <br>
> We created a `Scene` which renders a huge *City* icon onto the screen. 

> #### Final Code
> The final code **for tutorial 2** is available 
> [here at GitHub](https://github.com/EngoEngine/TrafficManager/tree/02-first-system). 

### Step 1: The Concept
Before creating any `System`, it is often good to ask yourself: what is **the purpose** (or task) of this `System`? As
we are building a traffic-managing game which has multiple cities, it would make sense to create a system which creates
new cities. 

To keep it simple for the sake of this tutorial, we shall create a `System` which creates a new *City* at the location
of the cursor, whenever someone presses **F1**. 

### Step 2: Defining and Adding our System

#### Defining the System

Before adding any functions, let's start by creating a dummy `System` and adding it to our game. 

A `System` is anything that implements all these functions:
{% highlight go %}
// System is an interface which implements an ECS-System. A System
// should iterate over its Entities on `Update`, in any way 
// suitable for the current implementation.
type System interface {
	// Type returns a unique string identifier, usually 
	// something like "RenderSystem", "CollisionSystem"...
	Type() string
	// Priority is used to create the order in which Systems 
	// (in the World) are processed
	Priority() int

	// New is the initialisation of the System
	New(*World)
	// Update is ran every frame, with `dt` being the time
	// in seconds since the last frame
	Update(dt float32)

	// AddEntity adds a new Entity to the System
	AddEntity(entity *Entity)
	// RemoveEntity removes an Entity from the System
	RemoveEntity(entity *Entity)
}
{% endhighlight %}

Let's start off with a very simple implementation of this interface:

{% highlight go %}
type CityBuildingSystem struct {}

// Type returns a unique string identifier, usually 
// something like "RenderSystem", "CollisionSystem"...
func (*CityBuildingSystem) Type() string { 
    return "CityBuildingSystem" 
}

// Priority is used to create the order in which Systems 
// (in the World) are processed
func (*CityBuildingSystem) Priority() int { 
    return 0 
}

// AddEntity is called whenever an Entity is added to the
// World, which "requires" this System
func (*CityBuildingSystem) AddEntity(*ecs.Entity) {}

// RemoveEntity is called whenever an Entity is removed from
// the World, which "requires" this System
func (*CityBuildingSystem) RemoveEntity(*ecs.Entity) {}

// New is the initialisation of the System
func (*CityBuildingSystem) New(*ecs.World) {}

// Update is ran every frame, with `dt` being the time
// in seconds since the last frame
func (*CityBuildingSystem) Update(dt float32) {}

{% endhighlight %}

Our `CityBuildingSystem` builds *Cities* on top of the cursor. This requires use of the keyboard and mouse. At first, 
we don't need to worry about entities, so we'll ignore the `AddEntity` and `RemoveEntity` functions for now. 

Let's keep **code quality** in mind, and create a *separate package* for our Systems, called `systems`. This means we create
the directory `$GOPATH/github.com/EngoEngine/TrafficManager/systems`. Within this, we create a new file, with the 
exact contents of the `CityBuildingSystem` described above. However, we do add `package systems` to the top of the file.

#### Adding the System
Now that we've created the `System`, let's add it to our game. The usual way to add `System`s to a game, is by doing
it within the `Setup` function of the `Scene` you're going to use. Since we don't want this to interfere with the
big *City* icon we created in the previous tutorial, we are not only going to add the `System`, but also remove 
the `Entity` we created last week. 

{% highlight go %}
// Setup is called before the main loop starts. It allows you 
// to add entities and systems to your Scene.
func (*myGame) Setup(world *ecs.World) {
	engo.SetBackground(color.White)

	world.AddSystem(&systems.CityBuildingSystem{})
	world.AddSystem(&engo.RenderSystem{})
}
{% endhighlight %}

> ##### How do we know it works?
> At the moment, we can't be sure it actually works, right?
> Let's change the `New` function of our `CityBuildingSystem` to this:
> {% highlight go %}
// New is the initialisation of the System
func (*CityBuildingSystem) New(*ecs.World) {
	fmt.Println("CityBuildingSystem was added to the Scene")
}
{% endhighlight %}

> When we now run our game, we can see the white background, no *City* icon (because we removed it), and the
> console outputs *"CityBuildingSystem was added to the Scene"*. 

### Step 3: Figuring out when **F1** is pressed
We want to spawn a *City* whenever someone presses **F1**, so it makes sense that we want to know *when* this happens. 

For this, we will check "did the gamer press **F1**", on every frame. The `Update` function of our `CityBuildingSystem`
gets called every frame by the `World`. So our checking-code needs to be written there. Engo has a neat feature which
allows you to lookup the state of keys, such as:

* `Down()`: when the button is pressed; will be true as long as the user holds the button, 
* ` Up()`: opposite of `Down()`,
* `JustPressed()`: when the button was just pressed; will be true for one frame, and cannot become true again unless the 
button was released first, 
* `JustReleased()`: same as `JustPressed()`, but with releasing the button instead of pressing it,
* `State()`: returns a `string` representation of the four functions above.

We don't want to risk placing two cities on top of each other in a very short time period (it's very likely that the
key is pressed longer than one frame, because a frame usually lasts between 16.6ms and 6.94ms). Therefore, we shall use
the `JustPressed()` function. 

{% highlight go %}
// Update is ran every frame, with `dt` being the time
// in seconds since the last frame
func (*CityBuildingSystem) Update(dt float32) {
	if engo.Keys.Get(engo.F1).JustPressed() {
		fmt.Println("The gamer pressed F1")
	}
}
{% endhighlight %}

> #### How do we know it works?
> Let's run the game, and press **F1** and find out! It should print "The gamer pressed **F1**" just once, when we
> hold the F1-key. It should print it twice when we double-tap it. It shouldn't print it as long as we don't press F1. 

### Spawning a *City*
Now we know *when* to spawn a *City*, we can actually write the code to do so. 

Remember the code we used for the large *City*-icon we removed? 
{% highlight go %}
entity := ecs.NewEntity("RenderSystem")

entity.AddComponent(&engo.SpaceComponent{
    Position: engo.Point{10, 10},
    Width: 303,
    Height: 641,
})

texture := engo.Files.Image("city.png")
entity.AddComponent(engo.NewRenderComponent(
    texture,
    engo.Point{1, 1},
    "city texture",
))

world.AddEntity(entity)

{% endhighlight %}

There are a few things we want to change, before using this in our `CityBuildingSystem`:

* The size: 303 by 641 is a bit too big. Let's change this to 30 by 64. 
* The location: we want to spawn it at the location of the cursor
* The `world` is unknown in the `Update` function, so we have to work around that

#### The size
As stated, we can easily change the size, by changing the numbers. However, changing the values at the 
`SpaceComponent` isn't enough. The texture is still way too big; we can change this by setting the scale to `0.1` 
instead of `1`:

{% highlight go %}
entity.AddComponent(&engo.SpaceComponent{
    Position: engo.Point{10, 10},
    Width: 30,
    Height: 64,
})

texture := engo.Files.Image("city.png")
entity.AddComponent(engo.NewRenderComponent(
    texture,
    engo.Point{0.1, 0.1},
    "city texture",
))
{% endhighlight %}

#### The location
In order to spawn them at the correct location, we need to know where the cursor is. Our first guess might be to use
the `engo.Mouse` struct which is available. However, this one returns the actual `(X, Y)` location relative to the 
screen size, not the in-game grid system. We have a special `MouseSystem` available for just that. 

##### The `MouseSystem`
The first thing you want to do, is add the `MouseSystem` to your `Scene`:

{% highlight go %}
// Setup is called before the main loop starts. It allows you to add entities and systems to your Scene.
func (*myGame) Setup(world *ecs.World) {
	engo.SetBackground(color.White)

	world.AddSystem(&engo.MouseSystem{})
	world.AddSystem(&engo.RenderSystem{})
	
	world.AddSystem(&systems.CityBuildingSystem{})
}
{% endhighlight %}

*Note that we added the other systems before the `CityBuildingSystem`. This is to ensure any systems we might depend
upon, are already initialized when we're initializing the `CityBuildingSystem`.*

THe `MouseSystem` is mainly written to keep track of mouse-events for entities; you can check whether your `Entity`
has been hovered, clicked, dragged, etc. In order to use it, we therefore need an `Entity` which uses the `MouseSystem`. 
This one needs to hold a `MouseComponent`, at which the results/data will be saved. 

We first will update our `CityBuildingSystem` to contain the new `Entity`:

{% highlight go %}
type CityBuildingSystem struct {
	mouseTracker *ecs.Entity
}
{% endhighlight %}

Then, we want to ensure this `mouseTracker` entity gets initialized and added to the `World` whenever we start using
this `CityBuildingSystem`:

{% highlight go %}
// New is the initialisation of the System
func (cb *CityBuildingSystem) New(w *ecs.World) {
	fmt.Println("CityBuildingSystem was added to the Scene")
	
	cb.mouseTracker = ecs.NewEntity("MouseSystem")
	cb.mouseTracker.AddComponent(&engo.MouseComponent{Track: true})
	w.AddEntity(cb.mouseTracker)
}
{% endhighlight %}

Note that we added the `Track: true` variable to the `MouseComponent`. This allows us to know the position of the
mouse, regardless of where it is. If we were to leave it at `false` (the default), it would only contain anything
useful if the mouse was hovering the `Entity`. 

##### Getting information from the MouseSystem
We now have a reference to the `mouseTracker` entity. However, how do we access the `MouseComponent` we added to the
entity? There are two methods for that, both available at the `Entity`-level: `entity.Component` and 
`engi.ComponentFast`. The first one uses `reflect`, and therefore allows for more readable syntax. The second one
doesn't use `reflect`, and is therefore in return **a lot** faster (think 100-1000 times faster). 

First we shall explain the `ComponentFast` method:
{% highlight go %}
var (
    mouse *engo.MouseComponent
    ok bool
)

if mouse, ok = cb.mouseTracker.ComponentFast(mouse).(*engo.MouseComponent); !ok {
    return
} 
{% endhighlight %}

You simply create a variable `*engo.MouseComponent` and a boolean variable. Then you set assign them to the return-value
of `ComponentFast` (meaning: `ComponentFast` returns some kind of `Component`-interface, and we'll be casting it to
`*engo.MouseComponent` to be sure it's the correct one). If this somehow fails, `ok` will be false, and we'll stop
processing the `CityBuildingSystem` for this frame - because we cannot do anything if we don't know where the cursor is. 

Now as for the `Component` method:

{% highlight go %}
var mouse *engo.MouseComponent
if !cb.mouseTracker.Component(&mouse) {
    return
}
{% endhighlight %}

The syntax looks much cleaner: you simply define it, and try to fill it with the correct information. If this fails, 
we again stop processing the `CityBuildingSystem` for this frame. *Again, note that this runs much slower.* 

If we were to use the `ComponentFast` method, the entire `Update` function would look something like this:

{% highlight go %}
// Update is ran every frame, with `dt` being the time
// in seconds since the last frame
func (cb *CityBuildingSystem) Update(dt float32) {
  if engo.Keys.Get(engo.F1).JustPressed() {
    fmt.Println("The gamer pressed F1")
    entity := ecs.NewEntity("RenderSystem")

    var (
      mouse *engo.MouseComponent
      ok bool
    )

    if mouse, ok = cb.mouseTracker.ComponentFast(mouse).(*engo.MouseComponent); !ok {
      return
    }

    entity.AddComponent(&engo.SpaceComponent{
      Position: engo.Point{mouse.MouseX, mouse.MouseY},
      Width: 30,
      Height: 64,
    })

    texture := engo.Files.Image("city.png")
    entity.AddComponent(engo.NewRenderComponent(
      texture,
      engo.Point{0.1, 0.1},
      "city texture",
    ))

    cb.world.AddEntity(entity)
  }
}
{% endhighlight %}

Now, if we were to run this whole project, and place a few cities using the **F1**-key, we would get something like
this:

<figure class="callout text-center">
<a href="/img/tutorials/02/cities.png" target="_blank">
<img alt="City texture" src="/img/tutorials/02/cities.png" style="height:400px">
<figcaption>The result of tutorial 2, being able to place multiple cities</strong></figcaption>
</a>
</figure>