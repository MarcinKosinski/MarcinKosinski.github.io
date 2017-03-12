---
layout: post
title: FSelectorRcpp on CRAN
comments: true
published: false
author: Marcin Kosiński
date: 2017-03-14 8:00:00
categories: [Feature Selection]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/fire.jpg" class="fit image"> [FSelectorRcpp](http://mi2-warsaw.github.io/FSelectorRcpp/) - Rcpp (free of Java/Weka) implementation of [FSelector](https://cran.r-project.org/web/packages/FSelector/index.html) entropy-based feature selection algorithms with a sparse matrix support, has finally arrived on [CRAN](https://cran.rstudio.com/web/packages/FSelectorRcpp/index.html) after [a year of development](https://github.com/mi2-warsaw/FSelectorRcpp). It is also equipped with a parallel backend.

Big thanks to the main architect: [Zygmunt Zawadzki, zstat](https://github.com/zzawadz), and our reviewer: [Krzysztof Słomczyński](https://github.com/krzyslom). 

If something is missing or not clear - please chat with us on our [slack](https://fselectorrcpp.slack.com/messages/general/)? 

### Get started: [Motivation, Installation and Quick Workflow](http://mi2-warsaw.github.io/FSelectorRcpp/articles/get_started.html)

### [Provided functionalities](http://mi2-warsaw.github.io/FSelectorRcpp/reference/)

### Blog posts history with use cases

- [Entropy Based Image Binarization with imager and FSelectorRcpp, Marcin Kosiński](http://r-addict.com/2017/01/08/Entropy-Based-Image-Binarization.html)
- [Venn Diagram Comparison of Boruta, FSelectorRcpp and GLMnet Algorithms, Marcin Kosiński](http://www.r-bloggers.com/venn-diagram-comparison-of-boruta-fselectorrcpp-and-glmnet-algorithms/)


### [Quick Workflow](http://mi2-warsaw.github.io/FSelectorRcpp/articles/get_started.html#quick-workflow)

A simple entropy based feature selection workflow. **Information gain** is an easy, linear algorithm that computes the entropy of a dependent and explanatory variables, and the conditional entropy of a dependent variable with a respect to each explanatory variable separately. This simple statistic enables to calculate the belief of the distribution of a dependent variable when we only know the distribution of a explanatory variable.

<p>
   <script src="https://gist.github.com/MarcinKosinski/000cd586b9c610ecd6247f5551b46663.js"></script>
  <noscript>
    <pre>
       {% highlight r %}
# install.packages(c('magrittr', 'FSelectorRcpp'))
library(magrittr)
library(FSelectorRcpp)       
information_gain(               # Calculate the score for each attribute
    formula = Species ~ .,      # that is on the right side of the formula.
    data = iris,                # Attributes must exist in the passed data.
    type  = "infogain",         # Choose the type of a score to be calculated.
    threads = 2                 # Set number of threads in a parallel backend.
  ) %>%                          
  cut_attrs(                    # Then take attributes with the highest rank.
    k = 2                       # For example: 2 attrs with the higehst rank.
  ) %>%                         
  to_formula(                   # Create a new formula object with 
    attrs = .,                  # the most influencial attrs.
    class = "Species"           
  ) %>%
  glm(
    formula = .,                # Use that formula in any classification algorithm.
    data = iris,                
    family = "binomial"         
)
       {% endhighlight %}
    </pre>
  </noscript>
</p>


![Orly cover](https://raw.githubusercontent.com/mi2-warsaw/FSelectorRcpp/master/o_rly.png)

# Acknowledgements

The cover photo of this blog posts comes from https://newevolutiondesigns.com/20-fire-art-wallpapers
