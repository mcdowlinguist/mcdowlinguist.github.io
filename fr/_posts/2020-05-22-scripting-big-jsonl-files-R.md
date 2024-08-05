---
layout: post
title:  "Working with lots of big JSON Lines files in R"

categories: 
  - Scripting
date:   2020-05-22
tags: twitter R coronavirus
---

If you, like me, work with publicly available Twitter databases, you
might find yourself with a large number of huge JSON Lines files, like
the [COVID-19-TweetIDs
database](https://github.com/echen102/COVID-19-TweetIDs){:target=“\_blank”}.
In this database, you’ll find the IDs of all tweets since late January
2020 mentioning any of several COVID-19-related buzzwords, typically
divided into 23 files for each day. A Python script included in the
database turns each file of IDs into a .jsonl file of hydrated tweets,
some of which can exceed 1 Gb. Even with a decent computer, processing
and transforming these files into manageable data frames in R can be
pretty taxing, mostly because of the non-uniformity of the data and the
way the `jsonlite` package handles .jsonl files.

The following code loops through all the .jsonl files in a folder,
extracts only the fields you request and provides you with uniform .csv
files which can then later be re-imported in R without any of the
problems you may have run into working with Twitter data in this format.
Some commentary on the code follows.

{% highlight r %}
library(jsonlite)
library(rtweet)

## Replace '...' with the directory that has your files
setwd("...")
files = dir(pattern = "*.jsonl$")

## As many JSON fields as you want go here, for instance:
cols = c(
  "created_at",
  "id",
  "id_str",
  "full_text"
)

for (i in 1:length(files)){
  start = Sys.time()
  lines = readLines(files[i])
  temp = do.call(
    rbind, 
    lapply(
      lines, function(x)
        unlist(jsonlite::fromJSON(x))[cols]
    )
  )
  colnames(temp) = cols
  y = as.data.frame(temp)
  save_as_csv(y, sub("jsonl", "csv", files[i]), prepend_ids = T, fileEncoding = "UTF-8")
  end = Sys.time()
  message(files[i], ": ", round(difftime(end, start, units = "mins"), 2), 
    " minutes for ", length(lines), " tweets")
  # file.remove(files[i])
  ## Uncomment above if you want the jsonl files to be permanently deleted (like if space is an issue)
}
{% endhighlight %}

In this database at least, each line of the .jsonl (that is, each tweet)
has a variable amount of information, from 200 to – on rare occasions –
upwards 600 fields. In my study, I only need about 30, some of which
(like geographic information) may additionally be entirely absent from a
given tweet. As of writing, the `fromJSON()` call from the `jsonlite`
package automatically takes care of this when the `flatten = TRUE`
option is specified, but only with standard .json files. Trying to run
this command on an entire .jsonl file will run into an error (“trailing
garbage”) at the first line break. Note also that this command now
returns a data frame from .json files but lists from individual .jsonl
lines.

We could work around this by brute-forcing a conversion from each .jsonl
to .json, for example, by replacing all line breaks (save for the last)
with a comma and by placing the entire string between \[ \], but this
can be a time-consuming process and proves ultimately unnecessary.

As some solutions on StackOverflow suggest, we could import and unlist
each tweet individually in order to build a database iteratively in a
`for` loop. As this will provide us named character vectors of differing
lengths, we can simply name in advance those elements that we want to
retain (if present) or append (if absent) by creating a vector of field
names (called `cols` here) and subsetting the output of `unlist()`.
Since the names of those fields absent from a given tweet will be
missing (in addition to their values, obviously), we have to replace the
names of the vector with `cols`. Running this on every line of each
.jsonl file using `lapply()`, we get uniform vectors that we can
ultimately bind together with `do.call()`. The result is a single, large
matrix which we’ll be converting to a data frame and saving as a .csv
file. (Because a lot can go wrong between exporting and (re-)importing
.csv files of Twitter data, this script uses the `save_as_csv` function
from the `rtweet` package.)

We then loop this over all the files in a given folder, and a message
tells us when each file is done. Finally, `file.remove()` can be called
to permanently delete the .jsonl files if memory is an issue.

A good take-away from this code is the relative efficiency of the
`apply()` family, in comparison with a `for` loop. Benchmarking this
code versus a nested `for` loop showed the `lapply()` approach to be up
to 10 times faster, and with large files like these, that can make the
difference between 2 and 20 minutes per file!

For resources on why `for` loops are slow, try
[these](https://www.r-bloggers.com/why-loops-are-slow-in-r/){:target=“\_blank”}
[links](https://stackoverflow.com/questions/7142767/why-are-loops-slow-in-r){:target=“\_blank”},
and for a great resource on converting for loops to `apply()` functions,
look
[here](https://nicercode.github.io/guides/repeating-things/){:target=“\_blank”}.
