---
layout: post
title: "Create a StarCraft II Bot - Part 1: Setup on MacOS"
date:   2017-05-01
categories: cpp bot
---

Recently, Blizzard published the [StarCraft
II](https://news.blizzard.com/en-us/starcraft2/20944009/the-starcraft-ii-api-has-arrived)
API which allows developers to create bots to play the game.

I really like StarCraft series, so I want to look into it and create my own
bots. If it's possible, I can also learn how to use Machine Learning for bot.

This series of blog posts will be my notes for the process of learning. Today
I build the [sc2-client-api](https://github.com/Blizzard/s2client-api) and run a
sample bot.

The document of the repo is very clear. You will do these steps to build the
project:

1. Make sure a recursive clone of the project is done to download all
   submodules.

```shell
git clone --recursive https://github.com/Blizzard/s2client-api
```

2. Enter the working directory and create a build directory for CMake artifacts.

```shell
cd s2client-api
mkdir build
cd build
```

3. Create XCode project file

```shell
cmake ../ -G Xcode
```

4. Open the project and build using XCode

```shell
open s2client-api.xcodeproj/

# Then press Cmd+B to build
```

5. Run a sample bot.

```shell
cd bin/bot_mp
```

Now you can see two instances of StarCraft playing with each other.
