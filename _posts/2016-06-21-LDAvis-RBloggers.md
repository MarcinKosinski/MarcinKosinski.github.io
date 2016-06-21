---
layout: post
title: LDAvis Show Case on R-Bloggers
comments: true
published: true
author: Marcin Kosiński
categories: [Text Mining]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/LDAvis.png" class="fit image">Text mining is a new challenge for machine wandering practitioners. The increased interest in the text mining is caused by an augmentation of internet users and by rapid growth of the internet data which is said that *in 80% is a text data*. Extracting information from articles, news, posts and comments have became a desirable skill but what is even more needful are tools for text mining models diagnostics and visualizations. Such visualizations enable to better understand the insight from a model and provides an easy interface for presenting your research results to greater audience. In this post I present the Latent Dirichlet Allocation text mining model for text classification into topics and a great [LDAvis](https://cran.r-project.org/web/packages/LDAvis/index.html) package for interactive visualizations of topic models. All this on [R-Bloggers](http://r-bloggers.com/) posts!

LDAvis is not my package. It is created by [cpsievert](https://github.com/cpsievert) and this post's code for LDAvis-preparations is mostly based on his LDAvis tutorial: [A topic model for movie reviews](http://cpsievert.github.io/LDAvis/reviews/reviews.html)

# LDA overview and text preparation

From [Wikipedia](https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation)

> In natural language processing, latent Dirichlet allocation (LDA) is a generative statistical model that allows sets of observations to be explained by unobserved groups that explain why some parts of the data are similar. For example, if observations are words collected into documents, it posits that each document is a mixture of a small number of topics and that each word's creation is attributable to one of the document's topics.

For this post I have used articles from [R-Bloggers](http://r-bloggers.com/). 
They can be downloaded from [this repository](https://github.com/MarcinKosinski/r-bloggers-harvesting). The data harvesting process is explained at the end of this post. 


{% highlight r %}
library(RSQLite)
db.conn <- 
  dbConnect(
    dbDriver("SQLite"),
    "r-bloggers.db"
)
posts <- dbGetQuery(db.conn, 
           "SELECT text from posts")
dbDisconnect(db.conn)
{% endhighlight %}

Normally I would use `LDA()` function from [topicmodels](https://cran.r-project.org/web/packages/topicmodels/topicmodels.pdf) package to fit LDA model because the input can be of class `DocumentTermMatrix` which is an object from [tm](https://cran.r-project.org/web/packages/tm/vignettes/tm.pdf) package. `DocumentTermMatrix` object is very convinient for working with text data ([check this Norbert Ryciak's post](http://www.rexamine.com/2014/06/text-mining-in-r-automatic-categorization-of-wikipedia-articles/)) because there exists `tm_map` function which can be applied to all documents for stop words removal, lowering capital letters and removal of words that did not occur in x % of documents. I haven't seen `LDAvis` examples for models generated with topicsmodel package so we will use traditional approach to text processing. The [stemming](https://en.wikipedia.org/wiki/Lemmatisation) and stopwords removal was perfored during the data collection, which is described at the end of the post.


{% highlight r %}
## the following fragment of code in this section
## is motivated by
## http://cpsievert.github.io/LDAvis/reviews/reviews.html
# tokenize on space and output as a list:
doc.list <- strsplit(posts[, 1], "[[:space:]]+")
# compute the table of terms:
term.table <- table(unlist(doc.list))
term.table <- sort(term.table, decreasing = TRUE)
# remove terms that occur fewer than 5 times:
term.table <- term.table[term.table > 5]
vocab <- names(term.table)
{% endhighlight %}

The `lda.collapsed.gibbs.sampler()` function from `tm` package has uncomfortable input format (regarding to `LDA()` from topicmodels package) so I basically used [cpsievert](https://github.com/cpsievert) snippets


{% highlight r %}
# now put the documents into the format required by the lda package:
get.terms <- function(x) {
  index <- match(x, vocab)
  index <- index[!is.na(index)]
  rbind(as.integer(index - 1), as.integer(rep(1, length(index))))
}
documents <- lapply(doc.list, get.terms)

# Compute some statistics related to the data set:
D <- length(documents)  # number of documents (3740)
W <- length(vocab)  # number of terms in the vocab (18,536)
doc.length <- sapply(documents, 
                     function(x) sum(x[2, ]))  
# number of tokens per document [312, 288, 170, 436, 291, ...]
N <- sum(doc.length)  # total number of tokens in the data (546,827)
term.frequency <- as.integer(term.table)  
# frequencies of terms in the corpus [8939, 5544, 2411, 2410, 2143, ...]
{% endhighlight %}

Fitting the model: from `tm` package documentation

> ... [this function] takes sparsely represented input documents, perform inference, and return point estimates of the latent parameters using the state at the last iteration of Gibbs sampling. 


{% highlight r %}
# MCMC and model tuning parameters:
K <- 20
G <- 5000
alpha <- 0.02
eta <- 0.02

# Fit the model:
library(lda)
set.seed(456)
t1 <- Sys.time()
fit <- lda.collapsed.gibbs.sampler(
  documents = documents, K = K, 
  vocab = vocab, num.iterations = G, 
  alpha = alpha, eta = eta, 
  initial = NULL, burnin = 0,
  compute.log.likelihood = TRUE
)
t2 <- Sys.time()
t2 - t1  # about 7 seconds on my laptop

library(archivist)
saveToRepo(fit,
           repoDir = "../Museum")
{% endhighlight %}

The computations took very long, so in case you would like to 
get model faster, I have archived my model on GitHub with the help of
[archivist](http://r-bloggers.com/r-hero-saves-backup-city-with-archivist-and-github/) package.
You can easily load this model to R with

{% highlight r %}
archivist::aread('MarcinKosinski/Museum/fa93abf0ff93a7f6f3f0c42b7935ad4d') -> fit
{% endhighlight %}


# LDAVis use case

If you google out properly you'll wind out that LDAvis description is

> Tools to create an interactive web-based visualization of a topic model that has been fit to a corpus of text data using Latent Dirichlet Allocation (LDA). Given the estimated parameters of the topic model, it computes various summary statistics as input to an interactive visualization built with D3.js that is accessed via a browser. The goal is to help users interpret the topics in their LDA topic model.

[Detailed vignette about LDAvis input and output can be found here](https://cran.r-project.org/web/packages/LDAvis/vignettes/details.pdf).

> To visualize the result using LDAvis, we'll need estimates of the document-topic distributions, which we denote by the DxK matrix theta, and the set of topic-term distributions, which we denote by the K×W matrix phi. 


{% highlight r %}
theta <- t(apply(fit$document_sums + alpha,
                 2,
                 function(x) x/sum(x)))
phi <- t(apply(t(fit$topics) + eta,
               2,
               function(x) x/sum(x)))
{% endhighlight %}


{% highlight r %}
library(LDAvis)
# create the JSON object to feed the visualization:
json <- createJSON(
  phi = phi, 
  theta = theta, 
  doc.length = doc.length, 
  vocab = vocab, 
  term.frequency = term.frequency
)
serVis(json, out.dir = 'vis', 
       open.browser = FALSE)
{% endhighlight %}

The result is published under this link [http://r-addict.com/r-bloggers-harvesting/](http://r-addict.com/r-bloggers-harvesting/) where you can check Intertopic Distance Map (via multidimensional scaling) and top N relevant terms for a topic.


# Data Harvesting

Below is the code I have used for R-Bloggers web-scraping. At start I have extracted all links to posts
from first 100 main pages of R-Bloggers. Then I have created an SQLite database with empty table called `posts`.
This table has been used to store information like: post title, post link, post author, date of publication and the whole post text. For the text I have extracted only words (with [stringi](http://www.gagolewski.com/software/stringi/) package) that have length greater than 1 and applied `tolower` function to get rid of capital letters. Stop words removal was done thanks to `tm::removeWords()`. For [stemming](https://en.wikipedia.org/wiki/Stemming) I have used [RWeka::LovinsStemmer](https://cran.r-project.org/web/packages/RWeka/RWeka.pdf). I did not perform full lemmatization as I have found it troubling in R (couldn't install [this](http://stackoverflow.com/questions/28214148/how-to-perform-lemmatization-in-r) and [this](http://stackoverflow.com/questions/22993796/lemmatizer-in-r-or-python-am-are-is-be) took too long).



{% highlight r %}
library(rvest)
library(pbapply)
pbsapply(1:100, function(i){
  read_html(paste0('http://www.r-bloggers.com/page/', i)) %>%
  html_nodes('h2 a') %>%
  html_attr('href') %>%
  grep('r-bloggers', x = ., value = TRUE)
}) %>% 
  as.vector() %>%
  unique -> links


# create connection
library(RSQLite)
db.conn <- 
  dbConnect(
    dbDriver("SQLite"),
    "r-bloggers.db"
)
# create empty table for posts
posts <- 
  data.frame(link = "",
             title = "",
             # shares = "",
             text = "",
             date = "",
             author = "")
dbWriteTable(
  db.conn,
  name = "posts",
  posts,
  overwrite = TRUE,
  row.names = FALSE
) 

# function for info extraction
library(stringi)
library(tm)
library(RWeka)
postInfo <- function(link, conn) {
  post <- read_html(link)
  title <- html_nodes(post, 'h1') %>%
    html_text() %>%
    .[1] %>%
    gsub("\"", "", x = .) 
  # shares <- html_nodes(post,
  #   '.wpusb-counts , .wpusb-counts span') %>%
  #   html_text()
  text <- html_nodes(post, 'p') %>% 
    html_text() %>% head(-1) %>%
    paste(collapse=" ") %>%
    gsub("\"", "", x = .) %>%
    stri_extract_all_words()  %>% 
    .[[1]] 
  text <- text[stri_length(text) > 1] %>%
    tolower() %>%
    gsub("\\.", " ", x = .) %>%
    removeWords(c(stopwords('english'),
                  stopwords('SMART'),
                  "", "’s", "amp", 
                  "div", "’ve", 1:100, 
                  "’ll", "let’s", "’re", 
                  "’m")) 
  text <- text[stri_length(text) > 1] %>%
    stri_extract_all_words() %>% 
    unlist %>%
    LovinsStemmer(control = NULL) %>%
    paste(collapse = " ")
  
  date <- html_nodes(post, '.date') %>%
    html_text() %>%
    as.Date(format="%b %d, %Y")
  if (is.na(date)) {
    date <- ""
  }
  author <- html_nodes(post, '.meta') %>% 
    html_text() %>% strsplit("By ") %>% 
    # second element of length-1 list
    .[[1]] %>% .[2] %>%
    gsub("\"", "", x = .)
 
  dbGetQuery(
conn,
paste0("INSERT INTO posts
(link,title, text, date, author) VALUES",
"(\"", link, "\",\"", title, "\",\"", #shares, "\",\"",
text, "\",\"", date, "\",\"", author, "\")")
  )   
}

# for proper date extraction
lct <- Sys.getlocale("LC_TIME")
Sys.setlocale("LC_TIME", "C")
# get info for all links
pbsapply(
  links,
  postInfo, 
  db.conn
)
Sys.setlocale("LC_TIME", lct)
# disconnect with database
dbDisconnect(db.conn)
{% endhighlight %}

If you have any questions or comments, please feel free to share your ideas on the Disqus panel below.
Also, if you know how to web-scrap the number of shares per R-Bloggers article then I would love to hear your feedback as I am wondering what's the correlation between `Hadley Wickham` appearance in the post and its number of shares.

