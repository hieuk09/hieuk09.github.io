---
layout: post
title: "Thing I learn when writing bot for a spaceship shooting game"
date:   2021-09-19
categories: programming
---

My friend at [WeBuild](we-build-vn.slack.com/) introduces me to a bot writing
game, which we write instructions for a spaceship and let it chase, dodge and
shoot at each other. It's pretty fun and you can find the source code of the
game in [this repo](github.com/unrealhoang/tokyo-rs).

I learned a lot of things during the implementation:

- How to implement path collision:

  - At first I intend to find 2 vectors that represent velocity of each object,
  then find `ax + by = c` representation of them. From there, find the
  intersection point and see whether that intersection point is on the road for
  the next few seconds.
  - This actually isn't very good in reality, as representation of a path is not
  a line but a rectangle, and finding intersection and collision moment for 2
  rectangles is not trivial.
  - In the end, I opt to a less performance approach, but works better: loop
  through each 10ms, move the objects, then find whether they collide.

- How to find angle to X-axis from one point to another (2)

- Always add test if you have doubt about the implementation:
  - When implement point (2) above, I struggle a lot because I couldn't figure
  out how the system calculate the angle. Writing all cases and tests them first
  help me get a better feedback loop and verify my understanding easier.

- How to implement a simple generic algorithm to train my spaceship:
  - there are still quite a lot of bugs that make this implementation unstable
