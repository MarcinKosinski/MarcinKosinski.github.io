---
layout: post
title: Extending sparklyr to Compute Cost for K-means on YARN Cluster with Spark ML Library
comments: true
published: true
author: Marcin Kosiński
categories: [Spark, Clustering]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/sparklyr.png" class="fit image"> Machine and statistical learning wizards are becoming more eager to perform analysis with [Spark ML](https://spark.apache.org/docs/2.0.0/ml-clustering.html) library if this is only possible. It's trendy, posh, spicy and gives the feeling of doing state of the art machine learning and being up to date with the newest computational trends. It is even more sexy and powerful when computations can be performed on the extraordinarily enormous computation cluster - let's say 100 machines on [YARN hadoop](https://hadoop.apache.org/docs/r2.7.2/hadoop-yarn/hadoop-yarn-site/YARN.html) cluster makes you the real data cruncher! In this post I present [sparklyr](http://spark.rstudio.com/) package (by [RStudio](github.com/rstudio/sparklyr)), the connector that will transform you from a regular R user, to the supa! data scientist that can invoke Scala code to perform machine learning algorithms on YARN cluster just from RStudio! Moreover, I present how I have extended the interface to K-means procedure, so that now it is also possible to compute cost for that model, which might be beneficial in determining the number of clusters in segmentation problems. **Thought about learnig Scala? Leave it - user sparklyr!**

* sparklyr basics
* dplyr and DBI interface on Spark
* Running Spark ML Machine Learning K-means Algorithm from R
{:toc}



If you don't know much about Spark yet, you can read my April post [Answers to FAQ about SparkR for R users](http://r-addict.com/2016/04/06/FAQ-SparkR.html) - where I explained how could we use `SparkR` package that is distributed with Spark. Many things (code) might have changed since that time, due to the rapid development caused by great popularity of Spark. Now we can use version 2.0.0 of Spark. If you are migrating from previous versions I suggest you should look at [Migration Guide - Upgrading From SparkR 1.6.x to 2.0](http://spark.apache.org/docs/latest/sparkr.html#migration-guide). 

# sparklyr basics

