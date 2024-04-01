---
layout: post
title: Visual Customer Journey from nothing but simple text in markdown
date: 2024-04-01 11:00:00 +0200
categories: pm project work briefing value quality delivery
---

The following is a Customer Journey, written in the quickest possible way with nothing but text. It's the quickest way I can think of right now to get a visual Customer Journey in place and ensure everyone's on the same page. From text, my script renders a visual representation - and you can use embed the script into your work or your blog as well. Here's how...

Take this visual journey:

```journey
See Insta ad
Get to landing
Register for preview (e-mail / fakedoor)
...
```

It's generated from the following codeblock in markdown: 

~~~
```journey
See Insta ad
Get to landing
Register for preview (e-mail / fakedoor)
...
```
~~~

To get it to work, add my script as a new paragraph somewhere inside your markdown: 

```html
<script src="https://sebastianrothbucher.github.io/markdown-customer-journey/journey.js" integrity="sha256-D3HE7MCcr+8t3sfyX6fjWfA7IWWGs7rP9OcY+UwNmQs=" crossorigin="anonymous"></script>
<link rel="stylesheet" href="https://sebastianrothbucher.github.io/markdown-customer-journey/journey.css" integrity="sha256-HS161UTL65qQ6bfBULdFvROSbEE9w8uTvXvVJbTvvAk=" crossorigin="anonymous" />
```

<script src="https://sebastianrothbucher.github.io/markdown-customer-journey/journey.js" integrity="sha256-D3HE7MCcr+8t3sfyX6fjWfA7IWWGs7rP9OcY+UwNmQs=" crossorigin="anonymous"></script>
<link rel="stylesheet" href="https://sebastianrothbucher.github.io/markdown-customer-journey/journey.css" integrity="sha256-HS161UTL65qQ6bfBULdFvROSbEE9w8uTvXvVJbTvvAk=" crossorigin="anonymous" />

And that's it!

The selector supports both [pandoc](https://pandoc.org/)'s rendering (`pre.journey` as well as github page's rendering `.language-journey`) of the code block.

You find the source code (MIT licensed) in this [github repo](https://github.com/sebastianrothbucher/markdown-customer-journey)

Many thanks to Jacob Okamoto who had this idea first - just for D3 and graphviz instead of some specific CSS+SVG. You can read about it in his blog here: [ howto => GraphViz in Markdown](https://oko.io/howto/graphviz-in-markdown/)

That's it. Hope you find this useful - as ever: let me know what you think!

