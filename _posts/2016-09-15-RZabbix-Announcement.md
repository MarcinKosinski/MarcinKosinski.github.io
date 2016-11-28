---
layout: post
title: Monitoring R Applications with RZabbix
comments: true
published: true
author: Marcin Kosiński
categories: [Development]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/RZabbix.png" class="fit image"> As R users we mostly perform analysis, produce reports and create interactive shiny applications. Those are rather one-time performances. Sometimes, however, the R developer enters the world of the real software development, where R applications should be distributed and maintained on many machines. Then one really appreciates the value of a proper applications monitoring. In this post I present my last package called [RZabbix](https://cran.r-project.org/web/packages/RZabbix/index.html) - the R interface to the [Zabbix API](<https://www.zabbix.com/documentation/3.0/manual/api/reference>) data.

* What is Zabbix?
* Connect from R with RZabbix
* Example of usage
* You can contribute!
{:toc}



# What is Zabbix?

According to the [product overview](http://www.zabbix.com/product.php)

> With Zabbix it is possible to gather virtually limitless types of data from the network. High performance real-time monitoring means that tens of thousands of servers, virtual machines and network devices can be monitored simultaneously. Along with storing the data, visualization features are available (overviews, maps, graphs, screens, etc), as well as very flexible ways of analyzing the data for the purpose of alerting.

This **Ultimate Enterprise-class Monitoring Platform** was nominated for the 2nd time among world's best monitoring solutions according to Gartner

Zabbix can be used to [Monitor everything](http://www.zabbix.com/monitor_everything.php) - servers, network devices and applications, gathering accurate statistics and performance data. 

<!-- Place this tag in your head or just before your close body tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>
<!-- Place this tag where you want the button to render. -->

<h1> Connect from R with RZabbix <a aria-label="Star MarcinKosinski/RZabbix on GitHub" data-count-aria-label="# stargazers on GitHub" data-count-api="/repos/MarcinKosinski/RZabbix#stargazers_count" data-count-href="/MarcinKosinski/RZabbix/stargazers" data-style="mega" data-icon="octicon-star" href="https://github.com/MarcinKosinski/RZabbix" class="github-button">Star</a></h1>

I have created a simple R connector to [Zabbix API](https://www.zabbix.com/documentation/3.2/manual/api/reference), packed in the library and released it on [Share Zabbix](https://share.zabbix.com/dir-libraries/rzabbix) and also on [CRAN](https://cran.r-project.org/web/packages/RZabbix/index.html). 

You can install the package with


{% highlight r %}
install.packages('RZabbix') 
library(RZabbix)
?ZabbixAPI # help manual page
{% endhighlight %}
and then connect to Zabbix API with


{% highlight r %}
ZabbixAPI(
   'http://localhost/zabbix/',
    body = 
      list(method = "user.login",
           params = jsonlite::unbox(
            data.frame(user = "Admin",
                       password = "zabbix")))
   ) -> auth
{% endhighlight %}

The authentication is required, because every other call need to use `auth` parameter in its body.

Base information of the API version can be checked with


{% highlight r %}
ZabbixAPI(
   'http://localhost/zabbix',
    body = 
       list(method = "apiinfo.version"))
{% endhighlight %}

and the regular request to get history of an item of `item_id` number can be done as below


{% highlight r %}
ZabbixAPI(
   'http://localhost/zabbix',
    body = 
       list(method = "history.get",
            params = jsonlite::unbox(
           data.frame(output = "extend",
                      itemids = "item_id",
                      history = 0,
                      sortfield = "clock",
                      sortorder = "DESC",
                      limit = 10)
               ),
               auth = auth))
{% endhighlight %}

The body parameter can be also passed as a JSON string - see example for getting event data for object with `object_id` number


{% highlight r %}
library(jsonlite)
paste0('{
    "method": "event.get",
    "params": {
        "output": "extend",
        "select_acknowledges": "extend",
        "objectids": "object_id",
        "sortfield": ["clock", "eventid"],
        "sortorder": "DESC"
    },
    "auth": "', auth, '"
}') -> json_rpc

ZabbixAPI('http://localhost/zabbix',
          body = fromJSON(json_rpc)) -> event.info
{% endhighlight %}

Methods and params to Zabbix API can be found in [Method reference](https://www.zabbix.com/documentation/3.2/manual/api/reference). It is a great date, because [few days ago new version of Zabbix (3.2) was released](https://www.facebook.com/zabbix/photos/a.457911347570950.119798.144923832203038/1420122278016514/?type=3&theater).

<img src="/images/fulls/zab_32.jpg" class="fit image">


You can try to set your own [Zabbix Server from Docker container](https://www.zabbix.org/wiki/Dockerized_Zabbix) or you can use [public test Zabbix on zabbix.org](http://zabbix.org/zabbix/). If you are not yet motivated for using Docker then read my post from May [
R 3.3.0 is another motivation for Docker](http://r-addict.com/2016/05/13/Docker-Motivation.html) or join [10th Cracow R User Group meetup](http://www.meetup.com/Cracow-R-User-Group/events/233624341/) at the end of September where I will present [Rocker: explanation and motivation for Docker containers usage in applications development (by R user, for R users)](http://erkakrakow.pl/index.php/spotkania/24-10-spotkanie-entuzjastow-r-w-krakowie)

# Example of usage

[Jan Garaj](https://github.com/jangaraj) from [Monitoring Artist](http://www.monitoringartist.com/) has used RZabbix to provide [Monitoring Analytics](https://github.com/monitoringartist/monitoring-analytics) - R statistical computing and graphic tool for monitoring metrics from data scientists.

![Monitoring Analytics](https://raw.githubusercontent.com/monitoringartist/monitoring-analytics/master/doc/monitoring-analytics.gif) 

# You can contribute!

RZabbix is still a young package and many can be done. If you would like to contribute then fill free to <a aria-label="Fork MarcinKosinski/RZabbix on GitHub" data-count-aria-label="# forks on GitHub" data-count-api="/repos/MarcinKosinski/RZabbix#forks_count" data-count-href="/MarcinKosinski/RZabbix/network" data-icon="octicon-repo-forked" href="https://github.com/MarcinKosinski/RZabbix/fork" class="github-button">Fork</a> the RZabbix repository. I am currently looking for examples on how to [create a SLA overview for a specific host, calculate the amount of diskspace used by all active hosts and filesystems and create a overview of the most active triggers](https://github.com/MarcinKosinski/RZabbix/issues/6#issue-152191501) and I am thinking about [adding tests](https://github.com/MarcinKosinski/RZabbix/issues/1).
