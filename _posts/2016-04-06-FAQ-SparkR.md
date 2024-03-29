---
layout: post
title: Answers to FAQ about SparkR for R users
comments: true
published: true
author: Marcin Kosiński
categories: [Spark]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/07.jpg" class="fit image"> Many people keep asking me whether I have tried [SparkR](http://spark.apache.org/docs/latest/sparkr.html), is it worth using, is it sexy or **WHAT is it at all**. I felt that creating frequently asked questions (**FAQ**) in the field of **WHAT is that Spark/SparkR?** would help many R Scientists to understand this **Big Data Buzz**-tool. I have gathered information from the documentation and some code from stackoverflow questions in preparation for the list below.


* Q1: How to explain what is Spark to the regular R user
* Q2: Can I run Spark applications from R: SparkR
* Q3: How do I install SparkR?
* Q4: How do I install Spark?
* Q5: Can I use Spark on Windows?
{:toc}

## Q1: How to explain what is Spark to the regular R user?

From the documentation of [Spark Overview](http://spark.apache.org/docs/latest/#spark-overview)

> Apache Spark is a fast and general-purpose cluster computing system. It provides high-level APIs in Java, Scala, Python and R, and an optimized engine that supports general execution graphs. It also supports a rich set of higher-level tools including Spark SQL for SQL and structured data processing, MLlib for machine learning, GraphX for graph processing, and Spark Streaming.

So in simple words it is a computational engine that is being optimized for speed of computations.

## Q2: Can I run Spark applications from R: SparkR?

From the Spark Overview we know that it is possible to use Spark from R without knowing Spark syntax at all. It can be done through [SparkR: R frontend for Spark](https://amplab-extras.github.io/SparkR-pkg/) package, which is included in Spark since the version 1.4.0. Now Spark is released under version 1.6.1.

## Q3: How do I install SparkR?

By now, SparkR is distributed with Spark. So after you install Spark you will get SparkR package in the installation directory. This means that loading SparkR package will be possible from the `/R/lib` subdirectory of the directory in which Spark was installed with the usage of `lib.loc` parameter in `library()`


{% highlight r %}
library(SparkR, lib.loc = "/opt/spark/R/lib")
{% endhighlight %}

Installation under Windows and OS is well described on this [r-bloggers post](http://www.r-bloggers.com/installing-and-starting-sparkr-locally-on-windows-os-and-rstudio/).

## Q4: How do I install Spark?

Spark can be downloaded from [this site](http://spark.apache.org/downloads.html) and can be build with the [following instructions](http://spark.apache.org/docs/latest/building-spark.html)

## Q5: Can I use Spark on Windows?

Sure! The easiest solution on Windows is to build from source, after you first get [Maven](http://maven.apache.org/). See this guide [http://spark.apache.org/docs/latest/building-spark.html](http://spark.apache.org/docs/latest/building-spark.html)

Explanation on [stackoverflow](http://stackoverflow.com/a/25485800/3857701)


## Q6: How to start Spark application using SparkR from shell?

Assuming that the installation directory was `/opt/spark` one can run SparkR package in shell with


{% highlight r %}
/opt/spark/bin/sparkR
{% endhighlight %}

where probably `/spark/` directory at your machine would like rather like `spark-ver-hadoop-ver/`.

It will turn on R with attached SparkR package, that have run Spark application locally.
This way of using SparkR grants the advantage of having `Spark context` and `SQL context` created automatically at the
start without user engagement. You can see this with the start information

> Welcome to SparkR!
 Spark context is available as sc, SQL context is available as sqlContext

You can specify the number of executors and executor cores for Spark application with additional arguments

{% highlight r %}
/opt/spark/bin/sparkR --num-executors 5 --executor-cores 5
{% endhighlight %}


## Q7: How to start Spark application using SparkR from RStudio?

If you'd rather use RStudio to work with R frontend, which I really recommend, you have to load
SparkR with the non-default directory. Therefore you should get familiar with `lib.loc` parameter in `library()` function


{% highlight r %}
library(SparkR, lib.loc = "/opt/spark/R/lib")
{% endhighlight %}

Then you will have to start Spark context and SQL context manually with


{% highlight r %}
# this is optional
sparkEnvir <- list(spark.num.executors='5', spark.executor.cores='5')
# initializing Spark context
sc <- sparkR.init(sparkHome = "/opt/spark", 
                  sparkEnvir = sparkEnvir)
# initializing SQL context
sqlContext <- sparkRSQL.init(sc)
{% endhighlight %}

There is also extra `spark.driver.memory` option described in [Starting Up from RStudio](http://spark.apache.org/docs/latest/sparkr.html#starting-up-from-rstudio).

## Q8: What are those all Spark contexts and how I can benefit from them?

Spark context is required to run Spark itself and any other context is required to 
connect Spark with its extensions. SQL Context enables to use Spark SQL which mainly 
means that one can write SQL statements instead of using Spark functions and that Spark SQL module will translate those statements for Spark.

For example `hiveContext` can be created with


{% highlight r %}
hiveContext <- sparkRHive.init(sc)
{% endhighlight %}

and then one would be able to send SQL/HQL statements to HiveServer like


{% highlight r %}
SparkResult <- sql(hiveContext, "select statement")
{% endhighlight %}

> Note that Spark should have been built with Hive support and more details on the difference between SQLContext and HiveContext can be found in the SQL programming guide

- [Hive support](http://spark.apache.org/docs/latest/building-spark.html#building-with-hive-and-jdbc-support)
- [SQL programming guide](http://spark.apache.org/docs/latest/sql-programming-guide.html#starting-point-sqlcontext)


## Q9: Can I run Spark application locally or only on Yarn Hadoop Cluster?

You can do it both ways. Yarn Cluster requires additional `master` parameter specification.

- shell

{% highlight r %}
/opt/spark/bin/sparkR --master yarn-client --num-executors 5 --executor-cores 5
{% endhighlight %}

- RStudio

{% highlight r %}
sc <- sparkR.init(sparkHome = "/opt/spark", 
                  master = "yarn-client",
                  sparkEnvir = sparkEnvir)
{% endhighlight %}

> Ensure that HADOOP_CONF_DIR or YARN_CONF_DIR points to the directory which contains the (client side) configuration files for the Hadoop cluster. 

Source: [Launching Spark on YARN](http://spark.apache.org/docs/latest/running-on-yarn.html#launching-spark-on-yar)n

## Q10: Stopping Spark application?

There's nothing simpler


{% highlight r %}
sparkR.stop()
{% endhighlight %}


## Q11: What can be potential Spark data source?

One can connect to HiveServer by specifying `hiveContext`, but also one can read `csv` files to work with.
This requires additional Spark package, a good start is spark-csv_2.10:1.0.3 package which can be downloaded
from Spark packages repository [http://spark-packages.org/](http://spark-packages.org/)

- shell

{% highlight r %}
/opt/spark/bin/sparkR --packages com.databricks:spark-csv_2.10:1.0.3 --master yarn-client --num-executors 5 --executor-cores 5
{% endhighlight %}

- RStudio


{% highlight r %}
sparkEnvir <- list(spark.num.executors='5', spark.executor.cores='5',
                   packages='com.databricks:spark-csv_2.10:1.0.3')
# initializing Spark context
sc <- sparkR.init(sparkHome = "/opt/spark", 
                  sparkEnvir = sparkEnvir)
{% endhighlight %}



## Q12: Can I use R functions on Spark?

No. Mainly you are using Spark functions, but called from R equivalents. You can not run on Spark engine a function that exists in R but has no equivalents in Spark.

But you can get data from Spark application with


{% highlight r %}
SparkR::collect(SparkResult) -> RResult
{% endhighlight %}

use R function on collected dataset and send it back to the RDD Spark format, so called [SparkR DataFrames](A DataFrame is a distributed collection of data organized into named columns. It is conceptually equivalent to a table in a relational database or a data frame in R, but with richer optimizations under the hood.)



{% highlight r %}
operations_and_functions(RResult) -> RResultChanged
df <- createDataFrame(sqlContext/hiveContext, RResultChanged)
{% endhighlight %}


> A DataFrame is a distributed collection of data organized into named columns. It is conceptually equivalent to a table in a relational database or a data frame in R, but with richer optimizations under the hood.



## Q13: What about machine learning and MLlib?

Spark provide great machine learning library called [MLlib](http://spark.apache.org/mllib/) with algorithms based on stochastic gradient
descent optimization so that Spark can be connected to stream data and use online machine learning methods. It is not possible to connect R with [MLlib](http://spark.apache.org/mllib/) library so far and the only machine learning algorithms available from SparkR are [Gaussian and Binomial GLM models](http://spark.apache.org/docs/latest/sparkr.html#machine-learning).


## Q14: Where can I read more?! I want more examples!

There is a great documentation page about using Spark from R on [Spark Programming Guides](http://spark.apache.org/docs/latest/sparkr.html).


If you have any more questions or feel that something is not clear, you can add a comment in the Disqus panel below. If you'd want to hear more about R integration with Spark like [dplyr.spark.hive](https://github.com/rzilla/dplyr.spark.hive) in the future you can get feed by clicking this [ATOM feed](http://r-addict.com/atom.xml) link.
