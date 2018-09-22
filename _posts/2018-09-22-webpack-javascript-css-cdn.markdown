---
layout: post
title: "Using a CDN with webpack"
date:   2018-09-22 16:00:00 +0200
categories: web html css javascript webpack cdn
---

[Webpack](https://webpack.js.org) is a great tool for getting an application out of the door - and optimize the way code gets packaged and transported to users' browsers (like minimum size, combining many small downloads into one by inlining, eliminating dead code by [Tree Shaking](https://webpack.js.org/guides/tree-shaking/), etc.). However, one can go beyond optimizing the own delivery using *Content Delivery Networks (CDN)* like [CDNJS](https://cdnjs.com/) to download commonly used libraries (like React or jQuery) from there instead of packing them into the own delivery. This has a couple of advantages: 

* several websites will likely request the same library (esp. popular ones), i.e. the second site can just grab it from the cache without any load time
* CDN providers will have servers around the globe, possibly near a user's physical location - reducing the download times (a tiny little) more b/c of a shorter wire (literally)

That said, there's some drawbacks: 

* users need a public internet connection (at least initially)
* eliminating unused parts of a library (the Tree Shaking thing) is out when downloading the whole library unmodified from a public location directly onto users' machines
* the shorter new releases appear, the smaller the chance someone else is using the exact same release of a library (and that it's in the cache ever)

Given one wants to use a CDN at least for common libraries (it's perfectly possible to still package more exotic ones into the own bundle), it needs some setup for webpack. Furtunately, this is quite straightforward - and possible not only for JavaScript resources but also for CSS or fonts (at least with a little hack). 

## Externals

The main element to achieve using CDN is [Externals](https://webpack.js.org/configuration/externals/) which allows any named import (or require) to resolve against an object in the global namespace instead of *node_modules*. The assumption is that importing a library via CDN will have placed this object there before the code runs. So, in order to get react from CDN, it takes these steps: 

First, placing a reference to react from CDN into the head or body of the template (latter only when webpack also inserts at the end of body). Can do this well right away by adding [CORS](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes) and [SRI](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)

```html
<script language="javascript" type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/react/16.4.2/umd/react.production.min.js" crossorigin="anonymous" integrity="sha256-2EQx5J1ux3sjgPLtDevlo449XNXfvEplcRYWIF6ui8w="></script>
```

To compute the checksum, there's the quick and dirty way of putting in anything and check DevTools for the right one - or a clean way via a unix-like shell (thanks to superuser for help on [converting hex to base64](https://superuser.com/questions/158142/how-can-i-convert-from-hex-to-base64)) 

```
curl 'https://cdnjs.cloudflare.com/ajax/libs/react/16.4.2/umd/react.production.min.js' | shasum -a 256 | xxd -r -p | base64
```

All of this assumes we have a template HTML file like *indexcdn.html* specified in the webpack config, e.g. have sth like the following

```javascript
module.exports = {
    plugins: [
        new HtmlWebpackPlugin({
            template: "public/indexcdn.html",
            // (more)
        }),
        // (more)
    ],
    // (more)
}
```

Second, react needs to be defined as an *external* in the webpack config

```javascript
module.exports = {
    externals: {
        'react': 'React',
        // (more)
    },
    // (more)
}
```

This assumes above ```<script>``` did create a global ```React``` (a.k.a. ```window.React```) which we can use now: every ```require('react')``` or ```import react``` will now return this ```window.React```. 


## The same for CSS

(at least as long as it's not modules)

For imported CSS like *Bootstrap* or *FontAwesome*, one can do the same. The import of the style does not actually use whatever is returned by the import (rather, it triggers the style being added to the head), like 

```javascript
import 'font-awesome/css/font-awesome.css';
```

so, what does work (though a tiny little bit hacky) is the following webpack config

```javascript
module.exports = {
    externals: {
        'font-awesome/css/font-awesome.css': 'window',
        // (more)
    },
    // (more)
}
```

(the returned ```window``` is just discarded - all it takes is any value that is always available globally, and ```window``` sure is)

In addition to that, it again requires font-awesome to be included from the CDN in the head of the template HTML (like *indexcdn.html*): 

```html
<link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css" crossorigin="anonymous" integrity="sha256-eZrrJcwDc/3uDhsdt61sL2oOBY362qM3lon1gyExkL0=" />
```

CORS and SRI work the same as before; as webpack never actually descends into the FontAwesome CSS, all the webfont files will also be loaded from CDN.

The same method will **not** work when using CSS modules (i.e. really importing style names from a CSS and have prefixes with it). But on the other hand: these styles will most certainly be one's own (and are loaded from one's own bundle) anyway.


These steps are all it takes, really, to get a CDN working with webpack. As ever, hope you find it useful & let me know what you think!
