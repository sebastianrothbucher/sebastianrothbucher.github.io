---
layout: post
title:  "knitting - using RMarkdown for fast and simple blogging"
date:   2018-02-03 18:20:00 +0100
categories: datascience R visualization RMarkdown blogging jekyll knitr
---

I set up my little *github.io* blog with [Jekyll] - so I already get to write in [markdown] which makes a nice and fluent process. When working with R (not only - but especially - with [RStudio]) I can go even further: I can write things in [RMarkdown], *knit* them, paste the result into Jekyll and, well, I'm *done*. 

This (quite short) post aims to outline my first test drive in bringing RMarkdown (aka [knitr]), RStudio and Jekyll together to do some nice post. 

## The idea

behind RMarkdown is to have markdown with embedded R snippets - which get executed by the *knit* process. The results (both text outputs and plots) result in new plain markdown (or PNG, respectively), just like the one I could have typed myself. Just that it's done in zero seconds and consistent for sure. 

There's also the possibility to use [pandoc] to translate the plain markdown to HTML or PDF (which is actually the default option in RStudio), but I don't even need that. I'm more than fine with just getting all my R code evaluated into a markdown that I can copy over to the *sebastianrothbucher.github.io* repo for Jekyll, *sed* in some customizations and check in.

## My approach

(which must not be yours, but feel free to copy & paste from it). 

First of all, I need some post (like this one) as *.Rmd* (R markdown) file that includes R code resulting in either text: 


```r
2+2
```

```
## [1] 4
```

or a plot: 


```r
curve(log(x/2), 1, 10)
```

![plot of chunk plottest]({{ site.url }}/assets/knitting-plottest-1.png)

The file can also contain all the headers customary for Jekyll like *layout*, *title*, *date*, *categories*.

In short, the "secret" is blocks of:

~~~
```{r}
# some R code
```
~~~

When using [RStudio], I get a small *play* icon next to each block to like preview the result. I also have a *Knit* option to produce PDF or HTML - which I skip. Instead, I use 

```
Rscript -e 'library("knitr"); knit("knitting.Rmd")'
```

on the shell to *knit* the file (here: *knitting.Rmd*), i.e. to evaluate all R that is inside the file and produce a plain markdown. I get *knitting.md* along with a *figures* folder containig the plots as PNG. (I use bash on a Mac, you can also get bash for Windows along with [git] or bash for Linux - just kidding.)

I'm almost done - all it takes now to publish is copying figures and the markdown file, adjusting file paths to the Jekyll convention (I have the repo for my *github.io* page in ```$HOME/git/site```, give prefix ```knitting-``` for images and name the post ```2018-02-03-knitting-rmarkdown-jekyll-blogging.markdown```:

```
for i in $(ls figure); do cp figure/$i "$HOME/git/site/assets/knitting-$i"; done
cat knitting.md | sed -e 's/(figure\//(\{\{ site.url \}\}\/assets\/knitting-/g' > $HOME/git/site/_posts/2018-02-03-knitting-rmarkdown-jekyll-blogging.markdown
```
... and previewing in local Jekyll + pushing to github.

## So...

RMarkdown does indeed play nicely with Jekyll, I can use a local folder or another git repository to assemble all data, code, etc. and publish in one go. It's a little more learning and setup work than just starting to type in markdown. On the flipside: if I change a little thing in the R code (or get a newer / better data set to analyze), I get the publication updated completely and consistently with *one click* (or rather: three command-recalls on the shell).

Finally: there is a great [RMarkdown cheatsheet](https://www.rstudio.com/wp-content/uploads/2015/02/rmarkdown-cheatsheet.pdf) available from the RStudio team that did help me a lot. And: when wanting to write to write *about* the backtick syntax, there is info on [getting verbatim](http://rmarkdown.rstudio.com/articles_verbatim.html).

I put up a [gist](https://gist.github.com/sebastianrothbucher/8e1fc0cb2b84ce7dfd4bcfc33fe8faf1) with the base *knitting.Rmd* file if you want to compare.

As ever, I hope you find this useful. Feel free to try for yourself - and let me know what you think!

[Jekyll]: https://github.com/jekyll/jekyll
[markdown]: https://en.wikipedia.org/wiki/Markdown
[RStudio]: https://www.rstudio.com/
[RMarkdown]: http://rmarkdown.rstudio.com/
[knitr]: https://cran.r-project.org/web/packages/knitr/
[pandoc]: https://pandoc.org/
[git]: https://git-scm.com/
