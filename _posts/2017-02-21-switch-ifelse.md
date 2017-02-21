---
layout: post
title: Use switch() instead of ifelse() to return a NULL
comments: true
published: true
date: 2017-02-21 11:00:00
author: Marcin Kosiński
categories: [Tips]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/statement_ifelse.png" class="fit image"> Have you ever tried to return a `NULL` with the `ifelse()` function? This function is a simple vectorized workflow for conditional statements. However, one can't just return a `NULL` value as a result of this evaluation. Check a tricky workaround solution in this post.

Imagine a simple `R` logical statement like


{% highlight r %}
statement <- length(character()) > 0
statement
{% endhighlight %}



{% highlight text %}
[1] FALSE
{% endhighlight %}

and depending on that logical value you would like to create a new variable called `res`. You can follow a regular `if` conditional statement


{% highlight r %}
if (statement) {
   res <- "message"
} else {
   res <- NULL
}
res
{% endhighlight %}



{% highlight text %}
NULL
{% endhighlight %}

or use simplified interface with `ifelse()` function


{% highlight r %}
ifelse(statement, "message", NULL)
{% endhighlight %}



{% highlight text %}
Error in ifelse(statement, "message", NULL): replacement has length zero
{% endhighlight %}

However it looks like one is not able to return the `NULL` as a result of this operation. 

A solution is to jump to the other conditional function called `switch()`, which for the first parameter takes number (`n`) and returns the value passed as the `n-th` parameter. If one treats logical values as `TRUE is 1` and `FALSE is 0` then primary `ifelse()` statement can be rebuild to `switch()` call like


{% highlight r %}
switch(statement + 1, NULL, "message")
{% endhighlight %}



{% highlight text %}
NULL
{% endhighlight %}

What do you think about such workaround? Do you use other solutions for such a situation?
