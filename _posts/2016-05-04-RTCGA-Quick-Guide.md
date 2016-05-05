---
layout: post
title: RTCGA factory of R packages - Quick Guide
comments: true
published: true
author: Marcin Kosiński
categories: [Rbio]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---


<img src="https://raw.githubusercontent.com/RTCGA/RTCGA/master/RTCGA_workflow_ver3.png" class="fit image"> Yesterday we have been delivered with the [new version of R](http://www.r-bloggers.com/r-3-3-0-is-released/) - R 3.3.0 (codename **Supposedly Educational**). This enabled [Bioconductor](http://bioconductor.org/) (yes, not all packages are distributed on [CRAN](https://cran.r-project.org/)) to release it's new [version 3.3](https://www.bioconductor.org/developers/release-schedule/). This means that all packages held on Bioconductor, that were under rapid and vivid development, have been moved to stable-release versions and now can be easily installed. This happens once or twice a year. With that date I have finished work with [RTCGA](http://rtcga.github.io/RTCGA/) package and released, on Bioconductor, the **RTCGA Factory of R Packages**. Read this quick guide to find out more about this R Toolkit for Biostatistics with the usage of data from [The Cancer Genome Atlas](http://cancergenome.nih.gov/) study.

# About The Cancer Genome Atlas

> The Cancer Genome Atlas (TCGA) is a comprehensive and coordinated effort to accelerate our understanding of the molecular basis of cancer through the application of genome analysis technologies, including large-scale genome sequencing - [http://cancergenome.nih.gov/](http://cancergenome.nih.gov/). 

[Our team](https://github.com/orgs/RTCGA/people) converted selected datasets from this study into few separate packages that are hosted on Bioconductor. These R packages make selected datasets easier to access and manage. Data sets in RTCGA packages are large and cover complex relations between clinical outcomes and genetic background.

To use RTCGA install package with instructions from it's [Bioconductor home page](https://www.bioconductor.org/packages/RTCGA/)


{% highlight r %}
## try http:// if https:// URLs are not supported
source("https://bioconductor.org/biocLite.R")
biocLite("RTCGA")
{% endhighlight %}

# Check, Download and Read Data

Packages from the RTCGA factory  will be useful for at least three audiences: biostatisticians that work with cancer data; researchers that are working on large scale algorithms, for them RTGCA data will be a perfect blasting site; teachers that are presenting data analysis method on real data problems. 


{% highlight r %}
library(RTCGA)
{% endhighlight %}


TCGA releases various datasets over time for different cohorts, that are determined by cancer types. One can check

- `infoTCGA()` - what are cohort codes and counts for each cohort from TCGA project,
- `checkTCGA('Dates')` - what are TCGA datasets' dates of release,
- `checkTCGA('DataSets', cancerType = "BRC")` - what are TCGA datasets' names for current release date and cohort.

With that knowledge we are able to download specific datasets from TCGA study. The following command downloads datasets that have string `Merge_Clinical.Level_1` in it's name for `BRCA` cohort type (**Breast carcinoma**) for `2015-11-01` date of release.


{% highlight r %}
downloadTCGA(cancerTypes = "BRCA",
             dataSet = "Merge_Clinical.Level_1",
             destDir = "output_dir",
             date = "2015-11-01")
{% endhighlight %}

For specific datasets (8 types) we have prepared `readTCGA` funciton that reads dataset to the tidy format, using `datatable::fread` function. For expression datasets we also change columns types to natural numeric values.


{% highlight r %}
readTCGA(path = file.path("output_dir",
                          grep("clinical_clin_format.txt",
                               list.files("output_dir/",
                                          recursive = TRUE),
                               value = TRUE)
                          ),
         dataType = "clinical") -> BRCA.clinical.20151101
dim(BRCA.clinical.20151101)
{% endhighlight %}



{% highlight text %}
[1] 1098 1494
{% endhighlight %}

# Prepared Available Datasets

For the most popular datasets types we have prepared data packages that provides various genetic information for `2015-11-01` date of TCGA release. You can read about those datasets and install them with

{% highlight r %}
?datasetsTCGA
?installTCGA
{% endhighlight %}
Those datasets can be converted to Bioconductor format with `convertTCGA` function. You can check full documentation prepared with [staticdocs](https://github.com/hadley/staticdocs) here - [http://rtcga.github.io/RTCGA/staticdocs/](http://rtcga.github.io/RTCGA/staticdocs/).

# Manipulate and Visualize Data

For prepared datasets we have provided functions to manipulate and visualize effect of statistical procedures like Principal Component Analysis (based on [ggbiplot](https://github.com/vqv/ggbiplot)) or estimates of the Kaplan-Meier survival curves (based on the elegant [survminer](http://www.sthda.com/english/rpkgs/survminer/) package). Check few examples below

### Survival Curves


{% highlight r %}
library(RTCGA.clinical)
survivalTCGA(BRCA.clinical,
             OV.clinical,
             extract.cols = "admin.disease_code") -> BRCAOV.survInfo
## Kaplan-Meier Survival Curves
kmTCGA(BRCAOV.survInfo,
       explanatory.names = "admin.disease_code",
       pval = TRUE,
       xlim = c(0,2000),
       break.time.by = 500)
{% endhighlight %}

![plot of chunk unnamed-chunk-7](/figure/source/2016-05-04-RTCGA-Quick-Guide/unnamed-chunk-7-1.png)

### PCA Biplot


{% highlight r %}
library(dplyr)
## RNASeq expressions
library(RTCGA.rnaseq)
expressionsTCGA(BRCA.rnaseq, OV.rnaseq, HNSC.rnaseq) %>%
   rename(cohort = dataset) %>%  
   filter(substr(bcr_patient_barcode, 14, 15) == "01") -> 
   BRCA.OV.HNSC.rnaseq.cancer

pcaTCGA(BRCA.OV.HNSC.rnaseq.cancer,
        group.names = "cohort",
        title = "Genes expressions vs cohort types")
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/figure/source/2016-05-04-RTCGA-Quick-Guide/unnamed-chunk-8-1.png)

For more visualization examples visit [RTCGA project website](http://rtcga.github.io/RTCGA/Visualizations.html). If you have noticed any bugs or have any reflections please open an issue under [project's repository](https://github.com/RTCGA/RTCGA/issues/new) or post a comment on below Disqus panel. 

