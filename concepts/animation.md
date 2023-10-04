---
layout: tutorial
title: Animation
---
# What is Animation?

Consider the following image:
![](http://www.angryanimator.com/tut/pic/002_walkcycle/wlk01.gif)

Here we can see a number of frames, when we move those frames in one place at a certain speed we animate the character.

The `engo` game engine supports raster and vector graphics animation.

# Spritesheets

It is quite common in computer animation to see so called "Spritesheets". A spritesheet is an image that consists of one or many frames that become part of one or multiple animations.

Spritesheet can be divided into two categories:
- symmetrical spritesheets
- unsymmetrical spritesheets

Engo supports both symmetrical spritesheets and unsymmetrical ones but handling of them in the code is slightly different.

# Symmetrical spritesheet:

```
engo.Spritesheet
```

For symmetrical textures in engo game engine, there is a class that allows us to pass texture and automatically compute sprites (sub cells) out of an image file. A sprite is represented by a Region class.

Useful methods from a Spritesheet:

```
NewSpritesheetFromTexture(texture *Texture, cellWidth, cellHeight int) *Spritesheet

NewSpritesheetFromFile(textureName string, cellWidth, cellHeight int) *Spritesheet

Drawable(index int) Drawable

CellCount() int

Drawables() []Drawable
```

# How to animate from a symmetrical spritesheet?

First, we need to create a spritesheet. Let's load a texture. We can define a Preload() method in our GameWorld:

```go
func (*DefaultScene) Preload() {
  engo.Files.Load("assets/hero.png")
}
```

now we need to take a look at our spritesheet and check which frames correspond to our animation. Frames can be accessed using index and:

```go
   Drawable(index int) Drawable
```

once we have that jotted down we can in setup method instantiate Spritesheet and AnimationAction

```go
func (*myScene) Setup(u engo.Updater) {
	world, _ := u.(*ecs.World)
  game.AddSystem(&engo.RenderSystem{})
  game.AddSystem(&engo.AnimationSystem{})

  spriteSheet := engo.NewSpritesheetFromFile("hero.png", 150, 150)

  animationAction := &engo.AnimationAction{Name: "run", Frames: []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10}}

  hero := scene.CreateEntity(&engo.Point{0, 0}, spriteSheet, animationAction)

  // Add our hero to the appropriate systems
  	for _, system := range w.Systems() {
  		switch sys := system.(type) {
  		case *engo.RenderSystem:
  			sys.Add(&hero.BasicEntity, &hero.RenderComponent, &hero.SpaceComponent)
  		case *engo.AnimationSystem:
  			sys.Add(&hero.BasicEntity, &hero.AnimationComponent, &hero.RenderComponent)
  		case *ControlSystem:
  			sys.Add(&hero.BasicEntity, &hero.AnimationComponent)
  		}
  	}
}
```
