---
layout: post
title:  "Automating IPA transcription grading in R"

categories:
  - Teaching
date:   2020-06-08
tags: phonetics R grading
---

Large group sizes in introductory phonetics classes may be discouraging if you want to assign longer IPA transcriptions. This R script automates that process by generating a list of acceptable model transcriptions (e.g., where variation is permitted) from your input, importing student transcriptions from a .txt file, finding the closest model transcription(s) based on Levenshtein Distance, calculating a grade based on the minimal difference and the maximal number of characters between the two (model and student transcription), and providing a .txt feedback file in each student's folder which can then be uploaded to your online learning environment. (Thanks to my colleague François Lareau who shared the original idea of using the Levenshtein Distance and for sharing his Python script. The largest personal addition here is generating a list of possible variants.)

Let's look at the script first (comments will follow), assuming a simple sample transcription \"I don't watch football\" with some room for variation. Namely, let's say we'll accept: 
- \[j\] or \[ɪ\] as the second member of certain diphthongs,
- \[tʃ\] (two characters), \[t͡ʃ\] (two characters with middle tie bar above) or \[ʧ\] (single character) for the affricate in \"watch\",
- \[t\] or \[ʔ\] at the end of \"don't\",
- \[p\] (for place assimilation), \[t\] or \[ʔ\] as the first stop in the cluster of \"football\", 
- \[ɑ\] or \[ɔ\] for the vowel in \"ball\" (o hai *caught-cot* merger), and
- \[l\] or \[ɫ\] at the end of \"football\".

{% highlight r %}
# Replace ... with main directory containing all student sub-folders
setwd("...")

# Targeted IPA transcription goes here. Accepted variants for a single position go in c("", "" ...), 
# everything else should intervene unbroken.
corrige.a = do.call(paste0, expand.grid("a",
  c("j", "ɪ"),
  "doʊn",
  c("t", "ʔ"),
  "wa",
  c("tʃ", "t͡ʃ", "ʧ"),
  "fʊ",
  c("p", "t", "ʔ"),
  "b",
  c("ɑ", "ɔ"),
  c("l", "ɫ")
))

# Get a list of all the files in your subdirectories matching a certain pattern 
# (here, all submissions are named "texteenligne.txt")
temp = list.files(pattern="texteenligne.txt", recursive=T)

# This is only for testing purposes and makes the rest of the script run on only the first 5 papers. 
# If you're sure it works and want to work on the whole group, just uncomment the line 
# following the next to redefine temp.lite as temp
temp.lite = temp[1:5]
# temp.lite = temp

# Make some dummy vectors that will be filled in by the for loop
result = vector("character", length(temp.lite))
transcr = vector("character", length(temp.lite))
dist.lev = vector("numeric", length(temp.lite))
max.char = vector("numeric", length(temp.lite))
cor.char = vector("numeric", length(temp.lite))
var = vector("character", length(temp.lite))

# The heavy legwork is here. For each file, it provides...
for (i in 1:length(temp.lite)) {
  # ... the filename. This is important for getting the student's name in the feedback file!
  name = temp.lite[i]
  result[i] = name
  # ... the cleaned-up transcription (sent to lower case and whitespaces removed). 
  # Note that it only takes the first line! This is because in my assignment, 
  # each line is a language and is being graded separately
  text =  gsub(" ", "", read.csv(temp.lite[i], header=F, stringsAsFactors = F, encoding = "UTF-8")[1,])
  transcr[i] = tolower(text)
  # ... the minimal Lev. distance between the student's transcription 
  # and all the permutations of the target transcription
  dist = min(adist(corrige.a,text))
  dist.lev[i] = dist
  # ... all the permutations matching that minimal distance
  corr = paste(corrige.a[adist(corrige.a, text)==min(adist(corrige.a, text))], collapse=" / ")
  var[i] = corr
  # ... the number of characters for whichever of the two texts is longest
  char = pmax(nchar(corrige.a[min(adist(corrige.a, text))]), nchar(text))
  max.char[i] = char
  # ... and the maximum number of characters of just the permutations matching that minimal Lev. distance
  char2 = max(nchar(corrige.a[adist(corrige.a, text)==min(adist(corrige.a, text))]))
  cor.char[i] = char2
}