This packages is based on [sparkapi](http://github.com/rstudio/sparkapi/) package that enables to run Spark applications locally or on YARN cluster just from R. It translates R code to bash invocation of spark-shell. It's biggest advantage is [dplyr](https://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html) interface for working with Spark Data Frames (that might be Hive Tables) and possibility to invoke algorithms from Spark ML library. 

Installation of sparklyr, then Spark itself and simple application initiation is described by this code


{% highlight r %}
library(devtools)
install_github('rstudio/sparklyr')
library(sparklyr)
spark_install(version = "2.0.0")
sc <- 
spark_connect(master="yarn",
   config = list(
     default = list(
       spark.submit.deployMode= "client",
       spark.executor.instances= 20, 
       spark.executor.memory= "2G",
       spark.executor.cores= 4,
       spark.driver.memory= "4G")))
{% endhighlight %}

One don't have to specify config by himself, but if this is desired then remember that you could also specify parameters for Spark application with [config.yml](http://spark.rstudio.com/deployment.html#configuration) files so that you can benefit from many profiles (development, production). In version 2.0.0 it is desired to name master `yarn` instead of `yarn-client` and passing the `deployMode` parameter, which is different from version 1.6.x. All available parameters can be found in [Running Spark on YARN](http://spark.apache.org/docs/latest/running-on-yarn.html) documentation page. 

# dplyr and DBI interface on Spark

When connecting to YARN, it is most probable that you would like to use data tables that are stored on Hive. Remember that

> Configuration of Hive is done by placing your hive-site.xml, core-site.xml (for security configuration), and hdfs-site.xml (for HDFS configuration) file in conf/.

where `conf/` is set as `HADOOP_CONF_DIR`. Read more about using [Hive tables from Spark](http://spark.apache.org/docs/latest/sql-programming-guide.html#hive-tables)

If everything is set up and the application runs properly, you can use dplyr interface to provide lazy evaluation for data manipulations. Data are stored on Hive, Spark application runs on YARN cluster, and the code is invoked from R in the simple language of data transformations (dplyr) - everything thanks to sparklyr team great job! Easy example is below


{% highlight r %}
library(dplyr)
# give the list of tables
src_tbls(sc) 
# copies iris from R to Hive
iris_tbl <- copy_to(sc, iris, "iris") 
# create a hook for data stored on Hive
data_tbl <- tbl(sc, "table_name") 
data_tbl2 <- tbl(sc, sql("SELECT * from table_name"))
{% endhighlight %}

You can also perform any operation on datasets use by Spark


{% highlight r %}
iris_tbl %>%
   select(Petal_Length, Petal_Width) %>%
   top_n(40, Petal_Width) %>%
   arrange(Petal_Length)
{% endhighlight %}

Note that original commas in iris names have been translated to `_`.


This package also provides interface for functions defined in `DBI` package


{% highlight r %}
library(DBI)
dbListTables(sc)
dbGetQuery(sc, "use database_name")
data_tbl3 <- dbGetQuery(sc, "SELECT * from table_name")
dbListFields(sc, data_tbl3)
{% endhighlight %}

# Running Spark ML Machine Learning K-means Algorithm from R

The basic example on how sparklyr invokes Scala code from Spark ML will be presented on K-means algorithm.
If you check the code of `sparklyr::ml_kmeans` function you will see that for input `tbl_spark` object, named x and character vector containing features' names (`featuers`)


{% highlight r %}
envir <- new.env(parent = emptyenv())
df <- spark_dataframe(x)
sc <- spark_connection(df)
df <- ml_prepare_features(df, features)
tdf <- ml_prepare_dataframe(df, features, ml.options = ml.options, envir = envir)
{% endhighlight %}

sparklyr ensures that you have proper connection to spark data frame and prepares features in convenient form and naming convention. At the end it prepares a Spark DataFrame for Spark ML routines.

This is done in a new environment, so that we can store arguments for future ML algorithm and the model itself in its own environment. This is safe and clean solution. You can construct a simple model calling a Spark ML class like this


{% highlight r %}
envir$model <- "org.apache.spark.ml.clustering.KMeans"
kmeans <- invoke_new(sc, envir$model)
{% endhighlight %}

which invokes new object of class `KMeans` on which we can invoke parameters setters to change default parameters like this



{% highlight r %}
model <- kmeans %>%
    invoke("setK", centers) %>%
    invoke("setMaxIter", iter.max) %>%
    invoke("setTol", tolerance) %>%
    invoke("setFeaturesCol", envir$features)
# features where set in ml_prepare_dataframe
{% endhighlight %}

For an existing object of `KMeans` class we can invoke its method called `fit` that is responsible for starting the K-means clustering algorithm 


{% highlight r %}
fit <- model %>%
invoke("fit", tdf)
{% endhighlight %}

which returns new object on which we can compute, e.g centers of outputted clustering


{% highlight r %}
kmmCenters <- invoke(fit, "clusterCenters")
{% endhighlight %}

or the Within Set Sum of Squared Errors (called Cost) (which is mine small contribution [#173](https://github.com/rstudio/sparklyr/pull/173) )


{% highlight r %}
kmmCost <- invoke(fit, "computeCost", tdf)
{% endhighlight %}

This sometimes helps to decide [how many clusters should we specify for clustering problem](http://stackoverflow.com/a/15376462/3857701)

<img src="/images/fulls/kmeans.png" class="fit image">

and is presented in `print` method for `ml_model_kmeans` object


{% highlight r %}
iris_tbl %>%
   select(Petal_Width, Petal_Length) %>%
   ml_kmeans(centers = 3, compute.cost = TRUE) %>%
   print()

K-means clustering with 3 clusters

Cluster centers:
  Petal_Width Petal_Length
1    1.359259     4.292593
2    2.047826     5.626087
3    0.246000     1.462000

Within Set Sum of Squared Errors =  31.41289
{% endhighlight %}

All that can be better understood if we'll have a look on [Spark ML docuemtnation for KMeans](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.ml.clustering.KMeans) ([be carefull not to confuse with Spark MLlib](https://github.com/rstudio/sparklyr/issues/178#issuecomment-240472368) where methods and parameters have different names than those in Spark ML). This enabled me to provide simple update for `ml_kmeans()` ([#179](https://github.com/rstudio/sparklyr/pull/179)) so that we can specify `tol` (tolerance) parameter in `ml_kmeans()` to support tolerance of convergence.
