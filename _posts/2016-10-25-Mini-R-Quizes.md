---
layout: post
title: Mini R Quizzes
date: 2016-10-25 6:00:00
comments: true
published: true
author: Marcin Kosiński
categories: [Quizes]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/rquiz2.JPG" class="fit image"> Data Science is sometimes a stressful and responsible job... so we spontaneously organize mini R quizzes in our R team at the company!

## Chaining R packages

One of my favourite fast 5-minutes long R game is `chaining R packages`. One person says the name of an R package and the next person needs to say the name of an R package that starts on a letter on which the previous package has finished (and you can't use packages that were already mentioned by previous players): example


{% highlight r %}
dplyr -> ROCR -> rpart -> tm -> mstate -> e1071
{% endhighlight %}



{% highlight text %}
Error in eval(expr, envir, enclos): object 'dplyr' not found
{% endhighlight %}

`e1071` always finishes the game :) so instead of looking after the last character of a previous package name you can look after the last letter. Don't open browser with CRAN archives!

## Guess which R package you are

<img src="/images/fulls/rquiz3.jpg" class="left image"> In this game you would normally have yellow sticker card glued to your forehead with a name of a famous person. Asking questions that can be only asked `NO` or `YES` by the audience you need to figure out who you really are.. but in our version your are a famous R package or a famous software! Last time I've seen **Docker with paint, Windows 95, elasticsearch, crontab** and **dplyr** :)

## Mini R quiz

People keep asking questions about R and the person that is the closest to the answer wins a point. You can play till the 5th or 10th point - the winner might not pay for the beer if you organize an integration party next time. Even simple questions can not be easy to answer! In the last game I've asked

- What is the number of packages in which Hadley Wickham is the co-author? Guesses: 17, 30 and 40 - all wrong: r-pkgs states that it might be 93 http://r-pkg.org/search.html?q=hadley+wickham
- What is the longest function name from base R? The best guess: `suppressPackageStartupMessages()` with 30 characters in its name has finally lost with `getDLLRegisteredRoutines.character` (34 chars!)


{% highlight r %}
tail(sort(sapply(ls("package:base"), nchar)),6)
{% endhighlight %}



{% highlight text %}
       print.DLLRegisteredRoutines       as.character.numeric_version 
                                27                                 28 
     as.data.frame.numeric_version     suppressPackageStartupMessages 
                                29                                 30 
  getDLLRegisteredRoutines.DLLInfo getDLLRegisteredRoutines.character 
                                32                                 34 
{% endhighlight %}

<img src="https://media2.giphy.com/media/hkMXte9dBJFfO/200_s.gif" class="fit image">


If you have any great ideas on mini R games then please feel free to leave a comment below [on my blog](http://r-addict.com/2016/10/25/Mini-R-Quizes.html)
