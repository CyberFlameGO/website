---
title: "How we built an auto-scalable Minecraft server for 1000+ players using WorldQL's spatial database"
description: Raising Minecraft world capacity by spreading players across multiple synchronized server processes
date: 2021-08-29
tags:
    - minecraft
    - case study
layout: layouts/post.njk
author: Jackson Roberts
---


## Minecraft's multiplayer performance problems
Minecraft's server software is single-threaded, meaning it must process all events in the world sequentially on a single CPU core.
Even on the most powerful computers a standard Minecraft server will struggle to keep up with over 200 players. Too many players attempting to load too much world will this cause the server [tick rate](https://minecraft.fandom.com/wiki/Tick#Game_tick) to plummet to unplayable levels.
YouTuber SalC1 made a [video](https://www.youtube.com/watch?v=MBpN679o5Yk) talking about this issue which has garnered nearly a million views.

Back at the beginning of the 2020 quarantine I became interested in the idea of a supermassive Minecraft server, one with thousands of players unimpeded by lag.
This was not possible at the time due to the limitations of Minecraft's server software, so I decided to build a way to share player load across multiple server processes. I named this project "Mammoth".


My first attempt involved slicing the world into 1024 block-wide segments which were "owned" by different servers. Areas near the borders were synchronized and ridden entities such as horses or boats would be transferred across servers. [Here's a video on how it worked](https://youtu.be/Q1RXHS4N6wg?t=60).

It was a neat proof-of-concept, but it had some pretty serious issues.
Players couldn't see each other across servers or interact. There was a jarring reconnect whenever crossing server borders.
If one server was knocked offline, certain regions of the world became completely inaccessible. It had no way to mitigate lots of players in one area, meaning large-scale PvP was impossible. The experience simply wasn't great.

To actually solve the problem, something more robust was needed. I set the following goals:
1. Players must be able to see each other, even if on different server processes.
2. Players must be able to engage in combat across servers.
3. When a player places a block or updates a sign, it should be immediately visible to all other players.
4. If one server is down, the entire world should still be accessible.
5. If needed, servers can be added or removed at-will to adapt to the amount of players.

## WorldQL is born
While early versions of Mammoth used redis as a backend, I had some new requirements if I needed to meet my goals.
- I needed fast message-passing based on proximity, so I could send the right updates to the right Minecraft servers (and eventually, clients).
-

## Part 1: Synchronizing player positions

This uses WorldQL subscriptions and messages. These are familiar if you've used something like redis.

Unlike redis, WorldQL is designed for games, so it supports unreliable transports like UDP for latency-sensitive applications (Minecraft uses TCP, but this is very uncommon among real-time games).

## Part 2: Synchronizing blocks and the world

This uses WorldQL records, a data structure intended for permanent world alterations.

## Part 3: Bragging about performance

## Coming soon: Program entire Minecraft mini-games inside WorldQL using JavaScript
Powered by the V8 JavaScript engine, WorldQL's scripting environment allows you
to develop Minecraft mini-games without compiling your own server plugin.
This means you don't have to restart or reload your server with every code change, allowing you to develop fast.

As an added bonus, every Minecraft mini-game you write will be scalable across multiple servers, just like our "vanilla" experience.

The experience of developing Minecraft mini-games using WorldQL is very similar to using WorldQL to develop multiplayer for stand-alone titles. If you're interesting in
trying it out when it's ready, be sure to join our Discord to get updates first.

