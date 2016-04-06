---
layout: post
title: Answers to FAQ about SparkR for R users
comments: true
published: true
author: Marcin Kosiński
categories: [RTech]
---

{:toc}




Many people keep asking me whether I have tried [SparkR](http://spark.apache.org/docs/latest/sparkr.html), is it worthy using, is it sexy or WHAT is it at all. I felt that writing answers to the frequently asked question (**FAQ**) in the are of **WHAT is that Spark/SparkR?** would help many R Scientists to understand this **Big Data Buzz**-tool.


## Q1: How to explain what is Spark to the regular R user?

From the documentation of [Spark Overview](http://spark.apache.org/docs/latest/#spark-overview)

> Apache Spark is a fast and general-purpose cluster computing system. It provides high-level APIs in Java, Scala, Python and R, and an optimized engine that supports general execution graphs. It also supports a rich set of higher-level tools including Spark SQL for SQL and structured data processing, MLlib for machine learning, GraphX for graph processing, and Spark Streaming.

So in simple words it is a computational engine that is being optimized for speed of computations.

## Q2: Can I run Spark applications from R: SparkR

From the Spark Overview we know that it is possible to use Spark from R without knowing Spark syntax at all. It can be done through [SparkR: R frontend for Spark](https://amplab-extras.github.io/SparkR-pkg/) package, which is included in Spark since the version 1.4.0. Now Spark is released under version 1.6.1.

## Q3: How do I install SparkR?

By now, SparkR is distributed with Spark. So after you install Spark you will get SparkR package in the installation directory.

## Q4: How do I install Spark?

Spark can be downloaded from [this site](http://spark.apache.org/downloads.html) and can be build with the [following instructions](http://spark.apache.org/docs/latest/building-spark.html)

## Q5: Can I use Spark on Windows?

Sure!

> I found the easiest solution on Windows is to build from source.

Explanation on [stackoverflow](http://stackoverflow.com/a/25485800/3857701)


## Q6: How to start Spark application using SparkR from shell?


## Q7: How to start Spark application using SparkR from RStudio?


## Q8: What are those Spark Context and how I can benefit from them?

## Q9: Can I run Spark application locally or only on Yarn Hadoop Cluster?

## Q10: What can be potential Spark data source?

## Q11: Can I use R functions on Spark?

## Q12: What about machine learning and MLlib. Which algorithms are available from Spark and SparkR?

