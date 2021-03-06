---
layout: post
title: Venn Diagram Comparison of Boruta, FSelectorRcpp and GLMnet Algorithms
comments: true
published: true
author: Marcin Kosiński
categories: [Feature Selection]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/venn.png" class="fit image"> Feature selection is a process of extracting valuable features that have significant influence on [dependent](https://nces.ed.gov/nceskids/help/user_guide/graph/variables.asp) variable. This is still an active field of research and machine wandering. In this post I compare few feature selection algorithms: traditional [GLM with regularization](https://cran.r-project.org/web/packages/glmnet/vignettes/glmnet_beta.html), computationally demanding [Boruta](https://m2.icm.edu.pl/boruta/) and entropy based filter from [FSelectorRcpp](http://github.com/mi2-warsaw/FSelectorRcpp) (free of Java/Weka) package. Check out the comparison on [Venn Diagram](https://cran.r-project.org/web/packages/VennDiagram/) carried out on data from the [RTCGA](http://r-addict.com/2016/05/04/RTCGA-Quick-Guide.html) factory of R data packages.

I would like to thank [Magda Sobiczewska](https://github.com/Magda13) and [ pbiecek](http://github.com/pbiecek/) for inspiration for this comparison. I have a chance to use Boruta nad FSelectorRcpp in action. GLMnet is here only to improve Venn Diagram.

# RTCGA data

Data used for this comparison come from RTCGA ([http://rtcga.github.io/RTCGA/](http://rtcga.github.io/RTCGA/)) and present genes' expressions (RNASeq) from human sequenced genome. Datasets with RNASeq are available via [RTCGA.rnaseq](https://bioconductor.org/packages/RTCGA.rnaseq/) data package and originally were provided by [The Cancer Genome Atlas](http://gdac.broadinstitute.org/). It's a great set of over 20 thousand of features (1 gene expression = 1 continuous feature) that might have influence on various aspects of human survival. Let's use data for Breast Cancer (**Breast invasive carcinoma** / BRCA) where we will try to find valuable genes that have impact on dependent variable denoting whether a sample of the collected readings came from tumor or normal, healthy tissue.


{% highlight r %}
## try http:// if https:// URLs are not supported
source("https://bioconductor.org/biocLite.R")
biocLite("RTCGA.rnaseq")
{% endhighlight %}


{% highlight r %}
library(RTCGA.rnaseq)
BRCA.rnaseq$bcr_patient_barcode <- 
   substr(BRCA.rnaseq$bcr_patient_barcode, 14, 14)
{% endhighlight %}

The dependent variable, `bcr_patient_barcode`, is the [TCGA barcode](https://wiki.nci.nih.gov/display/TCGA/TCGA+barcode) from which we receive information whether a sample of the collected readings came from tumor or normal, healthy tissue (14th character in the code).

Check another RTCGA use case: [TCGA and The Curse of BigData](http://rtcga.github.io/RTCGA/Usecases.html). 

# GLMnet

Logistic Regression, a model from generalized linear models (GLM) family, a first attempt model for class prediction, can be extended with regularization net to provide prediction and variables selection at the same time. We can assume that not valuable features will appear with equal to zero coefficient in the final model with best regularization parameter. Broader explanation can be found in the [vignette of the glmnet package](https://cran.r-project.org/web/packages/glmnet/vignettes/glmnet_beta.html#log). Below is the code I use to extract valuable features with the extra help of cross-validation and parallel computing.


{% highlight r %}
library(doMC)
registerDoMC(cores=6)
library(glmnet)
# fit the model
cv.glmnet(x = as.matrix(BRCA.rnaseq[, -1]),
          y = factor(BRCA.rnaseq[, 1]),
          family = "binomial", 
          type.measure = "class", 
          parallel = TRUE) -> cvfit
# extract feature names that have 
# non zero coefficiant
names(which(
   coef(cvfit, s = "lambda.min")[, 1] != 0)
   )[-1] -> glmnet.features
# first name is intercept
{% endhighlight %}

Function `coef` extracts coefficients for fitted model. Argument `s` specifies for which regularization parameter we would like to extract them - `lamba.min` is the parameter for  which miss-classification error is minimal. You may also try to use `lambda.1se`.


{% highlight r %}
plot(cvfit)
{% endhighlight %}

![plot of chunk unnamed-chunk-5](/figure/source/2016-06-19-Venn-Diagram-RTCGA-Feature-Selection/unnamed-chunk-5-1.png)

[Discussion about standardization for LASSO can be found here](http://stats.stackexchange.com/questions/86434/is-standardisation-before-lasso-really-necessary). I normally don't do this, since I work with streaming data, for which checking assumptions, model diagnostics and standardization is problematic and is still a rapid field of research.

# Boruta

The second feature selection algorithm, Boruta, is based on [ranger](https://cran.r-project.org/web/packages/ranger/index.html) - a fast implementation of random forests. 
At start this algorithm 
creates duplicated variables for each attribute in the model's formula. Duplicates
then have randomly ordered values. Algorithm takes into consideration the Z-statistic of each variable in each tree and checks
whether the boxplot of Z-statistics for a variable is higher than the highest boxplot for
randomly ordered duplicate. Boruta is more sophisticated and its 9 page explanation can be found in [Journal of Statistical Software](https://www.jstatsoft.org/article/view/v036i11/v36i11.pdf). Below I present code that extracts valuable and tentative attributes


{% highlight r %}
library(Boruta)
# fit features selection algorithm
system.time({
Boruta(as.factor(bcr_patient_barcode) ~.,
       data = BRCA.rnaseq) -> Boruta.model
}) # 20425 seconds ~ 35.5 min
# archive result
library(archivist)
saveToRepo(Boruta.model, 
           repoDir = "Museum")

# correct names
gsub(x = getSelectedAttributes(Boruta.model),
     pattern = "`",
     replacement = "",
     fixed = T) -> Boruta.features
{% endhighlight %}

There exist `plot()` overloaded function for `Boruta` class object which enables plotting boxplots of Z-statistics for each variable. It's a good way to visualize the selection process. In this case there are over 20 thousand of attributes so I am using `getSelectedAttributes` function to extract valuable features. You could also extract `tentative` variables with `withTentative = TRUE`.

In case you run `Boruta` code yourself and it takes too long or you get such an error


{% highlight r %}
Error: protect () : protection stack overflow 
{% endhighlight %}

I have archived my model on GitHub, and you can load it to R global environment with [archivist](http://r-addict.com/2016/06/13/RHero-Saves-Backup-City-With-archivist-github.html) package


{% highlight r %}
library(archivist)
aread("MarcinKosinski/Museum/d3732742b989ed49666b0472ba52d705") -> Boruta.model
{% endhighlight %}

The history of Boruta decisions of rejecting and accepting features can be seen on such a graph


{% highlight r %}
plotImpHistory(Boruta.model)
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figure/source/2016-06-19-Venn-Diagram-RTCGA-Feature-Selection/unnamed-chunk-9-1.png)


# FSelectorRcpp::information_gain

We have realized that there are many sophisticated and computationally absorbent feature selection algorithms. In many cases the time is a huge blocker and we can't wait for Boruta to finish its computing. For such cases easier algorithms are demanded. One of them is entropy based filter implemented in [FSelector](https://github.com/larskotthoff/fselector) package and its faster Rcpp (free of Java/Weka) reimplementation in [FSelectorRcpp](https://github.com/mi2-warsaw/FSelectorRcpp)  package (by [ zzawadz](https://github.com/zzawadz)) that also has support for sparse matrixes. If you would like to contribute to this package, please check our [issues list on GitHub](https://github.com/mi2-warsaw/FSelectorRcpp/issues).

*Information gain* is a simple, linear algorithm that computes the entropy of dependent and explanatory variables, and the conditional entropy of dependent variable with respect to each explanatory variable separately. This simple statistic enables to calculate the belief of the distribution of dependent variable when we only know the distribution of explanatory variable. Below code shows how easy it is to use


{% highlight r %}
devtools::install_github('mi2-warsaw/FSelectorRcpp')
library(FSelectorRcpp)
information_gain(y = BRCA.rnaseq[, 1],
  x = BRCA.rnaseq[, -1]) -> FSelectorRcpp.weights
library(FSelector)
cutoff.k.percent(FSelectorRcpp.weights, 
   k = 0.01) -> FSelectorRcpp.features
{% endhighlight %}

Information gain algorithm returns scores for attributes and we will cut top 1 % features. For that we will use simple `cutoff.k.percent` from `FSelector`.

# Venn Diagram

Finally we can visualize differences in decisions of algorithms. One way is a [Venn Diagram](https://en.wikipedia.org/wiki/Venn_diagram) that (after Wikipedia)

> shows all possible logical relations between a finite collection of different sets.



{% highlight r %}
named.list.for.venn <-
   list(` Boruta` = Boruta.features,
             `GLMnet     ` = glmnet.features,
      FSelectorRcpp = FSelectorRcpp.features)
{% endhighlight %}

{% highlight r %}
library(VennDiagram)
venn.diagram(
    x = named.list.for.venn,
    filename = "venn.png",
    imagetype = "png",
    #col = "transparent",,
    col = "black",
    lty = "dotted",
    lwd = 4,
    fill = c("cornflowerblue", "green", 
             "yellow"),
    alpha = 0.50,
    label.col = c("orange", "white",  
                  "orange", "white", 
      "darkblue", "white", "darkgreen"),
    cex = 2.5,
    fontfamily = "serif",
    fontface = "bold",
    cat.col = c("darkblue", "darkgreen", 
                "orange"),
    cat.cex = 2.5,
    cat.fontfamily = "serif"
    )
{% endhighlight %}

The effect of this code can be seen on the welcome graph of this post.
We can see that only 1 variable was chosen with all 3 algorithms. VennDiagram package allows to find out the names of intersections.


{% highlight r %}
get.venn.partitions(named.list.for.venn) -> 
   partitions
apply(partitions[, c(1,2,3)],
      MARGIN = 1,
      function(configuration) {
         all(configuration == rep(T,3))
      }) -> all.algorithms
partitions[all.algorithms, ]
{% endhighlight %}



{% highlight text %}
   Boruta GLMnet      FSelectorRcpp                           ..set..   ..values.. ..count..
1    TRUE        TRUE          TRUE  Boruta∩GLMnet     ∩FSelectorRcpp ABCA10|10349         1
{% endhighlight %}

Do you have any comments about the diagram or the unbelievable result that only 1 variable overlaps all 3 algorithms? Maybe some algorithms can be improved? Is this the case of random nature of `cv.glmnet` or `Boruta`? Feel free to leave your comments on the Disqus panel below.
