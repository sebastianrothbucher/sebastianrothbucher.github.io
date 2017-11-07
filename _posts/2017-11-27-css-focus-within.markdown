---
layout: post
title:  "Why :focus-within is huge"
date:   2017-11-27 20:20:00 +0200
categories: visualization CSS styling focus
---

Writing a post on any CSS selector or property (or even any new one coming up) would produce a long (and boring) blog. But *:[focus-within]* is different - and it's something I have been waiting for *forever*. To me, it's more than another pseudo-class, it's a game-changer in many ways. And it's in the stable versions of both Chrome and Firefox (and others). See the MDN page on :[focus-within] for a full (and esp. up2date) info on browser support.

What *:focus-within* does is quite simple, actually: **it allows styling the parent(s) of the element that has the focus**. Opposed to *:focus* (which references the element having the focus itself), it can be the direct parent, or the parent's parent or the parent of the parent's parent or... (you get it). 

Read my whole [article on my team's blog](https://blog.akquinet.de/2017/11/27/why-focus-within-is-huge/)...

[focus-within]: https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-within
