---
layout: post
title: Comparing (Fancy) Survival Curves with Weighted Log-rank Tests
comments: true
published: true
date: 2017-02-09 11:00:00
author: Marcin Kosiński
categories: [Biostatistics]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/fancy_survminer.png" class="fit image"> We have just adopted weighted Log-rank tests to the [survminer](http://www.sthda.com/english/rpkgs/survminer/) package, thanks to [survMisc::comp](https://cran.r-project.org/web/packages/survMisc/index.html). What are they and why they are useful? Read this blog post to find out. I used [ggthemr](https://github.com/cttobin/ggthemr) to make the presentation a little bit more bizarre.

* Log-rank statistic for 2 groups
* Weighted Log-rank extensions
*
{:toc}

# Log-rank statistic for 2 groups

Log-rank test, based on Log-rank statistic, is a popular tool that determines whether 2 (or more) estimates of survival curves differ significantly. As it is stated in the literature, the Log-rank test for comparing survival (estimates of survival curves) in 2 groups ($$A$$ and $$B$$) is based on the below statistic

$$LR = \frac{U^2}{V} \sim \chi(1),$$

where 

$$U = \sum_{i=1}^{T}w_{t_i}(o_{t_i}^A-e_{t_i}^A), \ \ \ \ \ \ \ \ V = Var(U) = \sum_{i=1}^{T}(w_{t_i}^2\frac{n_{t_i}^An_{t_i}^Bd_i(n_{t_i}-o_{t_i})}{n_{t_i}^2(n_{t_i}-1)})$$

and 

- $$t_i$$ for $$i=1, \dots, T$$ are possible event times, 
- $$n_{t_i}$$ is the overall risk set size on the time $$t_i$$ ($$n_{t_i} = n_{t_i}^A+n_{t_i}^B$$),
- $$n_{t_i}^A$$ is the risk set size on the time $$t_i$$ in group $$A$$,
- $$n_{t_i}^B$$ is the risk set size on the time $$t_i$$ in group $$B$$,
- $$o_{t_i}$$ overall observed events in the time $$t_i$$ ($$o_{t_i} = o_{t_i}^A+o_{t_i}^B$$),
- $$o_{t_i}^A$$ observed events in the time $$t_i$$ in group $$A$$,
- $$o_{t_i}^B$$ observed events in the time $$t_i$$ in group $$B$$,
- $$e_{t_i}$$ number of overall expected events in the time $$t_i$$ ($$e_{t_i} = e_{t_i}^A+e_{t_i}^B$$),
- $$e_{t_i}^A$$ number of expected events in the time $$t_i$$ in group $$A$$,
- $$e_{t_i}^B$$ number of expected events in the time $$t_i$$ in group $$B$$,
- $$w_{t_i}$$ is a weight for the statistic,

also remember about few notes

$$e_{t_i}^A = n_{t_i}^A \frac{o_{t_i}}{n_{t_i}}, \ \ \ \ \ \ \ \ \ \  e_{t_i}^B = n_{t_i}^B \frac{o_{t_i}}{n_{t_i}},$$

$$e_{t_i}^A + e_{t_i}^B = o_{t_i}^A + o_{t_i}^B$$

that's why we can substitute group $$A$$ with $$B$$ in $$U$$ and receive same results.

# Weighted Log-rank extensions

Regular Log-rank comparison uses $$w_{t_i} = 1$$ but many modifications to that approach have been proposed. The most popular modifications, called weighted Log-rank tests, are available in `?survMisc::comp`

- `n` Gehan and Breslow proposed to use $$w_{t_i} = n_{t_i}$$ (this is also called generalized Wilcoxon),
- `srqtN` Tharone and Ware proposed to use $$w_{t_i} = \sqrt{n_{t_i}}$$,
- `S1` Peto-Peto's modified survival estimate $$w_{t_i} = S1({t_i}) = \prod_{i=1}^{T}(\frac{1-e_{t_i}}{n_{t_i}+1})$$,
- `S2` modified Peto-Peto (by Andersen) $$w_{t_i} = S2({t_i}) = \frac{S1({t_i})n_{t_i}}{n_{t_i}+1}$$,
- `FH` Fleming-Harrington $$w_{t_i} = S(t_i)^p(1 - S(t_i))^q$$.

