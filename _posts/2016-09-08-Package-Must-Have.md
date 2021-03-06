---
layout: post
title: What Every R Package Must (REALLY) Contain? An Example on the eRum2016 Package
comments: true
published: true
author: Marcin Kosiński
categories: [Meetups and conferences, Development]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/what.jpg" class="fit image"> The R package development is a complex process of creating (mostly) a useful software, that will (probably) be used by other users. This means the provided tool should be resistant, immune, well tested and properly documented. Developers from many different languages have invented various approaches to improve software development, creating documentation or package testing. R users have adapted few of them and mostly we use [travis](https://travis-ci.org/) for continuous integration, [roxygen2](https://cran.r-project.org/web/packages/roxygen2/index.html) for documentation, [devtools](https://github.com/hadley/devtools) for testing and [knitr](http://yihui.name/knitr/) / [rmarkdown](http://rmarkdown.rstudio.com/) for writing manuals, tutorials, vignettes and package websites. This software development kit causes that the R package structure is rather broad, especially since many of us (R developers) puts source code from different languages into the package root to speed up the performance of created tools. Moreover we built our software libraries that are based on other packages, which complicates the NAMESPACE of the prepared package and forces the understanding of difference between dependent, imported and suggested packages. In this whole ecosystem of development equipment and requirements for proper package structure I've been asked **What Every R Package Must Contain?** You wouldn't guess how easy was the answer.

# DESCRIPTION file with 7 filled fields

On my last job interview on the R developer position the recruiting manager asked me **What every R package must contain?** Probably it was a trigger to conversation about the best practices in the R package development process, including roxygen2 or devtools. Because I was tired of elementary questions I have responded nonstandardly - **A DESCRIPTION file, and that's it**.

> In fact you only need a DESCRIPTION file with 7 filled fields to have a package that passes CRAN CHECK! 

I was amazed how we have developed and complicated the process of extending base R from this simple statement. 

Below you can see the `eRum2016` package that only have a DESCRIPTION file with such fields (URL is the extra, unneded field) [passes checks on Travis platform](https://travis-ci.org/eRum2016/eRum2016). 

By default, on Travis the `R CMD check --as-cran` is perfomed.


{% highlight r %}
cat(readLines('https://raw.githubusercontent.com/eRum2016/eRum2016/master/DESCRIPTION'),
    sep = '\n')
{% endhighlight %}



{% highlight text %}
Package: eRum2016
Title: European R Users Meeting
Version: 0.0.2016
Author: European R Community
Maintainer: European R Community <erum@konf.ue.poznan.pl>
Description: European R users meeting (eRum) is an international conference that aims at integrating users of the R language.
    eRum 2016 will be a good chance to exchange experiences, broaden knowledge on R and collaborate.
License: GPL-2
URL: http://erum.ue.poznan.pl/
{% endhighlight %}


This package can be even installed from GitHub and the index of this package can be opened to present the information included in the DESCRIPTION file


{% highlight r %}
devtools::install_github('eRum2016/eRum2016')
library(eRum2016)
help(package = "eRum2016")
{% endhighlight %}

This package present the key information for European R Users Meeting that will be held September in Poznań, Poland. More information about this converence can be checked on the conference website [http://erum.ue.poznan.pl/](http://erum.ue.poznan.pl/).
