---
layout: post
title: "Build game with Godot Engine - Part 1"
date:   2018-02-21
categories: game
---

[Godot](https://godotengine.org/) is a free and opensource game engine which
provides powerful support for 2D and 3D games. I recently start working with
this framework to create my own game. This series is the note of the learning I
made during building by game. I currently use Godot 3.0.

## Build the most basic 2D application

Godot application is built as a collection of scenes. To build and run a game,
we must create at least one scene. We can go to `Scene > New Scene` to create a
scene. A scene can be a resource or an entire level of the game. Here, the new
scene we create will be our game main scene.

A scene is built using a collection of Nodes. We can click on `+` button on
Scene panel to add new Node:

![create_node]({{ site.url }}/assets/create_node.png)

For root node, you can choose `Node` option in the creation options.

Now you can play your game with `Cmd + B`, or the `|>` button on the shortcut
menu:

![play_game]({{ site.url }}/assets/play_game.png)

Now the game is built successfully and appear. Even though we cannot play it
yet, we've succeeded in creating our first game with Godot!
