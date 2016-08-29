---
layout: tutorial
title: Hello World
permalink: /tutorials/01-hello-world
number: 1
---

In this tutorial, we will create our first window in `engo`.

> #### Recap
> Remember what we did in [the last tutorial](/tutorials/00-foreword)?
> We created the directory `$GOPATH/github.com/EngoEngine/TrafficManager` - and will be working from this directory
> from now on. All `go`-commands will be executed from this working directory. You will probably be
> working from some similar directory.

> #### Final Code
> The final code **for tutorial 1** is available
> [here at GitHub](https://github.com/EngoEngine/TrafficManager/tree/01-hello-world).

### Our first file
As with all Go-projects, we usually have one main file, and a few other packages we're using. We shall create that
single main file in this tutorial. We shall call it `traffic.go`, but you may give it any other name you want.

> #### Ensure you can build
> Before continuing, you might want to make sure you can compile and run the most basic Go program there is: hello
> world:
>
> {% highlight go %}
package main

import "fmt"

func main() {
    fmt.Println("Hello!")
}
{% endhighlight %}


There is one function in `engo` that will allow you to start your game / window: `engo.Run(RunOptions, Scene)`. As you
see, there are two parameters to this function: one of type `RunOptions`, where you can set things like `Fullscreen`,
`FPSLimit`, `VSync`, and much more, and the second one is your default `Scene`.

> #### Scenes
> Scenes are the barebone of `engo`: they contain pretty much all other things within the game. You should have multiple
> scenes and switch between them: i.e. one for the main menu, another for the loading screen, and yet another for the
> in-game experience. Specifically, a `Scene` contains **one** `World`, a collection of multiple `System`s and a
> magnitude of `Entity`s - we will be creating those in the next tutorial.

The easiest way to output "Hello World" within `engo`, is by having this `traffic.go` file:

{% highlight go %}
package main

import (
	"engo.io/engo"
	"engo.io/ecs"
)

type myScene struct {}

// Type uniquely defines your game type
func (*myScene) Type() string { return "myGame" }

// Preload is called before loading any assets from the disk,
// to allow you to register / queue them
func (*myScene) Preload() {}

// Setup is called before the main loop starts. It allows you
// to add entities and systems to your Scene.
func (*myScene) Setup(*ecs.World) {}

func main() {
	opts := engo.RunOptions{
		Title: "Hello World",
		Width:  400,
		Height: 400,
	}
	engo.Run(opts, &myScene{})
}
{% endhighlight %}

Currently, this `Scene` does nothing. But it will compile and run. It'll open up a window, with the title "Hello World".

### Preparing our first Texture
We want to render something onto the screen at this point. For this, we want to create a new directory within our
`TrafficManager` directory, called `assets` (you can also name it `res`, or any name you like). In here, we shall be
putting all resource-files, such as textures, audio files, etc. Let's create a subfolder `textures`. The full path would
be: `$GOPATH/github.com/EngoEngine/TrafficManager/assets/textures`.

The texture we would like to draw onto the screen, is going to be an illustration of the "Location"-icon we all know.

<figure class="callout text-center">
<a href="/img/tutorials/01/city.png" target="_blank">
<img alt="City texture" src="/img/tutorials/01/city.png" style="height:200px">
<figcaption>The *City* icon, <strong>city.png</strong></figcaption>
</a>
</figure>

We shall save this file within the `assets/textures` directory.

### Loading the texture into the game
We have to load the actual file from the disk. Fortunately, `engo` has helper-functions available within
`engo.Files`. It has an `Load` function, which allows us to specify the location of one or more files, and they
automatically get loaded once the `Preload()` function returns.

Applying this, we get:

{% highlight go %}
// Preload is called before loading any assets from the disk,
// to allow you to register / queue them
func (*myScene) Preload() {
	engo.Files.Load("textures/city.png")
}
{% endhighlight %}

### Rendering the *City* icon
Rendering something consists of three things:

* The System (`RenderSystem`, specifically);
* The Entity (you can have a lot of these);
* Two Components (`RenderComponent` and `SpaceComponent`).

#### ECS
The three different types mentioned here, form the ECS (Entity Component System). The idea is quite simple: you have
a lot of entities (objects). These can have different values and variables - these form the different components each
entity can have. Entities which have a `SpaceComponent` for example, have some information as to their location
(in the game*space*). These entities and components do nothing at all. They are just glorified data containers. A
`Component` can have variables (and values for those variables), and an `Entity` is just a glorified `[]Component`.

**Doing** something, is something only systems are allowed to. They can change/add/remove entities, as well as change/add/remove any
components on those entities. They can change values like the location-values within the `SpaceComponent`. You can have
multiple systems, and each of them has its own task. Each frame, these systems are called so they can do things. They
will usually do things with entities. The `RenderSystem` is one of the system we have made, which already has a lot of
OpenGL-calls builtin (so you don't have to worry about those just yet). You will learn about creating your own
`System` in [tutorial 2](/tutorials/02-first-system).

[This YouTube video](https://www.youtube.com/watch?v=BvEK9-CU5Og) gives a (short) visual explanation of the
ECS-paradigm, and why it's so much better than what he used to use. It doesn't talk about systems that much, but more
about the modularity of the entities.

#### Adding the System
In order to add the `RenderSystem` to our `engo`-game, we want to add it within the `Setup` function of our `Scene`.
`RenderSystem` is located in the engo.io/engo/common package, along with other frequently used systems.

{% highlight go %}
import "engo.io/engo/common"

// Setup is called before the main loop starts. It allows you
// to add entities and systems to your Scene.
func (*myScene) Setup(world *ecs.World) {
	world.AddSystem(&common.RenderSystem{})
}
{% endhighlight %}

More speficially, an instance of the `RenderSystem` is added to the `World` of this specific `Scene`.

> ##### Do you recall?
> Each `Scene` has only **one** `World`. And this `World` is a collection of systems and entities.

#### Adding the Entity
After we've added the `RenderSystem` to the `World`, we are now ready to create our `Entity` and add it as well. We
shal begin by defining our `City` struct:

{% highlight go %}
type City struct {
	ecs.BasicEntity
	common.RenderComponent
	common.SpaceComponent
}
{% endhighlight %}

As you will see, this `City` struct (the 'Entity'), consists of one standard thing (`ecs.BasicEntity`, which provides
a unique identifier), and two `Component`s: they are the only way you can pass information around different systems,
like telling the `RenderSystem` what to render. The first (the `RenderComponent`) holds information about what to render
(i.e. which texture), and the second (the `SpaceComponent`) holds information about *where* it should be rendered. 

In order to correctly instantiate, we need to ensure that `ecs.BasicEntity` is set to a new, unique identifier. We
can do this by calling `ecs.NewBasic()`. 

{% highlight go %}
city := City{BasicEntity: ecs.NewBasic()}
{% endhighlight %}

> ##### SpaceComponent
> This will locate the `Entity` at 10 units lower and to the right, from the origin. It will be 303 units wide, and
> 641 units high.
>
> {% highlight go %}
city.SpaceComponent = common.SpaceComponent{
    Position: engo.Point{10, 10},
    Width:    303,
    Height:   641,
}
{% endhighlight %}

The `common.RenderComponent` is a bit tricky though, as it requires us to define a `Texture` to draw, and provide a `Scale` 
value (usually just `engo.Point{1, 1}`). The helper-function `common.LoadedSprite(url string)` will provide a
reference to the sprite that was preloaded earlier during the `Preload()` function.

> ##### RenderComponent
>
> {% highlight go %}
texture, err := common.PreloadedSpriteSingle("textures/city.png")
if err != nil {
    log.Println("Unable to load texture: " + err.Error())
}

city.RenderComponent = common.RenderComponent{
    Drawable: texture,
    Scale:    engo.Point{1, 1},
}
{% endhighlight %}

Now we've completed the `Entity`, we should not forget to add it to the appropriate systems:

> ##### Adding the `City` to the `World`
>
> {% highlight go %}
for _, system := range world.Systems() {
    switch sys := system.(type) {
    case *common.RenderSystem:
        sys.Add(&city.BasicEntity, &city.RenderComponent, &city.SpaceComponent)
    }
}
{% endhighlight %}

What are we doing? We're looping over all known Systems, to see if one is of type `common.RenderSystem`. If that is the 
case, we're using the RenderSystem-specific `Add` method to add our `City` to that system. This system requires three
parametesr: reference to `BasicEntity`, reference to `RenderComponent` and reference to `SpaceComponent`. We have all
of those, so we can easily do this. If we were to add our `City` to more systems, we could simply add additional
`case` clauses here, and call the appropriate `Add` functions. 

If we were to run our game (`go run traffic.go`), the result should look something like this:

<figure class="callout text-center">
<a href="/img/tutorials/01/black.png" target="_blank">
<img alt="City texture" src="/img/tutorials/01/black.png" style="height:200px">
<figcaption>Our game so far, with the (default) black background</strong></figcaption>
</a>
</figure>

### Making improvements
As you might have noticed, a black background doesn't fit in nicely with our non-transparent white background at the
*City* icon. We might change the white to transparent, but that won't improve that much. Let's change the background
to white instead:

> ##### Changing the background color
>
> You can change the background color with this line of code:
>
> {% highlight go %}
common.SetBackground(color.White)
{% endhighlight %}
>
> Note that we're using `color`, so be sure to import `image/color`. It's in the standard library. This line of code
> should go somewhere within the `Setup` function, and preferably at the top. However, it's not mandatory to put it
> *there*, and you can dynamically change the background color as you please.

It our game looks like this:

<figure class="callout text-center">
<a href="/img/tutorials/01/white.png" target="_blank">
<img alt="City texture" src="/img/tutorials/01/white.png" style="height:200px">
<figcaption>After changing the background color</strong></figcaption>
</a>
</figure>

<div class="button-group stacked">
<a class="button" href="/tutorials/02-first-system">Continue to Tutorial 2: <i>Our First System</i> &raquo;</a>
</div>
