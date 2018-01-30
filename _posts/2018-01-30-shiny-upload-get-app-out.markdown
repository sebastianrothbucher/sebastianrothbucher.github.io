---
layout: post
title:  "Uploads in Shiny - getting a complete app out of the door"
date:   2018-01-30 13:30:00 +0100
categories: datascience R visualization Shiny upload
---

In [Shiny 4 real]({% post_url 2017-10-28-shiny-r-sentiment-interactive-web %}), I quickly put up a Web app in [Shiny] to publish some findings on sentiment analysis. For publishing findings or exposing an existing data set (poss. including a database) this is great already. But one thing is still missing: exposing a method/algorithm to people's own data sets in order to offer a really versatile app. In order words: allowing upload of files to handle. 

Amazing Shiny has you covered, though: there is a read-made facility [fileInput] which I'm going to use to re-package the code from the [gRasping the sentiment]({% post_url 2017-09-04-grasping-sentiment-r %}) post into a truly versatile app: getting the sentiment of *any* file or files folks upload. 

Basically, I can begin with the same basic layout as in [Shiny 4 real]({% post_url 2017-10-28-shiny-r-sentiment-interactive-web %}) and add a file upload on top of that. I end up with this: 

```R
library(shiny)

# (analysis logic)

shinyApp(
  ui = fluidPage(sidebarLayout(
    sidebarPanel(verticalLayout(
      uiOutput("songSelection"),
      uiOutput("sentimentSelection"),
      fileInput(
        "addFile",
        "Upload text file w/ song:",
        accept = "text/plain"
      )
    )),
    mainPanel(plotOutput("plot"))
  )),
  server = function(input, output, session) {
    # (call analysis of new file and update selects)
  }
)
```

I use ```uiOutput``` to dynamically change more aspects of the select boxes than I could normally (explanation see below) - otherwise, the use of ```fileInput``` is quite straightforward: I need to give it an ID and a caption (like with all Shiny elements), and I have the possibility to specify which type of file I want to accept. This doesn't guarantee anything - people could still upload something else, which will result in an exception. But I can live with that - possibly far into actual use of such an app.

To implement the logic, I use the code from [gRasping the sentiment]({% post_url 2017-09-04-grasping-sentiment-r %}) virtually verbatim. It just takes some refactoring it to run several times instead of once, extending initially empty ```dfAll``` / NULL ```dfDim``` as I go. I compute the matrix on the go to avoid issues with missing dimension names (```as.matrix``` seems to get a little nasty when having just one column). As I change global values inside a function, I need the ```<<-``` operator to ensure I actually update the global variable instead of creating another local (and vain) one. The overall code to compute the sentiment is the following (check above link for details on what's being done): 

```R
library(tidytext)
library(tm)

stp <- stopwords()
dfAll <- data.frame(sent=c(), cnt=c(), song=c())
dfDim <- NULL

analyzeNew <- function (uploadDf) {
  for(i in c(1:nrow(uploadDf))){
    natL <- readLines(uploadDf$datapath[i])
    nat <- tolower(unlist(strsplit(natL, "\\s|[,?\\.()]")))
    df <- data.frame(table(nat))
    dfF <- df[(df$nat!='' & is.na(match(df$nat, stp))),]
    dfM <- merge(dfF, sentiments, by.x=c('nat'), by.y=c('word'))
    dfMA <- aggregate(Freq~nat+sentiment, data=dfM, FUN=max) # !!!
    dfAgg <- aggregate(Freq~sentiment, data=dfMA, FUN=sum)
    colnames(dfAgg) <- c('sent', 'cnt')
    dfAgg$song <- sub("\\....$", "", uploadDf$name[i])
    dfAll <<- rbind(dfAll, dfAgg)
  }
  dfDim <<- data.frame(sent=unique(dfAll$sent))
  for(s in unique(dfAll$song)){
    dfSlc <- dfAll[dfAll$song==s, c('sent', 'cnt')]
    colnames(dfSlc) <- c('sent', s)
    dfDim <<- merge(dfDim, dfSlc, by=c('sent'), all=TRUE)
  }
  rownames(dfDim) <<- dfDim$sent
}
```

I can then listen for the upload and call the analysis inside the server function via ```observeEvent```:

```R
vals <- reactiveValues(redraw = 0, selSong = c(), selSent = c())
shinyApp(
  # (ui=...)
  server = function(input, output, session) {
    observeEvent(input$addFile, {
      oldSong <- unique(dfAll$song)
      oldSent <- unique(dfAll$sent)
      analyzeNew(input$addFile)
      newSong <- unique(dfAll$song)
      newSent <- unique(dfAll$sent)
      vals$selSong <- c(vals$selSong, setdiff(newSong, oldSong))
      vals$selSent <- c(vals$selSent, setdiff(newSent, oldSent))
      vals$redraw <- vals$redraw + 1
    }, ignoreNULL = TRUE, ignoreInit = TRUE)
    # (more logic)
  }
)
```

Basically, I call ```analyzeNew``` passing in a data frame containing (amongst others) name and temporary path of the file(s) uploaded. The reference manual of [fileInput] has an excellent summary of this. Around that, I do a little (and quite basic) UX: I make sure that all new songs and all sentiments being introduced by new songs are actually selected. I use my own variable instead of calling ```updateSelectInput``` to really influence every aspect of the selects (explanation shortly below). Finally, I really enforce a new paint cycle by incrementing the ```redraw``` variable by one. The ```reactiveValues``` is quite interesting: it's more than normal global variables in the sense that each change causes other observers to trigger. R is quite smart in the background about which observer uses which variables. In basic setups (like those in my previous post), one doesn't have to think about it in the first place as changes to ```input$...``` causes redraws of depending ```output$... <- ...``` statements automatically. Anyways, I get a method to make sure I redraw upon any change of variables - either changes in selection or changes due to a new upload. 

As changes in variables (reactive or not) won't do anything by itself, I need a handler to redraw the selects and the plot (```observe``` is needed to keep watching the ```reactiveValues``` defined above). ```renderUI```-ing the selects does allow for updating the actual selection based on logic (which otherwise won't work). Same goes for updating the length of the sentiments select box to always show all values. What I get is:

