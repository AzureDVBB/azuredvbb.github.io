---
title: "YUMS - A retrospective"
date: 2023-01-23T16:53:28+01:00
draft: false
---

This is a blog post for the archival of my [YUMS](https://github.com/AzureDVBB/YUMS) project.

We had a good run, a month of development that taught me much. But in the end. It all ends the same. Just another bowl of "modular" sphagetti.

### What was the plan?

A simple, barebones implementation that is extendable and scalable. It also should use as few dependancies as possible.

There was already an immensely more mature and batteries included framework called [Evennia](https://www.evennia.com/). Though I couldn't make heads or tails of it, plus it used quite a few dependencies.

### What went wrong?

In short, a lack of architecting and design.

The authentication had to use multiprocessing, making things monolithic. The only real scaling that could happen would be with cloud-based load balancers. Feature creep was setting in etc...

Many things were an afterthought, like implementing a way for re-loading python module definitions during runtime just to deal with updates to a running server.

Documentation was all over the place with readme files in folders, which isn't too bad inherently, but heck if anyone would be able to search for a specific thing. The addition of features and organisation of knowledge was... non-existent really.

It was turning into a mess really.

### so what now?

The project has been archived and will likely stay so for an indeterminate amount of time.

Python is a language I am quite familiar with, but it is evidently clear that it is not suited for such a back end, due to it's loose type system.

Later on I hope to resurrect this project as my foray into the [Rust](https://www.rust-lang.org/) language, as it is much more strict and memory safe.