---
layout: post
title: "Tracking w/out hassle - finally - by properties in Obsidian / markdown"
date:   2024-02-03 18:00:00 +0100
categories: markdown pm management reporting controlling kpi obsidian
---

I have yet to meet someone who's like "cool, finally doing documentation!" or "at last I can track my hours, fun time!". It's just unnatural, (unnecessarily) complex, taking a ton of mental bandwidth, argh. At the same time, a solution done and well-document in the past may help down the line. If only writing it down was fast and easy. Same goes for hours - no invoice without them. Recently, I came across Obsidian and started using it a bit for my own stuff. And it looks like this program has a nice approach for both issues - making documenting easier and also making reporting hours and stuff easier. This post digs into some details.

## Why is it so frustrating?

I'd say: because you need a ton of mental bandwith to get anything done. Imagine just adding a small comment in Con****nce or one of the office suites: find the right page or doc, find the right part of it, switch to edit mode (and find the right part again), add sth small, "Your token explired" and so on and so on and so on. When all of this is done, you can essentially start over with whatever else you were really doing. 

I proposed a solution for this a while ago called *debriefit* (can [try live here](https://github.com/sebastianrothbucher/debriefit)) - and didn't get much love, tbh. BUT: seems like I was not alone with the idea of ***quickly*** looking up the right part and ***quickly*** adding sth small. 

What obsidian offers is the *Meta+O* command that has a quick search for every doc - so jumping somewhere is quick and does not use a lot of mental bandwith. So one issue solved. 

Secondly, doing sth trivial like reporting time spent on a task is a black hole for mental bandwith just the same. The spreadsheet I use (already a really great one) shows 60 ***(sixty)*** icons on top of a sheet. And you still need to select project again, and task type again, and date, and so on. Over time, I discovered many features that really makes things easier - but I know of very few people using even 10% of what's in there. 

Likewise: if your project desicides to use J**a or tools like that, you wish the 60 icons back. Especially the most (in)famous tool seems to have a contest going for making things hard. It sucks up all mental bandwith (at least in my opinion) for sth as trivial as "I've worked 5h on this today".

## The two features that make such a difference

As quickly mentioned above, I think two features stand out: 

*1:* Quickly opening any file with *Meta-O* (similar to *Meta-P* in VSCode) - make jumping to a doc easy - allowing inserting a small hint without big overhead. I find that often, small comments like "slowdown might be due to this queue; steps to troubleshoot: 1, 2, 3"

*2:* Editing properties that are machine readable (*Meta-P* lets you insert properties - and also remembers property names already used). This makes logging hours et al very easy.

Aside from that, the fact that one can even check in content and trace (just like code; little optimizations apply) makes a ton of sense as well. Having used plain files on github before, this is just one more step. When using it for business, the only disadvantage vis-a-vis plain github: it costs a bit of money. 

## Working with properties

Having used Jekyll for almost a decade, properties are just normal - every Jekyll post has properties for layout, title, keywords, etc. So does Rmarkdown. Take this post:

```
layout: post
title: "Tracking without hassle - finally - with properties in Obsidian"
date:  2024-02-03 18:00:00 +0100
categories: markdown pm management reporting controlling kpi obsidian
```

(by the way: most old-school office suites also have "custom document props" - just not the most widely-used feature)

Obsidian also supports those markdown-style properties.

Now consider the following example: basic facts on a project are listed as markdown properties: 

```
./Feature I/Main.md:
---
estimate: "40"
status: Ongoing
---

some more detail

./Feature I/Hours/SRo Feb 03.md:
---
timespent: "7"
date: 2024-02-03
---
Did the basics now%   

./Feature I/Hours/SRo Feb 04.md:
---
timespent: "5"
date: 2024-02-04
---
Added some more
```

All it takes now is a very simple script - like the following one - to pull reports

```
#!/bin/bash
echo '---' > Stats.md
echo -n 'estimate: ' >> Stats.md
grep -r 'estimate:' . | grep '.md' | grep -v Stats.md | grep -v grep | awk -F: '{gsub("\"", "", $3); sum = sum + $3} END {print sum}' >> Stats.md
echo -n 'timespent: ' >> Stats.md
grep -r 'timespent:' . | grep '.md' | grep -v Stats.md | grep -v grep | awk -F: '{gsub("\"", "", $3); sum = sum + $3} END {print sum}' >> Stats.md
echo '---' >> Stats.md
```

which results in sth like: 

```
Stats.md: 
---
estimate: 40
timespent: 12
---
```

Of course, doing more (like in node or Python) makes it even more powerful. Yet: sth as simple as above is enough. Which does not only make documentation a **ton** easier - but also allows for reports being pulled easily.

## Alternatives

With Gen-AI being all the craze, just firing off "Claim 5h on this task in this project" (for me, today) will be the way to go. Taking a bit of prompt engineering to sniff out correct values for project and task, that looks very feasible. 

Besides, as a super-simple option, just checking plain markdown into github can also work. Properties are available in markdown throughout - and search is a feature of IDEs throughout, including VSCode with *Meta-P* (yes, I know - almost the same). So I could have just written up "use VSCode to check markdown with properties to github". And that's a legit option. All scripts apply just the same. 

### VSCode snippets

VSCode has a snippets feature (see this [Introduction](https://dev.to/ceceliacreates/use-vs-code-snippets-to-generate-markdown-front-matter-fpc)) that allows for almost fully automating markdown properties - like timesheets. No tools needed outside VSCode, and it's *fast*.

Sticking with the timesheet example, you can create a file `.vscode/snippets.code-snippets` in your project and give it a content like this one: 

```json
{
  "Timesheet": {
		"scope": "markdown",
		"prefix": "timesheet",
		"description": "Timesheet entry",
		"body": [
			"---",
			"timespent: \"0\"",
      "date: 2000-00-00",
			"---",
			""
		]
	}
}
```

Now, in any mardown file, you can type `time` (or `timesheet`), press CTRL+SPACE and choose the snipped from the IntelliSense. 

## Wrap-up

There's a lot of value in markdown properties (allowing for automated processing of info) + good search. Obsidian is a way, much can be emulated with plain VSCode and github as well. In any case, it's a *massive* improvement vis-a-vis "legacy" tools. 

Feel free to try it out - and as ever: let me know what you think!