```R
shinyApp(
  # (ui=...)
  server = function(input, output, session) {
    # (prev logic)
    observe({
      output$songSelection <- renderUI(selectInput("songs", "Choose uploaded songs:", unique(dfAll$song), multiple = TRUE, selected = vals$selSong))
      output$sentimentSelection <- renderUI(selectInput("sentiments", "Choose one or more sentiments:", unique(dfAll$sent), size = length(unique(dfAll$sent)), multiple = TRUE, selectize = FALSE, selected = vals$selSent))
      if(vals$redraw > 0 && length(vals$selSong) > 0 && length(vals$selSent) > 0){
        output$plot <- renderPlot(barplot(t(as.matrix(dfDim[vals$selSent, vals$selSong])), names.arg=vals$selSent, beside = TRUE, legend = TRUE))
      }
    })
    # (even more logic)
  }
)
```

One of the longer quests was getting UX for one song right: unless ```names.arg``` is stated explicitly, there won't be any captions along the x-axis. Anyways, I finally need to link explicit manual selection to the reactive values: 

```R
shinyApp(
  # (ui=...)
  server = function(input, output, session) {
    # (prev logic)
    observeEvent(input$songs, {
      vals$selSong <- input$songs
    }, ignoreInit = TRUE, priority = -1)
    observeEvent(input$sentiments, {
      vals$selSent <- input$sentiments
    }, ignoreInit = TRUE, priority = -1)
  }
)
```

The priority of minus one (one less than the default of 0) ensures that these handlers always run **after** both processing an upload and redrawing the boxes. Otherwise, there is chaos about which sentiments are selected already. Giving an explicit priority makes all behavior deterministic again. Updating ```sel...``` values will automatically trigger above ```observe``` and redraw - which is pretty smart and avoids second guessing (or forgetting) about elements requiring an update.

Above code (put together - I also created a gist with it) is all that's needed to upload any file and analyze its sentiment.

When I start it (in [RStudio] or vanilla R), I get a UI like this:

![comparing some sentiments between two uploaded songs]({{ site.url }}/assets/shinyupload.png)

So, in a little more than a (domestic) flight, I got an app going that is able to analyze any song (or text, for that matter) via a Web interface. I could even go along and polish the UI some more if I wanted (like outlied in [Test-driving Shiny with CouchDB]({% post_url 2017-10-14-test-drive-shiny-r-rstudio-couchdb %})). Yet, I don't have to, as it looks great already and is totally usable from any browser. It's certainly an MVP (as in Minimum Viable Product) and (given this was a real offering) I could send out the link to get feedback.

I put the complete and runnable code into a [gist](https://gist.github.com/sebastianrothbucher/eb14e4223bbf4873973ae6992bad6d03) (you need the 3rd file called *sentiments_upload.R*).

As ever, I hope you find this useful - feel free to try and let me know what you think!

[Shiny]: http://shiny.rstudio.com/
[RStudio]: https://www.rstudio.com/
[fileInput]: https://shiny.rstudio.com/reference/shiny/latest/fileInput.html
