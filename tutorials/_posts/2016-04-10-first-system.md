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
	// Update is ran every frame, with `dt` being the time
	// in seconds since the last frame
	Update(dt float32)

	// Remove removes an Entity from the System
	Remove(ecs.BasicEntity)
}
{% endhighlight %}

Let's start off with a very simple implementation of this interface:

{% highlight go %}
type CityBuildingSystem struct {}

// Remove is called whenever an Entity is removed from the World, in order to remove it from this sytem as well
func (*CityBuildingSystem) Remove(ecs.BasicEntity) {}

// Update is ran every frame, with `dt` being the time
// in seconds since the last frame
func (*CityBuildingSystem) Update(dt float32) {}

{% endhighlight %}

Our `CityBuildingSystem` builds *Cities* on top of the cursor. This requires use of the keyboard and mouse. At first, 
we don't need to worry about entities, so we'll ignore the `Remove` function for now. We haven't added any entities,
so we can safely ignore removing them.  

Let's keep **code quality** in mind, and create a *separate package* for our Systems, called `systems`. This means we create
the directory `$GOPATH/github.com/EngoEngine/TrafficManager/systems`. Within this, we create a new file, with the 
exact contents of the `CityBuildingSystem` described above. However, we do add `package systems` to the top of the file.

#### Adding the System
Now that we've created the `System`, let's add it to our game. The usual way to add `System`s to a game, is by doing
it within the `Setup` function of the `Scene` you're going to use. Since we don't want this to interfere with the
big *City* icon we created in the previous tutorial, we are not only going to add the `System`, but also remove 
the `Entity` we created in the last tutorial. 

{% highlight go %}
// Setup is called before the main loop starts. It allows you 
// to add entities and systems to your Scene.
func (*myScene) Setup(world *ecs.World) {
	common.SetBackground(color.White)
	world.AddSystem(&common.RenderSystem{})
	
	world.AddSystem(&systems.CityBuildingSystem{})
}
{% endhighlight %}

> ##### How do we know it works?
> At the moment, we can't be sure it actually works, right?
> Each system can optionally implement `New(*eces.World)`, which will be called whenever the System is added to the
> scene. 
> 
> Let's create a `New` function for our `CityBuildingSystem`  like this:
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

First, we need to tell the Engo to listen for the F1 key press. We'll do this by using the 
`engo.Input.RegisterButton(name string, keys Key...)` function. Multiple keys can be be assigned to one identifier.

Add the following line to the `Setup` function for your `Scene`.
{% highlight go %}
func (*myScene) Setup(world *ecs.World) {
	engo.Input.RegisterButton("AddCity", engo.F1)
	common.SetBackground(color.White)
	world.AddSystem(&common.RenderSystem{})
	
	world.AddSystem(&systems.CityBuildingSystem{})
}
{% endhighlight %}

We will check "did the gamer press **F1**", on every frame. The `Update` function of our `CityBuildingSystem`
gets called every frame by the `World`. So our checking-code needs to be written there. Engo has a neat feature which
allows you to lookup the state of keys, such as:

* `Down()`: when the button is pressed; will be true as long as the user holds the button, 
* `JustPressed()`: when the button was just pressed; will be true for one frame, and cannot become true again unless the 
button was released first, 
* `JustReleased()`: same as `JustPressed()`, but with releasing the button instead of pressing it,

We don't want to risk placing two cities on top of each other in a very short time period (it's very likely that the
key is pressed longer than one frame, because a frame usually lasts between 16.6ms and 6.94ms). Therefore, we shall use
the `JustPressed()` function. 

