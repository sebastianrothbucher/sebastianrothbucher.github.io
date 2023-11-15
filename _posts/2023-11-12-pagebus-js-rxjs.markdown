---
layout: post
title: "Why a pagebus makes a lot of sense"
date:   2023-11-12 12:00:00 +0100
categories: javascript rxjs pagebus microfrontend
---

Took me a bit to write this up - but here we are: event driven is just business as usual in backend. And it makes a ton of sense in frontend, too. Especially, when the UI consists of several components, pot. even from several builds. Sounds a lot like Kafka and several stream processors - a pattern that is successful for a reason. So time to do the same in the frontend. 

First of all: let's define the problem we're trying to solve: keeping several parts of the UI in sync. Sometimes (like with form-based apps), that's just not a thing. Other times (like in e-commerce), that is very much relevant - like updating the items in a shopping cart, which is done by a "buy" button far away from the cart symbol far away from a "related" box. Some might go hardcore and really have several builds with several scripts - pot. even in several frameworks and bound together by SSI - one realization of the idea of "micro frontends". But even if we stay grounded and just have one frontend (and one build): there's still several components to be kept in sync. 

The obvious answer is using a state framework à la Redux, Vuex, NgRx. And when you start of with that (and it fits the use case): great! You're all set.

What I want to do here is show an easy way to do it in environments where revamping state management is *not* an option. 

## Pagebus - what's the idea?

In short, it's

- a globally reachable object
- that contains one or more variables (e.g. shopping cart contents)
- that can, at any point, tell the current status of a variable (i.e. what's in the cart)
- that allows any part of the app to register for updates to a variable

Of course, you can just implement this yourself, no issue. I also did this a couple times. When anyhow using RxJS (a given when you're in the Angular world), you even get it for free - via *BehaviorSubject*

## Implementing with BehaviorSubject

I've put together this [CodePen](https://codepen.io/sebredhh/pen/OJdbQMq) with a full example. 

You can instantiate a BehaviorSubject from RxJS with an initial value: 

```javascript
var bs = new rxjs.BehaviorSubject(0);
```

Then, you can get the value via `bs.value` and can update it via `bs.next(newValue)`. Any part of the code can subsribe an Observer via `bs.subscribe((nextValue => doSth...)`. And that's it, really - no matter how many places you have where you need an up-to-date value: you got it.

Hope you find it useful & let me know what you think!
