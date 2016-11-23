---
layout: post
title: Determine optimal cutpoints for numerical variables in survival plots
date: 2016-11-21 11:00:00
comments: true
published: true
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

<img src="/images/fulls/maxstat.png" class="fit image"> The often demand in the biostatistical research is to group patients depending on explanatory variables that are continuous. In some cases the requirement is to test overall survival of the subjects that suffer on a mutation in specific gene and have high expression (over expression) in other given gene. To visualize differences in the Kaplan-Meier estimates of survival curves between groups, first the discretization of continuous variable is performed. Problems caused by categorization of continuous variables are known and widely spread ([Harrel, 2015](http://biostat.mc.vanderbilt.edu/wiki/Main/CatContinuous)), but in this case there appear a simplification requirement for the discretization. In this post I present the [maxstat(maximally selected rank statistics)](https://cran.r-project.org/web/packages/maxstat/index.html) statistic to determine the optimal cutpoint for continuous variables, which was provided it in the [survminer](https://cran.rstudio.com/web/packages/survminer/index.html) package by Alboukadel Kassambara [kassambara](https://github.com/kassambara). 


<!-- Place this tag in your head or just before your close body tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>
<!-- Place this tag where you want the button to render. -->

In this post I will use data from [TCGA](https://cancergenome.nih.gov/abouttcga) study, that are provided in the [RTCGA](https://bioconductor.org/packages/RTCGA/) package <a aria-label="Star RTCGA/RTCGA on GitHub" data-count-aria-label="# stargazers on GitHub" data-count-api="/repos/RTCGA/RTCGA#stargazers_count" data-count-href="/RTCGA/RTCGA/stargazers" data-style="mega" data-icon="octicon-star" href="https://github.com/RTCGA/RTCGA" class="github-button">Star</a> and [survminer](https://github.com/kassambara/survminer/) <a aria-label="Star kassambara/survminer on GitHub" data-count-aria-label="# stargazers on GitHub" data-count-api="/repos/kassambara/survminer#stargazers_count" data-count-href="/kassambara/survminer/stargazers" data-style="mega" data-icon="octicon-star" href="https://github.com/kassambara/survminer" class="github-button">Star</a> package to determine the optimal cutpoint for continuous variable.

* Data preparation
* maxstat - maximally selected rank statistics
{:toc}

# Data preparation

I wrote about TCGA datasets and their preprocessing in my earlier posts: [RTCGA factory of R packages - Quick Guide](http://r-addict.com/2016/05/04/RTCGA-Quick-Guide.html) and [BioC 2016 Conference Overview and Few Ways of Downloading TCGA Data](http://r-addict.com/2016/07/22/BioC2016-RTCGA.html). If your are not familiar with RTCGA family of data packages, you can visit the [RTCGA website](http://rtcga.github.io/RTCGA/). Below I join survival information with `ABCD4|5826` gene expression for patients suffering from BRCA (breast cancer) and HNSC (head and neck cancer). It can be done due to `bcr_patient_barcode` column which identifies each patient.


{% highlight r %}
library(tidyverse) # pipes (%>%) and dplyr data munging
# installation
# source("https://bioconductor.org/biocLite.R")
# biocLite("RTCGA.clinical")
# biocLite("RTCGA.rnaseq")
library(RTCGA.clinical) # survival times
library(RTCGA.rnaseq) # genes' expression


survivalTCGA(BRCA.clinical, 
             HNSC.clinical) -> BRCA_HNSC.surv

expressionsTCGA(
   BRCA.rnaseq, HNSC.rnaseq,
   extract.cols = c("ABCD4|5826")) %>%
   rename(cohort = dataset,
          ABCD4 = `ABCD4|5826`) %>%
   filter(substr(bcr_patient_barcode, 14, 15) == "01") %>% 
   # only cancer samples
   mutate(bcr_patient_barcode = 
             substr(bcr_patient_barcode, 1, 12)) -> BRCA_HNSC.rnaseq

head(BRCA_HNSC.surv)
{% endhighlight %}



{% highlight text %}
  times bcr_patient_barcode patient.vital_status
1  3767        TCGA-3C-AAAU                    0
2  3801        TCGA-3C-AALI                    0
3  1228        TCGA-3C-AALJ                    0
4  1217        TCGA-3C-AALK                    0
5   158        TCGA-4H-AAAK                    0
6  1477        TCGA-5L-AAT0                    0
{% endhighlight %}



{% highlight r %}
head(BRCA_HNSC.rnaseq)
{% endhighlight %}



{% highlight text %}
# A tibble: 6 × 3
  bcr_patient_barcode      cohort    ABCD4
                <chr>       <chr>    <dbl>
1        TCGA-3C-AAAU BRCA.rnaseq 322.2560
2        TCGA-3C-AALI BRCA.rnaseq 486.6775
3        TCGA-3C-AALJ BRCA.rnaseq 308.2502
4        TCGA-3C-AALK BRCA.rnaseq 589.1601
5        TCGA-4H-AAAK BRCA.rnaseq 615.7447
6        TCGA-5L-AAT0 BRCA.rnaseq 896.3191
{% endhighlight %}

Joining survival times and `ABCD4` gene' expression.


{% highlight r %}
BRCA_HNSC.surv %>%
   left_join(BRCA_HNSC.rnaseq,
             by = "bcr_patient_barcode") ->
   BRCA_HNSC.surv_rnaseq

head(BRCA_HNSC.surv_rnaseq)
{% endhighlight %}



{% highlight text %}
  times bcr_patient_barcode patient.vital_status      cohort    ABCD4
1  3767        TCGA-3C-AAAU                    0 BRCA.rnaseq 322.2560
2  3801        TCGA-3C-AALI                    0 BRCA.rnaseq 486.6775
3  1228        TCGA-3C-AALJ                    0 BRCA.rnaseq 308.2502
4  1217        TCGA-3C-AALK                    0 BRCA.rnaseq 589.1601
5   158        TCGA-4H-AAAK                    0 BRCA.rnaseq 615.7447
6  1477        TCGA-5L-AAT0                    0 BRCA.rnaseq 896.3191
{% endhighlight %}

13 patients have clinical info but they do not have expression information so I remove them from the analysis.


{% highlight r %}
table(BRCA_HNSC.surv_rnaseq$cohort, useNA = "always")
{% endhighlight %}



{% highlight text %}

BRCA.rnaseq HNSC.rnaseq        <NA> 
       1093         519          13 
{% endhighlight %}



{% highlight r %}
BRCA_HNSC.surv_rnaseq <- BRCA_HNSC.surv_rnaseq %>%
   filter(!is.na(cohort))
{% endhighlight %}

The complete data used for further analysis is printed below


{% highlight r %}
dim(BRCA_HNSC.surv_rnaseq)
{% endhighlight %}



{% highlight text %}
[1] 1612    5
{% endhighlight %}



{% highlight r %}
head(BRCA_HNSC.surv_rnaseq)
{% endhighlight %}



{% highlight text %}
  times bcr_patient_barcode patient.vital_status      cohort    ABCD4
1  3767        TCGA-3C-AAAU                    0 BRCA.rnaseq 322.2560
2  3801        TCGA-3C-AALI                    0 BRCA.rnaseq 486.6775
3  1228        TCGA-3C-AALJ                    0 BRCA.rnaseq 308.2502
4  1217        TCGA-3C-AALK                    0 BRCA.rnaseq 589.1601
5   158        TCGA-4H-AAAK                    0 BRCA.rnaseq 615.7447
6  1477        TCGA-5L-AAT0                    0 BRCA.rnaseq 896.3191
{% endhighlight %}

# maxstat - maximally selected rank statistics

[kassambara](https://github.com/kassambara) prepared a functionality that uses the [maxstat(maximally selected rank statistics)](https://cran.r-project.org/web/packages/maxstat/index.html) statistic to determine the optimal cutpoint for continuous variables and provided it in the [survminer](https://cran.rstudio.com/web/packages/survminer/index.html) package. The development process is described on the [survminer issues track](https://github.com/kassambara/survminer/issues)


{% highlight r %}
#devtools::install_github('kassambara/survminer')
library(survminer)
{% endhighlight %}

Articles in which the maxstat statistics was used:

- [http://www.haematologica.org/content/99/9/1410](http://www.haematologica.org/content/99/9/1410)
- [http://www.bloodjournal.org/content/bloodjournal/120/5/1087.full.pdf](http://www.bloodjournal.org/content/bloodjournal/120/5/1087.full.pdf)
- [http://www.ncbi.nlm.nih.gov/pmc/articles/PMC4058021/pdf/oncotarget-05-2487.pdf](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC4058021/pdf/oncotarget-05-2487.pdf)
- [http://www.impactjournals.com/oncotarget/index.php?journal=oncotarget&page=article&op=view&path[]=3237&path[]=6227](http://www.impactjournals.com/oncotarget/index.php?journal=oncotarget&page=article&op=view&path[]=3237&path[]=6227)

Explanation of the selection process for the cutpoint is described [in this vignette, chapter 2](https://cran.r-project.org/web/packages/maxstat/vignettes/maxstat.pdf)


Determining the optimal cutpoint for ABCD4 gene's expression


{% highlight r %}
BRCA_HNSC.surv_rnaseq.cut <- surv_cutpoint(
   BRCA_HNSC.surv_rnaseq,
   time = "times",
   event = "patient.vital_status",
   variables = c("ABCD4", "cohort")
)
summary(BRCA_HNSC.surv_rnaseq.cut)
{% endhighlight %}



{% highlight text %}
      cutpoint statistic
ABCD4 468.6417  1.944138
{% endhighlight %}

Plot the cutpoint for gene ABCD4


{% highlight r %}
# palette = "npg" (nature publishing group), see ?ggpubr::ggpar
plot(BRCA_HNSC.surv_rnaseq.cut, "ABCD4", palette = "npg")
{% endhighlight %}



{% highlight text %}
$ABCD4
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/figure/source/2016-11-21-Optimal-Cutpoint-maxstat/unnamed-chunk-8-1.png)

Categorize ABCD4 variable


{% highlight r %}
BRCA_HNSC.surv_rnaseq.cat <- surv_categorize(BRCA_HNSC.surv_rnaseq.cut) 
{% endhighlight %}

# Fit and visualize Kaplan-Meier estimates of survival curves 

Below I divided patients into 4 groups, denoting the membership to cancer type and patient's ABCD4 gene's expression level.


RTCGA way


{% highlight r %}
RTCGA::kmTCGA(
   BRCA_HNSC.surv_rnaseq.cat, 
   explanatory.names = c("ABCD4", "cohort"),
   pval = TRUE,
   conf.int = TRUE,
   xlim = c(0,3000)
)
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/figure/source/2016-11-21-Optimal-Cutpoint-maxstat/unnamed-chunk-10-1.png)

survminer way


{% highlight r %}
library(survival)
fit <- survfit(Surv(times, patient.vital_status) ~ ABCD4 + cohort,
               data = BRCA_HNSC.surv_rnaseq.cat)
ggsurvplot(
   fit,                     # survfit object with calculated statistics.
   risk.table = TRUE,       # show risk table.
   pval = TRUE,             # show p-value of log-rank test.
   conf.int = TRUE,         # show confidence intervals for 
                            # point estimaes of survival curves.
   xlim = c(0,3000),        # present narrower X axis, but not affect
                            # survival estimates.
   break.time.by = 1000,    # break X axis in time intervals by 500.
   ggtheme = theme_RTCGA(), # customize plot and risk table with a theme.
 risk.table.y.text.col = T, # colour risk table text annotations.
  risk.table.y.text = FALSE # show bars instead of names in text annotations
                            # in legend of risk table
)
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/figure/source/2016-11-21-Optimal-Cutpoint-maxstat/unnamed-chunk-11-1.png)

