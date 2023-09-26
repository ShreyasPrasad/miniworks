# So what are you really doing in queue?

## Introduction

Regardless of which multiplayer game you play, waiting in queue for a match is something you are likely accustomed to. After doing some quick math, I've personally spent close to 300 hours waiting (most of which is through *League of Legends* unfortunately). 

![Likely the most popular queue pop in the world.](https://images2.minutemediacdn.com/image/upload/c_fill,w_2160,ar_16:9,f_auto,q_auto,g_auto/shape%2Fcover%2Fsport%2Fdataimagepngbase64iVBORw0KGgoAAAANSUhEUgAABQAAAALQ-c51a6d3120e7c087a8739a15b1ed0bcd.jpg)

A recent shower thought prompted me to think of how these queues are implemented behind the scenes from a distributed systems point of view, especially when considering sequences like player matching, player match acceptance, and the various failure states that may occur.

Moreover, there are often several logistical complexities that exist based on the game context. Players can have specific criteria attached to them that makes certain matchings ineligible. Continuing with the League of Legends example, players have 2 or more preferred game roles; the system ideally wants to form a 10 player matching in which as many players as possible receive their higher preference roles. This "perfect world" matching has to be balanced against providing players with reasonable queue times, especially during off-peak hours (those 7am weekday games are a bit of a gamble).

## The Scope

This post leads readers through designing and building a versatile queueing system that can support queueing models of multiple games with relative ease. Aside from a discussion of competing architectural choices, there will be a few code samples to showcase some more technical features of what we build at a high-level.

The tutorial requires some knowledge of basic computer science concepts (i.e., TCP vs UDP), but external resources will be linked whenever possible to aid readers. When code is required, it will usually be implemented using Rust, unless the specific use case dictates otherwise.

Hope you have fun reading and get something useful from the content! Feel free to comment on any of the posts with feedback you may have!

## Requirements

### Support a scale of tens of millions of active players

Before attempting to design such a system, it's important to acknowledge the autoscaling nature of these systems in real life. Queueing systems for popular games can have millions of active searching players during peak hours and far closer to tens of thousands of active players during off-peak hours. Horizontal scalability will likely be our best friend for both of these situations.

### Be extensible

The system needs to be able to powerfully express complicated matching algorithms using a single interface. The internal representation of the queue should be a hidden implementation detail. This interface should also allow developers to specify a tradeoff between queue matching timeliness and player matching preference.

### Support 2-phase matchings 

In this context, a 2-phase matching requires the initial request by a player to enter a queue and their subseqeuent acceptance following the system's proposed matching. This is analagous to the concept of a [2-phase commit](https://en.wikipedia.org/wiki/Two-phase_commit_protocol) in distributed systems when coordinating actions between multiple servers.

## The Foundation

Let's get to the code! First, we need a way to accept incoming player connections and simulate a distributed environment.

### Choosing a webserver

A webserver will help us in achieving a separation of concerns. Our webserver can be responsible for:

- Load balancing between a number of downstream servers based on load.
- Eventually perform [`TLS` termination](https://en.wikipedia.org/wiki/TLS_termination_proxy) so our downstream servers don't have to.
- Automatically manage connection re-use for persistent `HTTP` [keep-alive](https://en.wikipedia.org/wiki/HTTP_persistent_connection) connections.

#### `NGINX`: Old but gold?

The classic, most widely used webserver is NGINX, which allows us to do all this and more.

Though, `NGINX` has its fair share of problems including poor `HTTP` connection re-use and limited support for request failover. There were brought to light after [Cloudflare went public with Pingora](https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/), its initiative to rewrite `NGINX` in Rust.

These problems are not relevant to our use case, and using `NGINX` lets us keep things simple.






