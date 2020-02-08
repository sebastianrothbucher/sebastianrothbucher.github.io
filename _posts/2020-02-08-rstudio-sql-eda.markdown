---
layout: post
title: "Using RStudio to explore data in MariaDB"
date:   2020-02-08 16:00:00 +0200
categories: sql databases r rstudio edi analysis explore mariadb
---

Most 'default' database UIs are OK(-ish) but not really exciting or really nice to work with. And even with a Frontend-focus it's important to understand data at hand to present it right. Strange behavior, memory hunger, no history of commands or post-processing of results almost made me write sth on my own (I have a Frontend focus, after all) - however: there's help already available. Sth with an enormous and committed community: *R* and *RStudio*. So here's how to tackle typical database tasks in RStudio - with history / recall, saving queries, and a lot more...

Hitting the back button at the wrong time in *phpMyAdmin* sure does not produce what you normally want (still great and helpful tool, though) - esp. as your carefully crafted query is, most likely, just lost. None of the alternative clients (all downloaded full-blown apps) did convince me - so before sitting down and starting writing (w/ little time at hand a longer journey) I actually 'found' sth I had already installed anyway: *RStudio*. 

There's comprehensive database support - for MariaDB, that would be [RMariaDB](https://rmariadb.r-dbi.org/) as well as recall, history, post-processing, even creating nice reports. 

So, here's how to use one of my favorite apps (RStudio) as SQL client...

## Getting started

First of all, it's simply

```r
install.packages("RMariaDB")
```
[RMariaDB](https://rmariadb.r-dbi.org/) offers a crisp intro with all important commands. Your setup might be a little different - on my Mac, the dev DB needs an explicit user (&pw), otherwise, a 

```r
library(DBI)
con <- dbConnect(RMariaDB::MariaDB(), user='magento', password='somethingverysecret')
dbSendQuery(con, 'use magento230')
# ready for queries
```

is all it takes to get started.

## Running queries

```r
res <- dbSendQuery(con, 'select * from eav_attribute')
df <- dbFetch(res, n = 5)
dbClearResult(res)
```

selects up to five rows from ```eav_attribute``` (a table in Magento, the e-commerce solution); same goes for any other query. ```df``` is a vanilla R data frame. 

## Shortcuts

When doing EDA (as in Exploratory Data Analysis), being sloppy about my one DB connection is OK for me (yet hoping backend devs are very different when writing production code). That sloppiness lets me create shortcuts like

```r
df <- dbFetch(dbSendQuery(con, 'explain eav_attribute'))
```

along with the ```View(resdetail)``` (or clicking *resdetail* in the *Environment* tab of RStudio) gets me details on the ```eav_attributes``` table of Magento.

What makes one-liners so efficient is recall: you can try variations - and also get a previous query back from the *History* tab of RStudio really quickly.

There are warnings about the previous query being cancelled - but again: we're in EDA.

## Multi-tab / updating result sets

What's nice about RStudio is that each ```View(dataframe)``` opens a new data tab with contents - and that tab *auto-updates* each time the data frame updates. So running several queries in reacall and viewing results instantly is super-simple.

## Getting info about the DB

The above 

```r
df <- dbFetch(dbSendQuery(con, 'explain eav_attribute'))
```

retrieves the metadata on ```eav_attribute``` (could be any other table).

## To wrap things up...

... I didn't know amazing R(Studio) can be used as SQL client (for routine queries, for EDA, for anything really) - but now I do - and I *love* it! As ever, hope you find it useful & let me know what you think!
