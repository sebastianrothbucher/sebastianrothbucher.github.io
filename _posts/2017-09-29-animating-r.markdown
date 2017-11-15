---
layout: post
title:  "animating R"
date:   2017-09-29 19:37:00 +0200
categories: datascience R visualization animation tgif
---

R's [plot] function does not just take the plain data, it accepts pre-defined limits for the axis. This is useful for drawing several lines into the same diagram (one would not know the limits of all lines upfront). It also allows playing around with iterative (animated) drawing. Either by just showing in the console output or by saving into several files and creating a GIF from it.

So, I have the following ultra-complex test data: 

```R
df <- data.frame(score=c(8, 6, 9, 8, 7, 7, 5, 9, 10, 9, 7, 8))
```

I add another column just for numbering the rows: 

```R
df$try <- c(1:length(df$score))
```

Now I can go and loop over *n* - displaying from first up to the *n*th with the axis ever staying the same: 

```R
for (i in c(1:length(df$score))) {
  plot(df$try[1:i], df$score[1:i], type="b", xlim=range(df$try), ylim=range(df$score))
  Sys.sleep(0.5)
}
```

In the middle of the animation, I get a result like this: 

![Middle of the animation]({{ site.url }}/assets/animating.png)

likewise, I can go and create PNGs via

```R
for (i in c(1:length(df$score))) { 
  png(paste0("img",(10+i),".png"),width=530,height=350)
  plot(df$try[1:i], df$score[1:i], type="b", xlim=range(df$try), ylim=range(df$score))
  dev.off()
}
```

and then join them via e.g. [ImageMagick] \(on a Mac: ```brew install imagemagick```) to [create a GIF from PNGs](http://www.imagemagick.org/discourse-server/viewtopic.php?t=29830). It basically takes this shell command:

```
convert -loop 0 -delay 40 $(ls *.png |sort) animating.gif
```

Which produces: 

![graph animation]({{ site.url }}/assets/animating.gif)

That's it already. Running this (there is a [gist](https://gist.github.com/sebastianrothbucher/7420eeb6703b85093066e63440c7363c) available) produces an animated output. Have fun - and let me know what you think!

[plot]: https://stat.ethz.ch/R-manual/R-devel/library/graphics/html/plot.html
[ImageMagick]: http://www.imagemagick.org/
