---
title: "How we built an auto-scalable Minecraft server for 1000+ players using WorldQL's spatial database"
description: Raising Minecraft world capacity by spreading players across multiple synchronized server processes
date: 2021-09-02
tags:
    - minecraft
    - case study
layout: layouts/post.njk
author: Jackson Roberts
---

## Minecraft's multiplayer performance problems
Minecraft's server software is single-threaded, meaning it must process all events in the world sequentially on a single CPU core.
Even on the most powerful computers, a standard Minecraft server will struggle to keep up with over 200 players. Too many players attempting to load too much of the world will cause the server [tick rate](https://minecraft.fandom.com/wiki/Tick#Game_tick) to plummet to unplayable levels.
YouTuber SalC1 made a [video](https://www.youtube.com/watch?v=MBpN679o5Yk) talking about this issue which has garnered nearly a million views.

Back at the beginning of the 2020 quarantine I became interested in the idea of a supermassive Minecraft server, one with thousands of players unimpeded by lag.
This was not possible at the time due to the limitations of Minecraft's server software, so I decided to build a way to share player load across multiple server processes. I named this project "Mammoth".


My first attempt involved slicing the world into 1024 block-wide segments which were "owned" by different servers. Areas near the borders were synchronized and ridden entities such as horses or boats would be transferred across servers. [Here's a video on how it worked](https://youtu.be/Q1RXHS4N6wg?t=60). This technique is no longer used; the Minecraft world is no longer sliced up by area.

It was a neat proof-of-concept, but it had some pretty serious issues.
Players couldn't see each other across servers or interact. There was a jarring reconnect whenever crossing server borders.
If one server was knocked offline, certain regions of the world became completely inaccessible. It had no way to mitigate lots of players in one area, meaning large-scale PvP was impossible. The experience simply wasn't great.

To actually solve the problem, something more robust was needed. I set the following goals:
- Players must be able to see each other, even if on different server processes.
- Players must be able to engage in combat across servers.
- When a player places a block or updates a sign, it should be immediately visible to all other players.
- If one server is down, the entire world should still be accessible.
- If needed, servers can be added or removed at-will to adapt to the amount of players.

To accomplish this, the world state needed to be stored in a central database and served to Minecraft servers as they popped in and out of existence.
There also needed to be a message-passing backend that allowed player movement packets to be forwarded between servers for cross-server visibility.

## WorldQL is created
While early versions of Mammoth used redis, I had some new requirements that my message passing and data storage backend needed:
- Fast messaging based on proximity, so I could send the right updates to the right Minecraft servers (which in turn send them to player clients)
- An efficient way to store and retrieve permanent world changes
- Real-time object tracking

I couldn't find any existing product with these qualities. I found incomplete attempts to use SpatialOS for Minecraft scaling, and I considered using it for this project. However, their license turned me off.

To meet these requirements, I started work on WorldQL. It's a real-time, scriptable spatial database built for multiplayer games.
WorldQL can replace traditional game servers or be used to load balance existing ones.

If you're a game developer or this just sounds interesting to you, please be sure to join [our Discord server](https://discord.gg/tDZkXQPzEw).

The new version of Mammoth uses WorldQL to store all permanent world changes and pass real-time player information (such as location) between servers.
Minecraft game servers communicate with WorldQL using [ZeroMQ](https://zeromq.org/) TCP push/pull sockets.

## Mammoth's architecture
Mammoth has three parts:
1. Two or more Minecraft server hosts running Spigot-based server software
2. WorldQL server
3. BungeeCord proxy server (optional)

![WorldQL architecture diagram](/img/mammoth-arch.png)

With this setup, a player can connect to **any** of the Minecraft servers and receive the same world and player data. Optionally, a server admin can choose to put the Minecraft servers behind a proxy, so they all share a single external IP/port.

## Part 1: Synchronizing player positions

To broadcast player movement between servers, Mammoth uses WorldQL's location-based pub/sub messaging. This is a simple two-step process:
1. Minecraft servers continuously report their players' locations to the WorldQL server.
2. Servers receive update messages about players in locations they have loaded.

Here's a video demo showing two players viewing and punching each other, despite being on different servers!

<video controls>
    <source src="/img/minecraft-cross-server-pvp.mp4" type="video/mp4">
</video>

The two Minecraft servers exchange real-time movement and combat events through WorldQL.
For example, when *Left Player* moves in front of *Right Player*:
1. _Left Player_'s Minecraft server sends an event containing their new location to WorldQL.
2. Because _Left Player_ is near _Right Player_, WorldQL sends a message to Right Player's server.
3. _Right Player_'s server receives the message and generates client-bound packets to make _Left Player_ appear.

## Part 2: Synchronizing blocks and the world

Mammoth tracks the authoritative version of the Minecraft world using WorldQL Records, a data structure designed for permanent world alterations.
In Mammoth, no single Minecraft server is responsible for storing the world. All block changes from the base seed are centrally stored in WorldQL. These changes are indexed by chunk coordinate and time, so a Minecraft server can request only the updates it needs since it last synced a chunk.

Here's a video demonstrating real-time block synchronization between two servers. Complexities such as sign edits, compound blocks (like beds and doors) and nether portal creation all work properly.

<video controls>
    <source src="/img/minecraft-cross-server-blocks.mp4" type="video/mp4">
</video>

When a new Minecraft server is created, it "catches up" with the current version of the world. Prior to recording the video below, I built a cute desert home then **completely deleted my Minecraft server's world files**. It was able to quickly sync the world from WorldQL. Normally this happens automatically, but I triggered it using Mammoth's /refreshworld command so I can show you.

<video controls>
  <source src="/img/minecraft-world-sync-catchup.mp4" type="video/mp4">
</video>

This feature allows a Minecraft server to **dynamically auto-scale**; server instances can be created and destroyed to match demand.

Mammoth's world synchronization is incomplete for the latest 1.17.1 update. We're planning to introduce redstone, hostile mob, and weapon support ASAP.

## Performance gains

While still a work in progress, Mammoth offers considerable performance benefits over standard Minecraft servers. It's particularly good for handling very high player counts.

Here's a demonstration showcasing 1000 cross-server players, this simulation is functionally identical to real cross-server player load. The server TPS never dips below 20 (perfect) and I'm running the whole thing on my laptop.

These simulated players are created by a loopback process which:
1. Receives WorldQL player movement queries.
2. Modifies their location and name 1000 times and sends them back to the server.

This stress test results in the player seeing a wall of copycats:

<video controls>
  <source src="/img/mammoth-stress-test.mp4" type="video/mp4">
</video>

Mammoth pushes Minecraft server performance further than ever and will enable entirely new massively-multiplayer experiences. Keep in mind this demo **exists only to show off the efficiency of the message broker and packet code**, this is not as stressing as 1000 real players connecting. Stay tuned for a real-player demo.

## Coming soon: Program entire Minecraft mini-games inside WorldQL using JavaScript
Powered by the V8 JavaScript engine, WorldQL's scripting environment allows you
to develop Minecraft mini-games without compiling your own server plugin.
This means you don't have to restart or reload your server with every code change, allowing you to develop fast.

As an added bonus, every Minecraft mini-game you write will be scalable across multiple servers, just like our "vanilla" experience.

The process of developing Minecraft mini-games using WorldQL is very similar to using WorldQL to develop multiplayer for stand-alone titles. If you're interesting in
trying it out when it's ready, [be sure to join our Discord to get updates first](https://discord.gg/tDZkXQPzEw).

## Conclusions

Thanks for reading this article! Feel free to check out our [GitHub repository for the Mammoth Minecraft server plugin](https://github.com/WorldQL/mammoth) and [join WorldQL's Discord](https://discord.gg/tDZkXQPzEw)!


<h2 id="comments">Comments</h2>

[/r/gamedev](https://www.reddit.com/r/gamedev/comments/pgoa7q/how_we_built_an_autoscalable_minecraft_server_for/)

