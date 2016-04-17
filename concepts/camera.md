---
layout: tutorial
title: Camera
---
This tutorial will explain a bit more on how the `Camera` works, and more importantly: how you should use it. 

## Coordinates
Everything you do and see within `engo`, is related to coordinates. Please imagine the following line of code:
```go
awesomeSpace := engo.SpaceComponent{engo.Point{25, 35}, 90, 100}
```
This effectively means: 

From the center of the game `(0, 0)`,
* go `25` units to the right and
* go `35`units to the bottom.

To create something that
* has a width of `90` units and
* has a height of `100` units.

If we don't want to go `25` units to the right, but let's say, `400` to the left, the code simply becomes:

```go
greatSpace := engo.SpaceComponent{engo.Point{-400, 35}, 90, 100}
```

## The Camera
Most things within `GLFW` / `OpenGL`, are being rendered by the GPU. This is a good thing, because the GPU is fast at calculating these kind of things. The downside is, that there's usually just some mathematics involved in what the GPU gets to do. 

The thing you'd have to remember here, is that the `Camera`, is doing some tricky mathematics to create the 'illusion' of *having a camera on top of (or in front of) the world*. So changing values for the `Camera`, will change the appearance of the objects rendered. 

## Usage
### Moving to a location
There are several functions available for the `Camera`:
* `engo.Cam.MoveToX(some_x_coordinate)`,
* `engo.Cam.MoveToY(some_y_coordinate)`.

If we wanted to zoom in on the `awesomeSpace` we just created in the example above, for the X-coordinate, we would have to do this:

```go
object_location := 25
object_width := 90

engo.Cam.MoveToX(object_location + object_width / 2)
```
We want to move the camera to the location of the object first (`25`), and then add half (divide `2)` of the width (`90`) (so we're halfway the object) to get where we want. 

For the Y-coordinate, we could do the same:

```go
engo.Cam.MoveToY(35 + 100 / 2)
```

### Moving just a bit
What if we don't know exactly where we want to go, but know only what direction we want to take? We could use these functions for that:
* `engo.Cam.MoveX(some_delta_x_value)`,
* `engo.Cam.MoveY(some_delta_y_value)`.

Let's say our `Camera` is positioned at `(50, 100)`. Now we call these functions:
```go
engo.Cam.MoveX(4)
engo.Cam.MoveY(-6)
```
Now the `Camera` is positioned at `(54, 94)`, because we moved `4` units to the right, and `6` units up. 

### Zooming
You may have noticed that we were talking about *units*, and not about pixels, centimeters, or any other unit of measure. This is because the length of these things changes according to our zooming level. 

When you're 5 meters away from a 10 meter tall tree, the tree looks a lot bigger than when you are 500 meters away from it. The same 10 meter tall tree that's right next to it, will probably look the same size compared to the other tree, but -- like the first tree -- will be affected by the distance you're from the tree. 

The same applies to `engi`. 

With this in mind, we can also manipulate the Z-axis:
* `engo.Cam.ZoomTo(zome_z_value)`,
* `engo.Cam.Zoom(zome_z_delta_value)`.

Like with the X and Y coordinates, the `ZoomTo` and `Zoom` function behave the same: 
* `Zoom` allows you to zoom in/out just a (small) change,
* `ZoomTo` allows you to zoom in/out to a specific zoom-level.

## Helper-functions
### Knowledge
Sometimes you want to know the current `Camera` values, for this we have the following functions:
* `engo.Cam.X()`
* `engo.Cam.Y()`
* `engo.Cam.Z()`

### Easy usage
Now you might think: that's a lot of effort to move the screen around. And you are correct; it's quite some effort. That's why we created these helper functions:
* `engo.NewKeyboardScroller(scrollSpeed, keyUp, keyRight, keyDown, keyLeft)`
   which allows you to move the screen around by pressing the given keys. This can be used like this:
```go
game.AddSystem(engo.NewKeyboardScroller(0.125, engo.W, engo.D, engo.S, engo.A))
// You can now press the W to move up, D to move right, S to move down, and A to move to the left. 
```
* `engo.NewEdgeScroller(scrollSpeed, edgeMargin)`
    which allows you to move the screen around by moving the cursor to the edges of the window. This can be used like this:
```go
game.AddSystem(engo.NewEdgeScroller(800, 20)) 
// You can now keep your mouse within the outermost 20px of the window
```
* Zooming by using the mouse wheel can be done by adding this function to your `Game` struct:
```go
func (game *Game) Scroll(amount float32) {
    engo.Cam.Zoom(-1 * amount * 0.125)
}
```
