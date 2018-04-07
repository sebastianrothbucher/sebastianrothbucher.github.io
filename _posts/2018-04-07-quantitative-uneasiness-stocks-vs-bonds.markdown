---
layout: post
title: "Quantitative Uneasiness - Timeseries analysis of stocks VS bonds"
date:   2018-04-07 13:11:00 +0200
categories: datascience R visualization financial ggplot tidyverse
---

When putting money aside, there's plenty of choice between assets - and it seems sensible that broad measures like QE and the state of the economy have some influcence. This blog post checks out how different asset classes (namely stocks and bonds) develop over time.

## Intro

*Quantitative Easing* (QE) describes the idea of essentially replacing (decreasing) interbank lending by (increasing) lending facilties from central banks (read up e.g. wikipedia on [Quantitative Easing - QE](https://en.wikipedia.org/wiki/Quantitative_easing)). One intended consequence is a relatively low interest rate throughout the economy. Per se, this makes bonds less favorable to invest in over stocks - as bonds' [Yield to Maturity - YTM](https://en.wikipedia.org/wiki/Yield_to_maturity) decreases. YTM is a rough approximation of the return the investment, well, *yields* (it leaves out e.g. reinvestment - but that's good enough nonetheless). When you're intested in the details and mechanics involved, there is e.g. Inside the Yield (by Homer and Leibowitz - at any bookstore near you or your browser) with a ton of insight on the subject. 

Anyways, when you put a little money aside and realize there's a significant portion of stocks in the mix, you might (as I do) wonder whether that's still smart and (though no one can predict the future) what relation between the asset classes there were in the past. That's the aim of this piece: to (visually) track indices from both worlds and see how they develop over time. It's less about cyclicity (what *timeseries analysis* in data science books is often about) than about the relation *between* asset classes over time. Even more so given starkly different environments and sentiments in the course of about the last decade and a half.

Plus: I wanted to learn more about time-related analysis in R - which makes a great combination.

**A word of caution: this is NOT investment advice. Following this article will likely make you lose your last shirt. Use it and other sources to read up and then make your very own choices. Whatever happens, don't blame it on me.**

## Preparing the data set

First of all, I need a couple (few) libraries, let's use the *tidyverse*


```r
library(tidyverse)
library(lubridate)
```

Secondly, [Quandl](https://www.quandl.com/) is a nice source for financial data beyond simply one stock quote. Some of it is free. It might not be up to the current day, but it goes back quite some years and is good enough for me. Specifically, I pull these data sets:

* the NASDAQ composite (a very broad look at the world's stocks) - provided by NASDAQ OMX (per se, the index goes *up* when more investments in stocks are being made).
* the yield of a 10-year zero bond in the U.S. - provided by the ECB (per se, the yield goes *down* when more investments in bonds are being made - at least for a constant reference rate, e.g. the marginal lending rate).
* the marginal lending rate of the ECB for banks in the Euro common currency area (vulgo *Key Interest Rate*) - provided by the ECB (they have to know, after all).
* the Bank of America Merrill Lynch Global Bond Index for below invest grade as well as tripple A. These two are not from Quandl but from the [St. Louis Fed](https://research.stlouisfed.org/)

I get them (after signing up) via CURL (you have to apend your Quandl API key in *yourkey*): 
```
curl 'https://www.quandl.com/api/v3/datasets/NASDAQOMX/COMP.csv?api_key=yourkey' -o nasdaq.csv
curl 'https://www.quandl.com/api/v3/datasets/ECB/FM_M_US_USD_RT_BZ_USD10YZ_R_YLDE.csv?api_key=yourkey' -o bond10yrus.csv
curl 'https://www.quandl.com/api/v3/datasets/ECB/FM_B_U2_EUR_4F_KR_MLFR_LEV.csv?api_key=yourkey' -o margineur.csv
curl 'https://fred.stlouisfed.org/graph/fredgraph.csv?chart_type=line&recession_bars=on&log_scales=&bgcolor=%23e1e9f0&graph_bgcolor=%23ffffff&fo=Open+Sans&ts=12&tts=12&txtcolor=%23444444&show_legend=yes&show_axis_titles=yes&drp=0&cosd=2013-02-06&coed=2018-02-05&height=450&stacking=&range=Custom&mode=fred&id=BAMLHYH0A0HYM2TRIV&transformation=lin&nd=1986-08-31&ost=-99999&oet=99999&lsv=&lev=&mma=0&fml=a&fgst=lin&fgsnd=2009-06-01&fq=Daily%2C+Close&fam=avg&vintage_date=&revision_date=&line_color=%234572a7&line_style=solid&lw=2&scale=left&mark_type=none&mw=2&width=1168' -o bofamlbndsub.csv
curl 'https://fred.stlouisfed.org/graph/fredgraph.csv?chart_type=line&recession_bars=on&log_scales=&bgcolor=%23e1e9f0&graph_bgcolor=%23ffffff&fo=Open+Sans&ts=12&tts=12&txtcolor=%23444444&show_legend=yes&show_axis_titles=yes&drp=0&cosd=2013-02-06&coed=2018-02-05&height=450&stacking=&range=Custom&mode=fred&id=BAMLCC0A1AAATRIV&transformation=lin&nd=1988-12-16&ost=-99999&oet=99999&lsv=&lev=&mma=0&fml=a&fgst=lin&fgsnd=2009-06-01&fq=Daily%2C+Close&fam=avg&vintage_date=&revision_date=&line_color=%234572a7&line_style=solid&lw=2&scale=left&mark_type=none&mw=2&width=1168' -o bofamlbndaaa.csv
```

If you're a PhD economist, you've probably started crying long before. And I know: this is incredibly imprecise and superficial - and it's not my intention to be scientific (**which I am not**): I'm not going not apply for your job anytime soon, I just want to show how a little bit of R and basic analysis empowers everone to dig a little deeper. After all, we should make informed decisions and read up a little more. 

A propos reading up: NYT did a great piece about how everyone struggles with figguring out [when exactly the (stock) party will be over](https://www.nytimes.com/2018/01/04/business/market-dow-2018.html?_r=0). Also listening up can work: Monocle24 investigated the [recent selloff in stocks](https://monocle.com/radio/shows/the-bulletin-with-ubs/175/) and the not-so-simple implications looking ahead.

Anyways, as I've got the files, I can read them, combine them and fill the blanks: 

```r
nasd <- read_csv('nasdaq.csv')
bnds <- read_csv('bond10yrus.csv')
mlsb <- read_csv('bofamlbndsub.csv')
mlta <- read_csv('bofamlbndaaa.csv')
marg <- read_csv('margineur.csv')
```

```r
# now merge ('full join' - tidyverse makes it so much easier) all of them
all <- full_join(x = nasd[,c('Trade Date', 'Index Value')], y = bnds, by = c("Trade Date" = "Date")) %>%
  full_join(marg, by = c("Trade Date" = "Date")) %>%
  full_join(mlsb, by = c("Trade Date" = "DATE")) %>%
  full_join(mlta, by = c("Trade Date" = "DATE"))
colnames(all) <- c('Date', 'Nasdaq', 'Zerobond', 'MarginEur', 'MlSub', 'MlAaa')
# check if I have empty dates (hopefully not, and I get 0)
nrow(all[is.na(all$Date),])
```

```
## [1] 0
```

```r
# check if I duplicate b/c of factor madness (hopefully not, and I get 0)
length(all$Date)-length(unique(all$Date))
```

```
## [1] 0
```

```r
# sort by date and fill
all <- all %>%
  arrange(Date) %>%
  # carry forward values (i.e. assume a value carries next days until new one is in)
  fill(Nasdaq, Zerobond, MarginEur, MlSub, MlAaa) %>%
  # I only care from when I have at least core values 2 compare
  filter(!(is.na(Nasdaq) | is.na(Zerobond) | is.na(MarginEur)))
# (final check / glimpse)
head(all)
```

```
## # A tibble: 6 x 6
##   Date       Nasdaq Zerobond MarginEur MlSub MlAaa
##   <date>      <dbl>    <dbl>     <dbl> <dbl> <dbl>
## 1 2003-01-21   1364     4.49      3.75    NA    NA
## 2 2003-01-22   1358     4.49      3.75    NA    NA
## 3 2003-01-23   1394     4.49      3.75    NA    NA
## 4 2003-01-24   1340     4.49      3.75    NA    NA
## 5 2003-01-27   1320     4.49      3.75    NA    NA
## 6 2003-01-28   1346     4.49      3.75    NA    NA
```

```r
summary(all)
```

```
##       Date                Nasdaq        Zerobond       MarginEur    
##  Min.   :2003-01-21   Min.   :1262   Min.   :1.366   Min.   :0.250  
##  1st Qu.:2006-11-01   1st Qu.:2136   1st Qu.:2.326   1st Qu.:0.750  
##  Median :2010-08-17   Median :2577   Median :3.353   Median :1.750  
##  Mean   :2010-08-11   Mean   :3155   Mean   :3.519   Mean   :2.142  
##  3rd Qu.:2014-05-26   3rd Qu.:4278   3rd Qu.:4.788   3rd Qu.:3.000  
##  Max.   :2018-02-05   Max.   :7261   Max.   :5.831   Max.   :5.250  
##                                                                     
##      MlSub            MlAaa      
##  Min.   : 948.1   Min.   :511.3  
##  1st Qu.:1026.0   1st Qu.:546.6  
##  Median :1069.5   Median :574.0  
##  Mean   :1090.5   Mean   :575.6  
##  3rd Qu.:1152.7   3rd Qu.:604.9  
##  Max.   :1274.0   Max.   :637.6  
##  NA's   :2562     NA's   :2562
```

Which is pretty cool as I have data for the last 15 years (way pre-Lehman) to look at. That the BofA/ML data is slightly less is not optimal, but let's just live with it. As a side note: Nasdaq rose from 1261.79 to 7261.06 within the timeframe which is an annualized return of approx. 12.4%.

## Initial plot

Having a really nice and tidy dataset, let's see how it looks like for the absolute basis: stocks (red) VS (zero)bonds (blue): 


```r
g <- ggplot(data=all) +
  geom_line(mapping=aes(x=Date,y=((Nasdaq-min(all$Nasdaq))/(max(all$Nasdaq)-min(all$Nasdaq))), color = "c0")) +
  geom_line(mapping=aes(x=Date,y=((Zerobond-min(all$Zerobond))/(max(all$Zerobond)-min(all$Zerobond))), color = "c1")) +
  geom_line(mapping=aes(x=Date,y=((MarginEur-min(all$MarginEur))/(max(all$MarginEur)-min(all$MarginEur))), color = "c2")) +
  scale_color_manual(name = "", values=c("c0" = "red", "c1" = "blue", "c2" = "black"), labels = c("c0" = "Nasdaq", "c1" = "Zerobond", "c2" = "ECB margin")) +
  theme(legend.position="bottom") +
  labs(x = NULL, y = NULL)
g
```

![plot of chunk plot1]({{ site.url }}/assets/timeseries-plot1-1.png)

I know, normalizing to percent of range is only the 2nd best solution (instead of just using 5 overlaping scales) - but it shows what I want to show: the percentual relation of how stocks and bonds correlate. And it seems indeed they correlate - but rather negatively, i.e. I need the inverse. So let's plot the stocks (red) to 1-(zero)bonds (blue):


```r
g <- ggplot(data=all) +
  geom_line(mapping=aes(x=Date,y=((Nasdaq-min(all$Nasdaq))/(max(all$Nasdaq)-min(all$Nasdaq))), color="c0")) +
  geom_line(mapping=aes(x=Date,y=(1-((Zerobond-min(all$Zerobond))/(max(all$Zerobond)-min(all$Zerobond)))), color="c1")) +
  geom_line(mapping=aes(x=Date,y=(1-((MarginEur-min(all$MarginEur))/(max(all$MarginEur)-min(all$MarginEur)))), color = "c2")) +
  scale_color_manual(name = "", values=c("c0" = "red", "c1" = "blue", "c2" = "black"), labels = c("c0" = "Nasdaq", "c1" = "1-Zerobond", "c2" = "1-ECB margin")) +
  theme(legend.position="bottom") +
  labs(x = NULL, y = NULL)
g
```

![plot of chunk plot2]({{ site.url }}/assets/timeseries-plot2-1.png)

Now for a 30 seconds EDA, this is really cool already: there seems to be something to the initial hypothesis - but it's by far not that easy as there is quite some time lag and differences to it. Looking at this over time first of all reveals that: 

* while the bond market (and marginal lending rates) hit a bottom, stocks hit an all-time high
* bonds more or less track the marginal lending rate
* during severe crisis (around 2008) both bond yields and stock prices lose at the same time
* there is no clear indication (just yet), what predates what, i.e. whether stocks predate bonds or vice versa, but it would be an interesting topic to research. 

I don't want to imply anything here - it's simply what happens over time. One might think up a plethora of reasons (like stocks getting too attractive to miss, or enough money to leverage speculation, etc.). But I don't want to get there. I'm really curious about what just *happens*, i.e. which movements actually take place.

## Quarter to Quarter view

So, to start out, let's see how each indicator develops from the beginning of a quarter to the end (= the beginning of the next). In the lingo of literature, create a *window*. There's libraries to do so (like [window](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/window.html) from the stats package); yet in the end, the result counts. Here, I have a data frame (or rather *tibble*), not a *ts* (time series), so I go ahead and do it myself. As # of observations per month is not necessarily constant, I join in the first day of each quarter of each year (Jan 1, Apr 1, Jul 1, Oct 1) that's within the timeframe (and poss. fill-forward): 


```r
quarterStarts <- data_frame(Date=make_date(sort(rep(c(min(year(all$Date)):max(year(all$Date))),4)), c(1,4,7,10), 1)) %>%
  filter(Date>=min(all$Date) & Date<=max(all$Date))
allWithQ <- full_join(all, quarterStarts, by = "Date") %>%
  arrange(Date) %>%
  # carry forward values (i.e. assume a value carries next days until new one is in)
  fill(Nasdaq, Zerobond, MarginEur, MlSub, MlAaa)
allQ <- inner_join(allWithQ, quarterStarts, by = "Date") %>%
  arrange(Date)
nrow(allQ) # should be about 15*4 (60) take or leave a few
```

```
## [1] 60
```

```r
# so I have a quarterly overview
head(allQ)
```

```
## # A tibble: 6 x 6
##   Date       Nasdaq Zerobond MarginEur MlSub MlAaa
##   <date>      <dbl>    <dbl>     <dbl> <dbl> <dbl>
## 1 2003-04-01   1356     4.50      3.50    NA    NA
## 2 2003-07-01   1642     4.10      3.00    NA    NA
## 3 2003-10-01   1832     4.65      3.00    NA    NA
## 4 2004-01-01   1997     4.92      3.00    NA    NA
## 5 2004-04-01   2019     4.55      3.00    NA    NA
## 6 2004-07-01   2007     5.33      3.00    NA    NA
```

So, I'm good to go - first computing the difference quarter over quarter (it's already condensed to a couple of records, so I go without self-joining and mapping): 

```r
allQ$NasdaqDelta <- 0
allQ$ZerobondDelta <- 0
allQ$MarginEurDelta <- 0
for (i in c(1:(nrow(allQ)-1))) {
  allQ$NasdaqDelta[i] <- (allQ$Nasdaq[i+1]/allQ$Nasdaq[i]-1)
  allQ$ZerobondDelta[i] <- (allQ$Zerobond[i+1]/allQ$Zerobond[i]-1)
  allQ$MarginEurDelta[i] <- (allQ$MarginEur[i+1]/allQ$MarginEur[i]-1)
}
head(allQ[,c("Date", "NasdaqDelta", "ZerobondDelta", "MarginEurDelta")])
```

```
## # A tibble: 6 x 4
##   Date       NasdaqDelta ZerobondDelta MarginEurDelta
##   <date>           <dbl>         <dbl>          <dbl>
## 1 2003-04-01     0.210         -0.0884         -0.143
## 2 2003-07-01     0.116          0.132           0    
## 3 2003-10-01     0.0897         0.0592          0    
## 4 2004-01-01     0.0113        -0.0754          0    
## 5 2004-04-01    -0.00615        0.172           0    
## 6 2004-07-01    -0.0321        -0.109           0
```

So now let's determine a momentum as discrete -1 (down), 0 (flat - or almost), 1 (up). I start out setting a threshold for "flat" (merely following intuition after looking at the data set; can still adjust):


```r
thresh <- 0.018
allQ$NasdaqMom <- ifelse(abs(allQ$NasdaqDelta)>thresh, sign(allQ$NasdaqDelta), 0)
allQ$ZerobondMom <- ifelse(abs(allQ$ZerobondDelta)>thresh, sign(allQ$ZerobondDelta), 0)
allQ$MarginEurMom <- ifelse(abs(allQ$MarginEurDelta)>thresh, sign(allQ$MarginEurDelta), 0)
head(allQ[,c("Date", "NasdaqMom", "ZerobondMom", "MarginEurMom")])
```

```
## # A tibble: 6 x 4
##   Date       NasdaqMom ZerobondMom MarginEurMom
##   <date>         <dbl>       <dbl>        <dbl>
## 1 2003-04-01      1.00       -1.00        -1.00
## 2 2003-07-01      1.00        1.00         0   
## 3 2003-10-01      1.00        1.00         0   
## 4 2004-01-01      0          -1.00         0   
## 5 2004-04-01      0           1.00         0   
## 6 2004-07-01     -1.00       -1.00         0
```

Now let's bring this on a timeline - plotting the ups/downs/flats along time. Again, amazing ggplot2 & tidyverse make that quite joyful: 


```r
ggplot(data=allQ) +
  geom_point(mapping=aes(x=Date, y="m0", shape = paste0("mom", NasdaqMom), color = paste0("mom", NasdaqMom))) +
  geom_point(mapping=aes(x=Date, y="m1", shape = paste0("mom", ZerobondMom), color = paste0("mom", ZerobondMom))) +
  geom_point(mapping=aes(x=Date, y="m2", shape = paste0("mom", MarginEurMom), color = paste0("mom", MarginEurMom))) +
  scale_shape_manual(name = "", values = c("mom-1" = 6, "mom0" = 1, "mom1" = 2)) +
  scale_color_manual(name = "", values = c("mom-1" = "red", "mom0" = "grey", "mom1" = "green")) +
  scale_y_discrete(name = "", labels = c("m0" = "Nasdaq", "m1" = "Zerobond", "m2" = "ECB margin")) +
  theme_light() + theme(legend.position = "none", aspect.ratio = 0.15) +
  labs(x = NULL, y = NULL)
```

![plot of chunk plot3]({{ site.url }}/assets/timeseries-plot3-1.png)

Now that shows a way more mixed bag than initially presumed: both stocks and bonds have the same momentum over several quarters (before, during and after 2008). It's not that suprising that it gets more expensive to borrow money in a booming economy with higher rates being paid for many things (and investors having the alternatives like stocks to invest in). Hence the positive correlation between all three indicators upwards around 2005 and down during the financial crisis around and after 2008. Likewise, it's not suprising that bond yiels at least roughly track the marginal lending rate (which is kind of the intention of the latter).

As the data is ready from preparing the momentum, let's compare the performance quarter of quarter of the three indicators. Just plotting the delta side by side won't cut it, though: the bond yield is already a delta (while Nasdaq is purely market capitalization based a.k.a. yield-free) - so I go ahead and plot the quarter over quarter delta of stocks VS a quarter of the annual return in that quarter. Point is: if you bought/sold stocks and cashed in on the delta: how well would you have fared in comparison to getting a return on a bond? It's all an approximation - but a good-enough indication anyway. As the extremes are quite different, I go for the (blunt but effective) solution of adjusting the Nasdaq's graph):


```r
ggplot(data = allQ) +
  geom_line(mapping = aes(x = Date, y = NasdaqDelta/8+0.04, color = "c0")) +
  geom_line(mapping = aes(x = Date, y = (Zerobond/100), color = "c1")) +
  geom_line(mapping = aes(x = Date, y = (MarginEur/100), color = "c2")) +
  scale_x_date(breaks = c(ymd(20080101),ymd(20100101),ymd(20130101),ymd(20150101),ymd(20170101)), date_minor_breaks = "3 months" , labels = c(2008,2010,2013,2015,2017)) +
  scale_y_continuous(minor_breaks = NULL, labels = NULL) +
  scale_color_manual(name = "", values=c("c0" = "red", "c1" = "blue", "c2" = "black"), labels = c("c0" = "Nasdaq (delta)", "c1" = "Zerobond", "c2" = "ECB margin")) +
  theme(legend.position="bottom") +
  labs(x = NULL, y = NULL)
```

![plot of chunk plot4]({{ site.url }}/assets/timeseries-plot4-1.png)

Now this graph does not provide an easy solution either - but at least it points to something better than randomness: 

- in the years up to Lehman's collapse, a drop in stocks predates a hike in bonds and vice versa
- immediately after Lehman, there is only one direction (a.k.a. south). Supports the notion that an unhealthy economy just does not yield anything for anybody.
- along with Obama's inauguration in early 2009, both stocks and bonds recover (supporting the notion that hope for recovery is good across the board - so it cuts both ways).
- from 2010 to about 2012, it seems a little like 'back to normal' (in the sense that both move in the opposite direction)
- from about 2012/2013 on (Obama's 2nd term), stocks perform constantly well, bonds perform constantly sub-par; but if there is a change, the change goes in the *same* direction.
- there is more than no (but little) evidence that from the 2nd half of 2015 and even more in the last quarters shown (end of 2017), we get back to a negative correlation, i.e. bonds re-bounding and stocks performing less well than before.

Furthermore, taking ECB into account, both compared with stocks and with bonds:

- bonds more or less track (i.e. are positively correlated) with the marginal interest rate
- whereas stocks are decidedly not

Which would suggest that the influence of marginal interest is significant. What should be noted: the lower values in stock performance actually mean money lost whereas the lower values for bonds (and ECB) just mean less money gained.

At the end of a long day, it was a lot of detours - because there is no simplistic 'if one goes down then the other goes up' explanation. Good concepts try to take into account as many factors as possible, including

- the overall outlook and confidence that companies one invests in will find a ready set of buyers
- the availability of leverage in the first place
- specific expectations towards countries / industries / technologies (temporarily incl. hypes)
- Zeitgeist - e.g. risk aversaion of the majority
- fundamentals - expected return per asset class

So, checking the performance of buy / sell of stocks VS the yield of bonds leaves out several of those factors. Unsurprisingly, this comparison alone is not sufficient guidance. 

That said, however, it's not fully random, either: both the *stocks VS 1-bonds* comparison and the *delta VS yield* comparison suggest that, outside extreme distress, investors have, to an extent, preference for one or the other asset class. 

## Summary / Wrap-up

All overall, looking at the different graphs, the pure *up/down* view didn't really yield anyting. Whereas both the *stocks VS 1-bonds* comparison and the *delta VS yield* comparison show that, with a delay in time, there is phenomenae to be observed. Taking this observation as one of *several* parts of one's thinking can be helpful in understanding the near future of asset classes one can choose from when putting some money on the side. 

Anyways, there is a lot more one can do to shed light to it (I didn't even use two of the data sets yet), so I'll do a 2nd part looking at probabilities shifted over time, etc. What I want to do now is get a first writeup out of the door, to offer some result.

I've put up the base *R Notebook* plus the combined quarterly data set ```allQ``` as *xport_allQ.csv* into a [gist](https://gist.github.com/sebastianrothbucher/74f46c4db61161784c72d84c0c3c893f). You can go up to *Initial plot*, read *xport_allQ.csv* and play with the code yourself. You can even go further and download the files for yourself - you need an API key then.  

As ever, I hope you found this useful or inspiring - let me know what you think!
