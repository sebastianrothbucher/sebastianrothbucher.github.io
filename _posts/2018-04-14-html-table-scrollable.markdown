---
layout: post
title: "Scrollable tables"
date:   2018-04-14 13:15:00 +0200
categories: web html css table scroll
---

Scrolling the table contents and having a sticky header is like the eternal problem in Web development. Setting a *max-height* plus *overflow-y* to tbody won't cut it. Using a table component library of some sorts is not a joyful solution either. However, there is hope: with just a *tiny* bit of JS, one get get quite far. 

In the end, I wanted to use the *table* tag instead of being dragged down by a library where the most basic tweaking quickly becomes painful. The browser has a great layout engine - and there's great table styles like in [bootstrap css](https://getbootstrap.com/docs/3.3/css/) or by DIY. A quick research got me to this [stackoverflow post](https://stackoverflow.com/questions/14834198/table-scroll-with-html-and-css) (btw: the number of views speaks for itself) with a bunch of ideas (inspired and sometimes crazy) - yet, none really struck a chord with me. 

So, I set out to contribute my solution - bearing in mind these goals: 

* the width of each column is dynamic and determined by the content (I could still later go in and make cols user-sizable, but first of all, I want to focus on using the browser's layout engine)
* same goes for the height of rows (header and content)
* the header shall be **sticky** (which is the point of it all)
* scrollbars with actual width (like on Windows) shall be handled nicely
* ditto different resolutions / zoom levels (actually s/o pointed out this last one on codepen - thanks again!)

The essential idea is to render the table in full within a scrollable div and replicate the header row above it after rendering (i.e. using the widths the browser computed). Here's the steps:

* create two divs within a ```display: inline-block```
* in the first div, put a table with only the header (header table ```tabhead```)
* in the 2nd div, put a table with header and data (data table / full table ```tabfull```)
* use JavaScript, use ```setTimeout(() => {/*...*/})``` to execute code after render / after filling the table with results from ```fetch```
* measure the width of each th in the data table (using ```clientWidth```) 
* apply the same width to the counterpart in the header table
* set visibility of the header of the data table to hidden and set the margin top to -1 * height of data table thead pixels

With a few tweaks, this is the method to use (for brevity / simplicity, I used [d3js](https://d3js.org/), the same operations can be done using plain DOM): 

```javascript
setTimeout(() => { // pass one cycle
  d3.select('#tabfull')
    .style('margin-top', (-1 * d3.select('#tabscroll').select('thead').node().getBoundingClientRect().height) + 'px')
    .select('thead')
      .style('visibility', 'hidden');
  let widths=[]; // really rely on COMPUTED values
  d3.select('#tabfull').select('thead').selectAll('th')
    .each((n, i, nd) => widths.push(nd[i].clientWidth));
  d3.select('#tabhead').select('thead').selectAll('th')
    .each((n, i, nd) => d3.select(nd[i])
          .style('padding-right', 0)
          .style('padding-left', 0)
          .style('width', widths[i]+'px'));
})
```

Waiting on render cycle has the advantage of using the browser layout engine thoughout the process - for any type of header; it's not bound to special condition or cell content lengths being somehow similar. It also adjusts correctly for visible scrollbars (like on Windows).

I've added this to aforementioned stackoverflow post and I've put up a [codepen](https://codepen.io/sebredhh/pen/QmJvKy) with the full example. It random-generates values for quite different cols. In fact, for a solution with close to no tweaking yet, this works quite nicely across browsers (incl. IE in this [gist](https://gist.github.com/sebastianrothbucher/97444a9cae32e6fca8ef576f22cf4ec7) and Safari).  

As ever, I hope you found this useful or inspiring - let me know what you think!