> Watch out for `FH` as I [submitted an info on survMisc repository](https://github.com/dardisco/survMisc/issues/15) where I think their mathematical notation is misleading for Fleming-Harrington.

## Why are they useful?

The regular Log-rank test is sensitive to detect differences in late survival times, where Gehan-Breslow and Tharone-Ware propositions might be used if one is interested in early differences in survival times. Peto-Peto modifications are also useful in early differences and are more robust (than Tharone-Whare or Gehan-Breslow) for situations where many observations are censored. The most flexible is Fleming-Harrington method for weights, where high `p` indicates detecting early differences and high `q` indicates detecting differences in late survival times. But there is always an issue on how to detect `p` and `q`.

> Remember that test selection should be performed at the research design level! Not after looking in the dataset.

# Plots 


{% highlight r %}
library(survminer)
library(survival)
data("kidney", package="KMsurv")
fit <- survfit(Surv(time=time, event=delta) ~ type, data=kidney)
{% endhighlight %}

After preparing a functionality for this GitHub's issue [Other tests than log-rank for testing survival curves](https://github.com/kassambara/survminer/issues/17) we are now able to compute p-values for various Log-rank tests in survminer package. Let as see below examples on executing all possible tests.

### gghtemr

Let's make it more interesting (or not) with [ggthemr](https://github.com/cttobin/ggthemr) package that has many predefinied palettes.

After installation

{% highlight r %}
devtools::install_github('cttobin/ggthemr')
{% endhighlight %}
one can set up a global ggplot2 palette/theme with 


{% highlight r %}
library(ggthemr)
ggthemr('dust')
{% endhighlight %}
and check current colors with 

{% highlight r %}
swatch()
{% endhighlight %}



{% highlight text %}
[1] "#555555" "#db735c" "#EFA86E" "#9A8A76" "#F3C57B" "#7A6752" "#2A91A2" "#87F28A" "#6EDCEF"
attr(,"class")
[1] "ggthemr_swatch"
{% endhighlight %}

> Note: the first colour in a swatch is a special one. It is reserved for outlining boxplots, text etc. For color lines first color is not used.

## Log-rank (survdiff) + sea theme


{% highlight r %}
ggthemr("sea") # set ggthemr theme

ggsurvplot(
   fit, # fitted survfit object
   risk.table  = TRUE, # include risk table?
   conf.int    = TRUE, # add confidence intervals?
   pval        = TRUE, # add p-value to the plot?
   pval.method = TRUE, # write the name of the test  
                       # that was used compute the p-value?
   pval.method.coord = c(3, 0.1), # coordinates for the name
   pval.method.size = 4,          # size for the name of the test
   log.rank.weights = "survdiff", # type of weights in log-rank test
   
   ### few options are set by defualt in survminer
   ### we will need to turn them off to allow
   ### ggthemr to work in his full glory
   palette = swatch()[2:3],  # pass the active palette
   ggtheme      = NULL, # disable adding custom survminer theme
   font.x       = NULL, # disable adding custom survminer font for the x axis
   font.y       = NULL, # disable adding custom survminer font for the y axis
   font.main    = NULL, # disable adding custom survminer font for the title
   font.submain = NULL, # disable adding custom survminer font for the subtitle
   font.caption = NULL  # disable adding custom survminer font for the caption
)
{% endhighlight %}

![plot of chunk unnamed-chunk-6](/figure/source/2017-02-09-Fancy-Survival-Plots/unnamed-chunk-6-1.png)

## Log-rank (comp) + dust theme


{% highlight r %}
ggthemr("dust") # set ggthemr theme

ggsurvplot(
   fit, # fitted survfit object
   risk.table  = TRUE, # include risk table?
   conf.int    = TRUE, # add confidence intervals?
   pval        = TRUE, # add p-value to the plot?
   pval.method = TRUE, # write the name of the test  
                       # that was used compute the p-value?
   pval.method.coord = c(3, 0.1), # coordinates for the name
   pval.method.size = 4,          # size for the name of the test
   log.rank.weights = "1", # type of weights in log-rank test
   
   ### few options are set by defualt in survminer
   ### we will need to turn them off to allow
   ### ggthemr to work in his full glory
   palette = swatch()[2:3],  # pass the active palette
   ggtheme      = NULL, # disable adding custom survminer theme
   font.x       = NULL, # disable adding custom survminer font for the x axis
   font.y       = NULL, # disable adding custom survminer font for the y axis
   font.main    = NULL, # disable adding custom survminer font for the title
   font.submain = NULL, # disable adding custom survminer font for the subtitle
   font.caption = NULL  # disable adding custom survminer font for the caption
)
{% endhighlight %}

![plot of chunk unnamed-chunk-7](/figure/source/2017-02-09-Fancy-Survival-Plots/unnamed-chunk-7-1.png)

## Gehan-Breslow (generalized Wilcoxon) + flat dark theme


{% highlight r %}
ggthemr("flat dark") # set ggthemr theme

ggsurvplot(
   fit, # fitted survfit object
   risk.table  = TRUE, # include risk table?
   conf.int    = TRUE, # add confidence intervals?
   pval        = TRUE, # add p-value to the plot?
   pval.method = TRUE, # write the name of the test  
                       # that was used compute the p-value?
   pval.method.coord = c(5, 0.1), # coordinates for the name
   pval.method.size = 4,          # size for the name of the test
   log.rank.weights = "n", # type of weights in log-rank test
   
   ### few options are set by defualt in survminer
   ### we will need to turn them off to allow
   ### ggthemr to work in his full glory
   palette = swatch()[2:3],  # pass the active palette
   ggtheme      = NULL, # disable adding custom survminer theme
   font.x       = NULL, # disable adding custom survminer font for the x axis
   font.y       = NULL, # disable adding custom survminer font for the y axis
   font.main    = NULL, # disable adding custom survminer font for the title
   font.submain = NULL, # disable adding custom survminer font for the subtitle
   font.caption = NULL  # disable adding custom survminer font for the caption
)
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/figure/source/2017-02-09-Fancy-Survival-Plots/unnamed-chunk-8-1.png)