# Make a dataframe out of it all
a = data.frame(result, transcr, var, dist.lev, cor.char, max.char)

# Calculate 2 grades: one that's more lenient (based on the absolute max no. character), 
# and one based on the max no. characters from the target permutations
a$len.grade = (max.char - dist.lev)/max.char
a$sev.grade = (cor.char - dist.lev)/cor.char

# Make a feedback sheet (here, named "retroaction.txt") for each student and place it in their subdirectory. 
# So far, it's just name, cleaned-up transcription, model transcriptions and percentage, 
# but that all can be manipulated in the paste command below.
for (i in 1:length(temp.lite)) {
  write.table(
    paste(
	  # It's assumed (because of Moodle) that the student's name begins the sub-folder name 
	  # and is followed by "_"
	  "Nom:", gsub("_.+", "", a[i,]$result),
	  "Transcription nettoyée:", a[i,]$transcr,
	  "Transcription modèle:", a[i,]$var,
	  "Score:", round(100*a[i,]$len.grade, 2),
	  sep="\n"), 
    paste0(
      gsub("texteenligne.txt", "", temp.lite[i]), "retroaction.txt"),
              quote=F, row.names=F, col.names = F, fileEncoding = "UTF-8")
}
{% endhighlight %}

Note that even with this small text, with all the variants accepted, we have 144 model transcriptions! Variables can, of course, be trimmed down with more specific instruction. (If absolutely necessary, you can randomly pick an *n*-sized sample of your model transcriptions.) In the retroaction file, you may wish to limit the model transcriptions to one or only a few (there can be many identified). You can print only the first one, for instance, in the feedback file by replacing `a[i,]$var` with `gsub("^(.+?) / .+", "\\1", a[i,]$var)`. Or you could replace the definition of `corr` above to `paste(head(corrige.a[adist(corrige.a, text)==min(adist(corrige.a, text))],3), collapse=" / ")` to give you, say, the first 3 matching models.

This script assumes a typical Moodle assignment structure, i.e., a folder with a sub-folder for each student (which is also named after each student) which includes their submission. Note that if students entered the text in an online form, rather than submitting a .txt file, you'll have to convert their submissions from .html to .txt.

Setting up the exercise takes some care. In particular, students have to:
- Use Unicode characters, such as copy-pasting from [this site](http://westonruter.github.io/ipa-chart/keyboard/){:target="_blank"}.
- Stick to a specific set or language-specific sets of simplified characters. (This is to avoid exponentially expanding the model transcriptions.)
- Avoid spaces, brackets, non-IPA capital letters, etc.
- If multiple languages or files are targeted, separate each transcription by a single line break.

Note that this sample script works on only the first line, i.e., transcription. You can, of course, define all your model transcriptions and expand the `for` loop (or make a separate, dedicated version for each transcription) for each line in the student file.

Failing to respect any one of these instructions can lead to a very low score. Some clean-up is done in the script, but it's impossible to predict every type of error, so papers with abnormally low scores should be dealt with individually. To give just one example, a handful of papers for one assignment had some trailing or \"invisible\" combining characters (like vowel nasalization) invisible to the naked eye; if memory serves, this can arise if the diacritic is added twice or more, or if at the beginning of a line, the diacritic is added before the character and then added again after the character.

The current script takes a somewhat \"brute strength\" approach to errors. That is, it doesn't distinguish the gravity or type of errors; it only gauges the similarity of the student text to the closest model text. I don't see this as a major problem for introductory levels, especially given the precautions detailed above. However, for intermediate and more advanced transcriptions, I plan on implementing a basic notion of featural similarity that can be context-dependent. For example, errors in place of articulation can be less serious if the major classes are correct, and could be even less in potential contexts for place assimilation. But that's a project for this summer!
