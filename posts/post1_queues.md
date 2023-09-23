# So what are you really doing in queue?

## Introduction

Regardless of which multiplayer game you play, waiting in queue for a match is something you are likely accustomed to. After doing some quick math, I've personally spent close to 300 hours waiting (most of which is through *League of Legends* unfortunately). 

![Likely the most popular queue pop in the world.](https://images2.minutemediacdn.com/image/upload/c_fill,w_2160,ar_16:9,f_auto,q_auto,g_auto/shape%2Fcover%2Fsport%2Fdataimagepngbase64iVBORw0KGgoAAAANSUhEUgAABQAAAALQ-c51a6d3120e7c087a8739a15b1ed0bcd.jpg)

A recent shower thought prompted me to think of how these queues are implemented behind the scenes from a distributed systems point of view, especially when considering sequences like player matching, player match acceptance, and the various failure states that may occur.

Moreover, there are often several logistical complexities that exist based on the game context. Players can have specific criteria attached to them that makes certain matchings ineligible. Continuing with the League of Legends example, players have 2 or more preferred game roles; the system ideally wants to form a 10 player matching in which as many players as possible receive their higher preference roles. This "perfect world" matching has to be balanced against providing players with reasonable queue times, especially during off-peak hours (those 7am weekday games are a bit of a gamble).

## The Scope

This post leads readers through designing and building a versatile queueing system that can support queueing models of multiple games with relative ease. Aside from a discussion of competing architectural choices, there will be a few code samples to showcase some more technical features of what we build at a high-level.

The tutorial requires some knowledge of basic computer science concepts (i.e., TCP vs UDP), but external resources will be linked whenever possible to aid readers. When code is required, it will usually be implemented using Rust, unless the specific use case dictates otherwise. 

Hope you have fun reading and get something useful from the content! Feel free to comment on any of the posts with feedback you may have; I'm new to technical writing so any feedback helps!

## Foundations