## Tharone-Ware + camoflauge


{% highlight r %}
ggthemr("camoflauge") # set ggthemr theme

ggsurvplot(
   fit, # fitted survfit object
   risk.table  = TRUE, # include risk table?
   conf.int    = TRUE, # add confidence intervals?
   pval        = TRUE, # add p-value to the plot?
   pval.method = TRUE, # write the name of the test  
                       # that was used compute the p-value?
   pval.method.coord = c(3, 0.1), # coordinates for the name
   pval.method.size = 4,          # size for the name of the test
   log.rank.weights = "sqrtN", # type of weights in log-rank test
   
   ### few options are set by defualt in survminer
   ### we will need to turn them off to allow
   ### ggthemr to work in his full glory
   palette = swatch()[2:3],  # pass the active palette
   ggtheme      = NULL, # disable adding custom survminer theme
   font.x       = NULL, # disable adding custom survminer font for the x axis
   font.y       = NULL, # disable adding custom survminer font for the y axis
   font.main    = NULL, # disable adding custom survminer font for the title
   font.submain = NULL, # disable adding custom survminer font for the subtitle
   font.caption = NULL  # disable adding custom survminer font for the caption
)
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figure/source/2017-02-09-Fancy-Survival-Plots/unnamed-chunk-9-1.png)

## Peto-Peto's modified survival estimate + fresh theme


{% highlight r %}
ggthemr("fresh") # set ggthemr theme

ggsurvplot(
   fit, # fitted survfit object
   risk.table  = TRUE, # include risk table?
   conf.int    = TRUE, # add confidence intervals?
   pval        = TRUE, # add p-value to the plot?
   pval.method = TRUE, # write the name of the test  
                       # that was used compute the p-value?
   pval.method.coord = c(5, 0.1), # coordinates for the name
   pval.method.size = 4,          # size for the name of the test
   log.rank.weights = "S1", # type of weights in log-rank test
   
   ### few options are set by defualt in survminer
   ### we will need to turn them off to allow
   ### ggthemr to work in his full glory
   palette = swatch()[2:3],  # pass the active palette
   ggtheme      = NULL, # disable adding custom survminer theme
   font.x       = NULL, # disable adding custom survminer font for the x axis
   font.y       = NULL, # disable adding custom survminer font for the y axis
   font.main    = NULL, # disable adding custom survminer font for the title
   font.submain = NULL, # disable adding custom survminer font for the subtitle
   font.caption = NULL  # disable adding custom survminer font for the caption
)
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/figure/source/2017-02-09-Fancy-Survival-Plots/unnamed-chunk-10-1.png)

