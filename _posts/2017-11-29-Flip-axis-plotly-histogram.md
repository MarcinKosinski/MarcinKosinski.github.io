---
layout: post
title: Flip axes in plotly histograms - few approaches
comments: true
published: true
date: 2017-11-29 8:00:00
author: Marcin Kosiński
categories: [Tips]
output:
  html_document:
    keep_md: true
---



<img src="/images/fulls/plotly1.jpg" class="fit image"> This seems easy as it requires just to change `x` parameter to `y` in the plot specification. Well, there are some edge cases in which R users might get in trouble!


{% highlight r %}
library(plotly)
packageVersion('plotly')
{% endhighlight %}



{% highlight text %}
[1] '4.7.1'
{% endhighlight %}

> Before you go, let me just explain myself. I have just started learning R interface to `plotly` library and I am really amazed by the effort people done to make those wires connected and possible to be used for a broad audience.


{% highlight r %}
set.seed(2); values <- rnorm(50)
p1 <- plot_ly(x = values, type = "histogram", name = "default")
p2 <- plot_ly(y = values, type = "histogram", name = "flipped")
subplot(p1, p2)
{% endhighlight %}

What if the plotly object specification is longer than several dozen of lines and one would like to have this feature parametrized in a function's call? Like in the shiny application, to have the flip as an option?

The quickest solution is to provide an `if` statement like


{% highlight r %}
#' @param values Numeric values to be used in the plot.
#' @param flip Logical: should the plot be a horizontal histogram?
to_flip_or_not_to_flip <- function(values, flip = FALSE){
   if (flip) {
      p <- plot_ly(y = values, type = "histogram", name = "flipped")
   } else {
      p <- plot_ly(x = values, type = "histogram", name = "default")
   }
   p
}
#' @examples
#' to_flip_or_not_to_flip(p)
#' to_flip_or_not_to_flip(p, flip = TRUE)
{% endhighlight %}


This is a typical redundancy of the code. Imagine the plot being created in a loop with many extra layers, where in the end the specification is longer than 50 lines. Would you copy and paste all 50 lines just to substitute `x` with `y`?

# orientation parameter

Maybe `orientation` parameter is an option? After the reference: https://plot.ly/r/reference/#histogram

<img src="/images/fulls/plotly3.jpg" class="fit image">


{% highlight r %}
p3 <- plot_ly(x = values, type = "histogram", name = "orient v", orientation = "v")
p4 <- plot_ly(x = values, type = "histogram", name = "orient h", orientation = "h")
subplot(p3, p4)
{% endhighlight %}

one get a wrong plot. `values` should be assigned to `y` parameter again.

<img src="/images/fulls/plotly2.jpg" class="fit image">

Of course there is a [plotly guide book](https://plotly-book.cpsievert.me/) for R users (where I've learned `subplot()`) but one is not going to read the whole documentation just to create a simple horizontal histogram (I suppose).


# The solution?

One can create the list with all possible parameters that he/she would like to specify (besides default parameters) and then
change the name of `x` parameter to `y` in that list depending on flip requirements?


{% highlight r %}
parameters <- list(
   x = values,
   type = "histogram",
   name = "x"
)
p5 <- do.call(plot_ly, parameters)

# if (flip) {
   names(parameters)[1] <- "y"
   parameters$name <- "y"
# }
p6 <- do.call(plot_ly, parameters)
subplot(p5, p6)
{% endhighlight %}

<img src="/images/fulls/plotly4.jpg" class="fit image">

# My personal best?

Change the object directly after you have specified the plot.
One can easily guess what needs to be changed after looking to `str(plot)` call.
We would change data attributes and will rename `x` object to `y`. See that we can also modify values, not only names of parameters.


{% highlight r %}
p7 <- p5
str(p7$x$attrs)
# if (flip) {
   names(p7$x$attrs[[1]])[1] <- "y"
   p7$x$attrs[[1]]$name <- "y - change object directly"
# }
subplot(p5, p7)
{% endhighlight %}

<img src="/images/fulls/plotly5.jpg" class="fit image">

# Other solutions?

Do you know cleaner approach? Please don't hesitate to leave comments [at mine blog](http://r-addict.com/2017/11/29/Flip-axis-plotly-histogram.html).

I suppose one could create a plot in `ggplot2` and then apply `ggplotly()` function but I am not convinced this function translates all possible further plots to the client ready format, so I'd like to stick to `plotly` interface.

> Note: sorry for print screens. I could not get interactive results for plotly in the Rmarkdown document compiled with a [jekyll and knitr](https://jekyll.yihui.name/2014/09/jekyll-with-knitr.html).
