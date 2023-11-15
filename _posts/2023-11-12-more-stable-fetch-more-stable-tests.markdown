---
layout: post
title: "Making tests more reliable - another idea: taping into fetch"
date:   2023-11-12 15:00:00 +0100
categories: javascript testing integrationtest playwright
---

Guess there is no contest that testing needs to happen. Which is easier said than done: unit tests don't cover the complexity of interplay between components - while acceptance tests are flaky. Still: the user needs to see sth so we need higher-level acceptance (and integration) tests. Not knowing when loading is complete is a major problem - and this proposes another (still rough) idea on how to deal with that. 

The core of the idea is to override `window.fetch` *only* during testing. As fetch returns a promise, we have to replace the return with our promise so we know when the result is there. Same goes for methods like `res.json()` - which again we have to wrap. Finally, let's jump to the end of the queue one more time (via `setTimeout`) - so that any promise handler likely fired before we assume all is finished. 

You can find the complete [implementation on github](https://github.com/sebastianrothbucher/web-small-stuff/blob/main/override-fetch/frontend/fetch-override.js). You can start checking from this part: 

```javascript
// a lot more detail
window.__hasNoOngoingFetch = function() {
    return __openFetchPromises === 0;
};
// a lot more detail
window.fetch = __instrumentPromise(window, window.fetch, 'fetch');
```

It's still alpha and *only for testing anyway* - but checking this in playwright yields a pretty great reasult already: 

```javascript
// initializing interaction
  await page.waitForFunction(() => (window as any).__hasNoOngoingFetch()); // so much better
// we can be sure we're done
```

You find the complete [example on github](https://github.com/sebastianrothbucher/web-small-stuff/blob/main/override-fetch/frontend/fetch-override.js) as well. 

Again: this is very much still **alpha** - but I wanted to share the idea regardless; let's see how this shakes out...

Hope you find it useful & let me know what you think!
