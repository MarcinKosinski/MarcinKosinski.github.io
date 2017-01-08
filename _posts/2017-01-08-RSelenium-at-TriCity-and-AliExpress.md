---
layout: post
title: Controlling Expenses on Ali Express with RSelenium
comments: true
published: true
date: 2017-01-08 11:00:00
author: Marcin Kosiński
categories: [Web Scraping]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/rsel.png" class="fit image"> Due to the incrising interest in the Internet and due to the its rising number of users, one can notice the surprising growth in the demand for analyzing data and information in the Internet that were left by users and for users. Many companies and institutions base their business decisions on the extensive research of social media portals and Internet forums, where users leave reviews on various products and brands. **Not only the same analysis, but also the ability to obtain data from the Internet, is a key part of the puzzle...**

... one can read in the description of the second talk (by Krzysztof Słomczynski ([krzyslom](https://github.com/krzyslom))) of the first meetup of the new Tri-City R Users Group - [meet(R) in Tricity!](https://www.meetup.com/Trojmiejska-Grupa-Entuzjastow-R/events/236257032/). The meeting will be held on this Thursday (12-01-2017) at the Gdańsk University of Technology.

* RSelenium
* Launching RSelenium
* Logging to Ali Express
* Few basic RSelenium functions
* Extracting information from Ali Express allowed only for logged user
* Plotting expenses
{:toc}

# RSelenium

Motivated by that upcoming talk I took a tour through [RSelenium](https://cran.r-project.org/web/packages/RSelenium/index.html) vignettes: [RSelenium basics](https://cran.r-project.org/web/packages/RSelenium/vignettes/RSelenium-basics.html) and [RSelenium Docker](https://cran.r-project.org/web/packages/RSelenium/vignettes/RSelenium-docker.html) to launch my first Docker container with Selenium Server. If you are not yet motivated to use Docker containers, then have a look at this post [R 3.3.0 is another motivation for Docker](http://r-addict.com/2016/05/13/Docker-Motivation.html).

RSelenium is an R interface that connects to [Selenium](http://docs.seleniumhq.org/) (Server), which is a project focused on automating web browsers and enables to create a regular web browser session that can be controlled with command lines. Such Internet browsing automatization is a huge trigger for web-harvesting because with RSelenium you are able to simulate a real user and to pass keys to the browser session (such as user login and password). With such a possibility you are able to log into any portal automatically (or manually) and to fill bot security captchas code (this rather manually). You can also interact with web elements that first need to be clicked to show information which are in the demand to be web-scraped.

# Launching RSelenium

I used [Selenium Docker container](https://hub.docker.com/r/selenium/) to launch Selenium Server which is available on Docker hub. The following command launches Selenium Server and binds it to the localhost on the port 4445. It also binds the remote desktop on the port 5901 so that we can run VNC viewer (Vinagre) to observe operations performed on a fake web session.


{% highlight r %}
docker run -d -p 5901:5900 -p 127.0.0.1:4445:4444 selenium/standalone-firefox-debug:2.53.1
# then open vinagre at port 127.0.0.1:5901 for VCN method
{% endhighlight %}

When Docker container is running, we are able to establish a connection with Selenium Server from R with the following commands


{% highlight r %}
library(RSelenium)
remDr <- remoteDriver(port = 4445L)
remDr$open(silent = TRUE)
{% endhighlight %}

# Logging to Ali Express

The result of previous commands is a `remoteDriver` object, in this situation called `remDr`. This S4 object can be used to navigate through the web browser session. The below command navigates to the Ali Express logging page.


{% highlight r %}
remDr$navigate("https://login.aliexpress.com/")
{% endhighlight %}

In this situation you can see than in the remote desktop VNC viewer we have entered Ali Express.

<img src="/images/fulls/ali55.png" class="fit image">

You can log in in 2 ways: by filling fields manually or by finding fields by their properties (like `ir` or `class` name) and by sending keys (like `password` and `user name`) to them.


{% highlight r %}
remDr$findElement("id", "fm-login-id")$sendKeysToElement(list(login))
remDr$findElement("id", "fm-login-password")$sendKeysToElement(list(password))
{% endhighlight %}

Then you can click on the `sign in` button with 


{% highlight r %}
remDr$findElement("id", "fm-login-submit")$clickElement()
{% endhighlight %}

<img src="/images/fulls/ali_automatic_login.png" class="right image"> You can check what are properties of a field with `inspect element` in any web browser, so that you will know how to navigate to the element with `findElement` by it's `id` or by it's `class` or even `css`. For this portal I wasn't successful with sending keys so I logged manually. 

However, for Facebook it worked like a charm


{% highlight r %}
remDr$navigate("https://www.facebook.com/") 
remDr$findElement("id", "email")$sendKeysToElement(list(login))
remDr$findElement("id", "pass")$sendKeysToElement(list(password))
remDr$findElement("id", "loginbutton")$clickElement()
{% endhighlight %}

# Few basic RSelenium functions

In this moment I think few comments about basics of RSelenium commands are required. With `findElement` method you can get the first element on the web page that suits the searching criterion, with `findElements` you can find all such elements. On each such element (or on the list of elements) you can perform further operations like 

- sending data to element (`sendKeysToElement`)
- clicking the element (`clickElement`)
- finding elements within that element (`findChildsElement`)
- extracting the text from element (`getElementText`)
- highlighting the element (`highlightElement`)


and many more!

# Extracting information from Ali Express allowed only for logged user

Ali Express is a market portal where you can order products mainly from China. They are of low quality but also of a cheap price, that's why this portal is very popular, even though the long period of a home delivery (which is free in many cases). You can buy `earrings` or `clothes` for few cents! 

**How much money did `I` :) spend for all my transactions?**

From `My orders` panel, allowed for a logged user I found out that for each page of orders (I had 32 pages of history of my transactions) I can extract whole body of a transaction, and then from that body I can extract the amount that I payed in dollars, check if the transaction wasn't cancelled and get the ID of an operation. I just need to properly specify the names of classes of HTML tags/objects I need.

I will use `order` and `transaction` interchangeably.


{% highlight r %}
# get all orders from the current page
remDr$findElements("class", "order-item-wraper") -> orders
# extract id, amount and status of the first order
orders[[1]]$findChildElement("class", "amount-num")$getElementText()[[1]] -> amount
orders[[1]]$findChildElement("class", "product-action")$getElementText()[[1]] -> payed
orders[[1]]$findChildElement("class", "info-body")$getElementText()[[1]] -> id
# the order date is the second element found with
orders[[1]]$findChildElements("class", "info-body")[[2]]$getElementText()[[1]]
{% endhighlight %}

<img src="/images/fulls/ali1.png" class="fit image">
<img src="/images/fulls/ali999.png" class="fit image">
<img src="/images/fulls/ali1234.png" class="fit image">

For all 32 pages of orders' history the following `for loop` extracts information for each sub page and then navigates to the next sub page of orders' history.


{% highlight r %}
payments <- list()
for(i in 1:32){
  remDr$findElements("class", "order-item-wraper") -> orders
  
  do.call(rbind,
    lapply(orders, function(element){
      element$findChildElement("class", "amount-num")$getElementText()[[1]] -> amount
      element$findChildElement("class", "product-action")$getElementText()[[1]] -> payed
      element$findChildElement("class", "info-body")$getElementText()[[1]] -> id
      data.frame(amount = amount,
                 payed = payed,
                 id = id)
  })) -> payments[[i]]
  
  remDr$findElement("class name", "ui-pagination-next")$clickElement()
  
  Sys.sleep(1);  cat(i, "\r")
}
{% endhighlight %}

# Plotting expenses

The extracted information is a plain text so it requires some text manipulation to achieve the tidy data, that can be plotted. Additionally I filter orders to those that haven't been cancelled.


{% highlight r %}
library(dplyr)
library(stringi)
do.call(rbind, payments) %>%
  filter(grepl('Open Dispute', payed)) %>% # 
  mutate(amount = as.numeric(unlist(
    stri_extract_all_regex(amount, "[0-9\\.]+")))) -> aliexpress
{% endhighlight %}


{% highlight r %}
head(aliexpress)
{% endhighlight %}



{% highlight text %}
  amount        payed             id
1   0.24 Open Dispute 81238841078231
2   1.46 Open Dispute 81235899648231
3   0.66 Open Dispute 81320512758231
4   1.07 Open Dispute 81192904428231
5   1.85 Open Dispute 81150862778231
6   0.61 Open Dispute 81116930588231
{% endhighlight %}

The plot of cumulative expenses can be obtained with the following code. I can't believe that 450 $ was spend over 8 months! The result of the code is the main photo of this post.


{% highlight r %}
library(ggthemr);ggthemr("camoflauge")
aliexpress %>%
   mutate(id = as.numeric(as.character(id))) %>%
   arrange(id) %>%
   mutate(cum_amount = cumsum(amount)) %>%
   ggplot(aes(id, cum_amount, group = 1)) + 
   geom_line() +
   theme(axis.ticks.x=element_blank(),
         axis.text.x=element_blank()) +
   labs(title = "Cumulative Ali Express expenses",
        subtitle = "web-harvested with RSelenium",
        caption = "source: r-addict.com") +
   ylab("Cumulative sum of spendings in $") + xlab(NULL)
{% endhighlight %}