## modified Peto-Peto's (by Andersen) + grass theme


{% highlight r %}
ggthemr("grass") # set ggthemr theme

ggsurvplot(
   fit, # fitted survfit object
   risk.table  = TRUE, # include risk table?
   conf.int    = TRUE, # add confidence intervals?
   pval        = TRUE, # add p-value to the plot?
   pval.method = TRUE, # write the name of the test  
                       # that was used compute the p-value?
   pval.method.coord = c(5, 0.1), # coordinates for the name
   pval.method.size = 4,          # size for the name of the test
   log.rank.weights = "S2", # type of weights in log-rank test
   
   ### few options are set by defualt in survminer
   ### we will need to turn them off to allow
   ### ggthemr to work in his full glory
   palette = swatch()[2:3],  # pass the active palette
   ggtheme      = NULL, # disable adding custom survminer theme
   font.x       = NULL, # disable adding custom survminer font for the x axis
   font.y       = NULL, # disable adding custom survminer font for the y axis
   font.main    = NULL, # disable adding custom survminer font for the title
   font.submain = NULL, # disable adding custom survminer font for the subtitle
   font.caption = NULL  # disable adding custom survminer font for the caption
)
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/figure/source/2017-02-09-Fancy-Survival-Plots/unnamed-chunk-11-1.png)

## Fleming-Harrington (p=1, q=1) + light theme


{% highlight r %}
ggthemr("light") # set ggthemr theme

ggsurvplot(
   fit, # fitted survfit object
   risk.table  = TRUE, # include risk table?
   conf.int    = TRUE, # add confidence intervals?
   pval        = TRUE, # add p-value to the plot?
   pval.method = TRUE, # write the name of the test  
                       # that was used compute the p-value?
   pval.method.coord = c(5, 0.1), # coordinates for the name
   pval.method.size = 4,          # size for the name of the test
   log.rank.weights = "FH_p=1_q=1", # type of weights in log-rank test
   
   ### few options are set by defualt in survminer
   ### we will need to turn them off to allow
   ### ggthemr to work in his full glory
   palette = swatch()[2:3],  # pass the active palette
   ggtheme      = NULL, # disable adding custom survminer theme
   font.x       = NULL, # disable adding custom survminer font for the x axis
   font.y       = NULL, # disable adding custom survminer font for the y axis
   font.main    = NULL, # disable adding custom survminer font for the title
   font.submain = NULL, # disable adding custom survminer font for the subtitle
   font.caption = NULL  # disable adding custom survminer font for the caption
)
{% endhighlight %}

![plot of chunk unnamed-chunk-12](/figure/source/2017-02-09-Fancy-Survival-Plots/unnamed-chunk-12-1.png)


# References

- Gehan A. A Generalized Wilcoxon Test for Comparing Arbitrarily Singly-Censored Samples. Biometrika 1965 Jun. 52(1/2):203-23. [JSTOR](www.jstor.org/stable/2333825)

- Tarone RE, Ware J 1977 On Distribution-Free Tests for Equality of Survival Distributions. Biometrika;64(1):156-60. [JSTOR](http://www.jstor.org/stable/2335790)

- Peto R, Peto J 1972 Asymptotically Efficient Rank Invariant Test Procedures. J Royal Statistical Society 135(2):186-207. [JSTOR](http://www.jstor.org/stable/2344317)

- Fleming TR, Harrington DP, O'Sullivan M 1987 Supremum Versions of the Log-Rank and Generalized Wilcoxon Statistics. J American Statistical Association 82(397):312-20. [JSTOR](http://www.jstor.org/stable/2289169)

- Billingsly P 1999 Convergence of Probability Measures. New York: John Wiley & Sons. [Wiley (paywall)](http://dx.doi.org/10.1002/9780470316962)

