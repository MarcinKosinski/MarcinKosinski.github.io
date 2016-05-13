---
layout: post
title: R 3.3.0 is another motivation for Docker
comments: true
published: true
author: Marcin Kosiński
categories: [Rtech]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---


<img src="/images/fulls/Rocker.jpg" class="fit image"> Have you ever encountered R packages versioning issues when one application required different dependent packages versions that other? Have you ever got stuck with your project because of wrong pre-installed software versions on machine on which you should run your code?  Or maybe you had heavy adventures with installing R software on a new machine because you couldn't recall all installation steps like; what have I done 2 years ago that RCurl works on my local machine but I can't install it now on my virtual machine with Windows? Or maybe installation of your R project on new machine was easy but admin couldn't manage with this process, as he's not regular R user? **If you ever find it problematic to move your R applications to other machines, then this [Docker](https://www.docker.com/) guid post is for you!**

## Docker Motivation

Recently R world have been announced with the new R 3.3.0 version (which was a great occasion to release many R projects like [RTCGA](http://r-addict.com/2016/05/04/RTCGA-Quick-Guide.html)). This is great that R open source project continues to grow, but switching to new software versions should be done carefully, especially when working in a team of R developers.

It is almost a regular daily routine that each person developing an R pet project has various libraries and R itself versions, which can be checked with `devtools::session_info()`. This causes situations in which application work on one machine but not on the other (or works partially), and mistakenly it might seem that having the newest R and its libraries versions is the correct way of cooperation (NOT!).

In most cases it is not complicated to configure one machine to support one R application, but what if you move your R project to a machine that have configured Shiny Server that should handle dozens of R applications, developed by people that might now even know each others? 
[Our team](https://github.com/orgs/mi2-warsaw/teams/wczasowicze) have stacked for a week after we've tried to publish our shiny application (that I wrote about it [here](http://r-addict.com/2016/04/20/Disqus-Shinydashboards.html)) due to server configurations for already working applications, but not for a one developed under new R 3.3.0. 

The server (example: [http://mi2.mini.pw.edu.pl:8080/RTCGA/](http://mi2.mini.pw.edu.pl:8080/RTCGA/) maintains applications created by students and graduates of Faculty of Mathematics and Information Science, Warsaw University of Technology and works under R 3.2.4 and some version of [shiny](http://shiny.rstudio.com/) package. We've found it impossible to install the newest version of shiny package  under the user-specified library path (not to affect other applications) as now it requires R 3.3.0, so the application broke completely. I could install R from scratch in the user-specified path and configure all packages and related software but I thought: what if we move our application in a month to another, better server? Would I have to proceed with the installation and configuration once more? Can't I do it once and have black box application that would work always and anywhere not affecting other processes? 

> Yes, I can. I used Docker.

## Docker short intro

Few sentences from Docker site might help understand [Whas is Docker?](https://www.docker.com/what-docker)

> Docker allows you to package an application with all of its dependencies into a standardized unit for software development.

> Docker containers wrap up a piece of software in a complete filesystem that contains everything it needs to run: code, runtime, system tools, system libraries – anything you can install on a server. This guarantees that it will always run the same, regardless of the environment it is running in. 

> When your app is in Docker containers, you don’t have to worry about setting up and maintaining different environments or different tooling for each language. Focus on creating new features, fixing issues and shipping software.

In a few words Docker helps creating black box applications that are pre-configured and standalone and can work regardless of the outside software. 

## rocker 

[Dirk Eddelbuettel](https://github.com/eddelbuettel) have prepared repository on [hub.docker.com](https://hub.docker.com/u/rocker/) called rocker which consists of docker containers having r-base, RStudio or Shiny Server ([Introducing Rocker: Docker for R](http://www.r-bloggers.com/introducing-rocker-docker-for-r/)). Every other Docker with R application can be build from other, more basic Docker, so you can develop your applications starting from [r-base](https://hub.docker.com/r/rocker/r-base/) Docker image.

After you [install Docker](https://docs.docker.com/engine/installation/) you can use open source shared containers with pre-configured software like Hadoop, Spark or Shiny Server. Below is the example on how to easily run Shiny Server on port 3838


{% highlight r %}
sudo docker run --rm -p 3838:3838 rocker/shiny
{% endhighlight %}

which will by default look for containers in [hub.docker.com](https://hub.docker.com), where `rocker` stands for repository name and `shiny` stands for container name.

You can check running Docker images with

{% highlight r %}
sudo docker ps
{% endhighlight %}

and check which Dockers images have you already downloaded or build locally


{% highlight r %}
sudo docker images
{% endhighlight %}

Sometimes you would like to enable Docker running image to communicate with the outer folder. It can be done with the `-v` flag like that


{% highlight r %}
docker run --rm -p 3838:3838 \
    -v /srv/shinyapps/:/srv/shiny-server/ \
    -v /srv/shinylog/:/var/log/ \
    rocker/shiny
{% endhighlight %}

where the first part of `/srv/shinyapps/:/srv/shiny-server/` is the outer path on machine and the part after `:` is the path inside docker which will be merged with the outher path. 

This is how we run our pet R project shiny dahsboard application [http://mi2.mini.pw.edu.pl:38383/CzasDojazdu/App/](http://mi2.mini.pw.edu.pl:38383/CzasDojazdu/App/) that enables to find rooms in Warsaw that are available to rent, with restrictions to the time distance from localization specified by a user.


## Creating your first Docker

Each Docker needs it configuration file named `Dockerfile` where you specify from which basic Docker image you would like to build your Docker container. [Here](https://github.com/mi2-warsaw/rocker/blob/master/shiny-extra/Dockerfile) is a link to a `shiny-extra` Docker configuration file which is also included below



{% highlight r %}
FROM rocker/shiny:latest 

MAINTAINER Marcin Kosiński "m.p.kosinski@gmail.com"

# install additional packages
RUN R -e "install.packages(c('shinydashboard', 'leaflet', 'dplyr', 'ggmap', 'stringi', 'RSQLite', 'DT'), repos='https://cran.rstudio.com/')"

CMD ["/usr/bin/shiny-server.sh"]
{% endhighlight %}

where `FROM` tells from which image you will build your Docker, `RUN` tells what command to run during your Docker container is being build and `CMD` tells which script to run when the Docker is run. In that case it is a script that runs a Shiny Server with additional R packages `c('shinydashboard', 'leaflet', 'dplyr', 'ggmap', 'stringi', 'RSQLite', 'DT')`.


I hope this post have convinced you to consider using Dockers based on rocker in your applications. It is standalone, easy to move to other machines and there already exist many containers with advanced pre-configured software.