{% highlight go %}
// Update is ran every frame, with `dt` being the time
// in seconds since the last frame
func (*CityBuildingSystem) Update(dt float32) {
	if engo.Input.Button("AddCity").JustPressed()  {
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
city := City{BasicEntity: ecs.NewBasic()}

city.SpaceComponent = common.SpaceComponent{
    Position: engo.Point{10, 10},
    Width: 303,
    Height: 641,
}

texture, err := common.LoadedSprite("textures/city.png")
if err != nil {
    log.Println("Unable to load texture: " + err.Error())
}

city.RenderComponent = common.RenderComponent{
    Drawable: texture,
    Scale:    engo.Point{1, 1},
}

for _, system := range world.Systems() {
    switch sys := system.(type) {
    case *common.RenderSystem:
        sys.Add(&city.BasicEntity, &city.RenderComponent, &city.SpaceComponent)
    }
}
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
city.SpaceComponent = common.SpaceComponent{
    Position: engo.Point{10, 10},
    Width: 30,
    Height: 64,
}

texture, err := common.LoadedSprite("textures/city.png")
if err != nil {
    log.Println("Unable to load texture: " + err.Error())
}

city.RenderComponent = common.NewRenderComponent(
    Drawable: texture,
    Scale: engo.Point{0.1, 0.1},
)
{% endhighlight %}

#### The location
In order to spawn them at the correct location, we need to know where the cursor is. Our first guess might be to use
the `common.Mouse` struct which is available. However, this one returns the actual `(X, Y)` location relative to the 
screen size, not the in-game grid system. We have a special `MouseSystem` available for just that. 

##### The `MouseSystem`
The first thing you want to do, is add the `MouseSystem` to your `Scene`:

{% highlight go %}
// Setup is called before the main loop starts. It allows you to add entities and systems to your Scene.
func (*myGame) Setup(world *ecs.World) {
	engo.Input.RegisterButton("AddCity", engo.F1)
	common.SetBackground(color.White)

	world.AddSystem(&common.RenderSystem{})
	world.AddSystem(&common.MouseSystem{})
	
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
type MouseTracker struct {
    ecs.BasicEntity
    common.MouseComponent
}

type CityBuildingSystem struct {
	mouseTracker MouseTracker
}
{% endhighlight %}

Then, we want to ensure this `mouseTracker` entity gets initialized and added to the `World` whenever we start using
this `CityBuildingSystem`:

{% highlight go %}
// New is the initialisation of the System
func (cb *CityBuildingSystem) New(w *ecs.World) {
	fmt.Println("CityBuildingSystem was added to the Scene")
	
	cb.mouseTracker.BasicEntity = ecs.NewBasic()
	cb.mouseTracker.MouseComponent = common.MouseComponent{Track: true}
	
	for _, system := range w.Systems() {
		switch sys := system.(type) {
		case *common.MouseSystem:
			sys.Add(&cb.mouseTracker.BasicEntity, &cb.mouseTracker.MouseComponent, nil, nil)
		}
	}
}
{% endhighlight %}

Note that we added the `Track: true` variable to the `MouseComponent`. This allows us to know the position of the
mouse, regardless of where it is. If we were to leave it at `false` (the default), it would only contain anything
useful if the mouse was hovering the `Entity`. We are also giving two parameters `nil` within the `Add` function
of the `MouseSystem`. That is because (as described in the documentation of that `Add` function), one can also provide
a `SpaceComponent` and `RenderComponent`, if one wants to know specifics about that particular entity (like 
hover-events). This is not our intention at the moment, we just want to know about the position of the cursor. Therefore
we can safely pass `nil` for those two parameters. 

##### Getting information from the MouseSystem
The `MouseSystem` updates the `mouseTracker.MouseComponent` every frame. Everything we need to know, is in there. 

{% highlight go %}
// Update is ran every frame, with `dt` being the time
// in seconds since the last frame
func (cb *CityBuildingSystem) Update(dt float32) {
	if engo.Input.Button("AddCity").JustPressed() {
		fmt.Println("The gamer pressed F1")

		city := City{BasicEntity: ecs.NewBasic()}

		city.SpaceComponent = common.SpaceComponent{
			Position: engo.Point{cb.mouseTracker.MouseX, cb.mouseTracker.MouseY},
			Width:    30,
			Height:   64,
		}

		texture, err := common.LoadedSprite("textures/city.png")
		if err != nil {
			panic("Unable to load texture: " + err.Error())
		}

		city.RenderComponent = common.RenderComponent{
			Drawable: texture,
			Scale:    engo.Point{X: 0.1, Y: 0.1},
		}

		for _, system := range world.Systems() {
			switch sys := system.(type) {
			case *common.RenderSystem:
				sys.Add(&city.BasicEntity, &city.RenderComponent, &city.SpaceComponent)
			}
		}
	}
}
{% endhighlight %}

But we still don't have a `world` reference within our `Update` function. We are given one inside our `New` function,
so let's save that reference. 

{% highlight go %}
type CityBuildingSystem struct {
	world *ecs.World

	mouseTracker MouseTracker
}

// New is the initialisation of the System
func (cb *CityBuildingSystem) New(w *ecs.World) {
	cb.world = w
	fmt.Println("CityBuildingSystem was added to the Scene")

	// ...
}
{% endhighlight %}

Now, we can reference it by saying `cb.world`, within our `Update` function. 

Now, if we were to run this whole project, and place a few cities using the **F1**-key, we would get something like
this:

<figure class="callout text-center">
<a href="/img/tutorials/02/cities.png" target="_blank">
<img alt="City texture" src="/img/tutorials/02/cities.png" style="height:400px">
<figcaption>The result of tutorial 2, being able to place multiple cities</strong></figcaption>
</a>
</figure>

<div class="button-group stacked">
<a class="button" href="/tutorials/03-camera-movement">Continue to Tutorial 3: <i>Camera Movement</i> &raquo;</a>
</div>
