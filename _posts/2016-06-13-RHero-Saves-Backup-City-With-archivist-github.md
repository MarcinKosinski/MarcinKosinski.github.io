---
layout: post
title: R Hero saves Backup City with archivist and GitHub
comments: true
published: true
author: Marcin Kosiński
categories: [Reproducible Research]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/archivist_github.png" class="fit image">Have you ever suffered because of the impossibility of reproducing graphs, tables or analysis' results in R? Have you ever bothered yourself for not being able to share R objects (i.e., plots or final analysis models) within your reports, posters or articles? Or maybe simply you have too many objects you can’t manage to store in a convenient and handy way? Now you can share partial results of analysis, provide hooks to valuable R objects within articles, manage analysis' results and restore objects’ pedigree with [archivist](http://pbiecek.github.io/archivist/Posts.html) package and its extension [archivist.github](http://r-addict.com/archivist.github/), allautomatically through [GitHub](https://github.com/) without closing RStudio. If you are tired of archiving results by yourself, then read how you can became an R Hero with the [archivist.github](http://r-addict.com/archivist.github/) package power.

# R Hero archiving power

Recently I've visited Backup City, a data analysis mecca in the middle of Reproducible Research RLand. That's where I ovearheared a feverish discussion between R Hero and commissar O'Rdon. You can read the story of their meeting at the opening comic.



# archivist.gitub: archivist and GitHub integration

archivist.github is a package with tools for archiving, managing and sharing R objects via GitHub and is the extension of the archivist. You can install package from [CRAN](https://cran.r-project.org/web/packages/archivist.github/index.html)


{% highlight r %}
install.packages('archivist.github')
{% endhighlight %}

I have prepared a workflow graph to visualize functionalities of archivist.github

<img src="/images/fulls/archivist_github_workflow.png" class="fit image">

and provide explanation of core powers in this post.

After you've created a GitHub developer application (the process is described at [archivist.github: 2.1 OAuth open autorization](http://r-addict.com/archivist.github/#21_oauth_open_autorization), set: Homepage URL - http://github.com, Authorization callback URL - http://localhost:1410) you will be able to automatically create repositories on GitHub from R console.
Below is an example on how to authorise with GitHub API (using your application Client ID and Client Secret), create a GitHub repository with [archivist-like Repository](http://pbiecek.github.io/archivist/staticdocs/Repository.html) and automatically archive R object on GitHub


{% highlight r %}
# I saved some variables earlier 
# no to provide them publicly
load("github_token.rda");load("password.rda")

library(archivist.github)
# this can be done only in interactive mode
# so have to be done before knitr compilation
#
# authoriseGitHub(ClientID, ClientSecret,
# scope = c("public_repo", "delete_repo")) ->
# github_token 
#
# repository creation
createGitHubRepo(repo = "RHero", 
                 user = "archivistR", 
                 password = password,
                 github_token = github_token,
                 default = TRUE)
{% endhighlight %}



{% highlight text %}
[1] "archivistR"
{% endhighlight %}



{% highlight r %}
# -> https://github.com/archivistR/RHero
{% endhighlight %}

{% highlight r %}
# parameters can be set globally,
# so you will not have to specify 
# them for each call
aoptions("password", password)
aoptions("user", "archivistR") 
aoptions("repo", "RHero")
aoptions("github_token", github_token)
{% endhighlight %}


{% highlight r %}
# archiving on GitHub
archive(iris, alink = TRUE)
{% endhighlight %}

[`archivist::aread('archivistR/RHero/ff575c261c949d073b2895b05d1097c3')`](https://raw.githubusercontent.com/archivistR/RHero/master/gallery/ff575c261c949d073b2895b05d1097c3.rda)

<img src="/images/fulls/github_R.jpg" class="fit image">

One can check that the artifact is really on GitHub and that the [commit was performed](https://github.com/archivistR/RHero/commits/master) (with great help of [git2r](https://github.com/ropensci/git2r) package)


{% highlight r %}
# sometimes GitHub need more 
# time to react
Sys.sleep(60)
# show archived objects with their hashes
archivist::showRemoteRepo()
{% endhighlight %}



{% highlight text %}
                           md5hash                             name         createdDate
1 ff575c261c949d073b2895b05d1097c3                             iris 2016-06-13 16:41:17
2 2e7b44a1845602a5e3a4898b618b4aa6 2e7b44a1845602a5e3a4898b618b4aa6 2016-06-13 16:41:17
{% endhighlight %}



{% highlight r %}
# one can check how many commits have been performed so far
length(
   jsonlite::fromJSON(
      rawToChar(
httr::GET('https://api.github.com/repos/archivistR/RHero/commits')$content)
      )$sha
)
{% endhighlight %}



{% highlight text %}
[1] 2
{% endhighlight %}

Each object (referred as [artifact](http://pbiecek.github.io/archivist/staticdocs/archivist-package.html)) is archived with it's metadata and [md5hash](http://pbiecek.github.io/archivist/staticdocs/md5hash.html) in case someone would like to [restore or search for archived objects](http://r-addict.com/archivist.github/#23_archiving_and_exploring_example) within [Repository](http://pbiecek.github.io/archivist/staticdocs/Repository.html).

# Partial results archiving and objects’ pedigree restoration

[We](https://github.com/pbiecek/archivist/graphs/contributors) have prepared extended version of [pipe - %>%](https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html) operator [%a%](http://pbiecek.github.io/archivist/staticdocs/magrittr.html) so that every partial result of analysis workflow can be archived. Below is an example of workflow archiving for [RTCGA](http://rtcga.github.io/RTCGA/) (about which I wrote [here](http://r-addict.com/2016/05/04/RTCGA-Quick-Guide.html)) RNASeq data (genes' expression) (broader example can be find [here](http://r-addict.com/archivist.github/#3_advanced_example_with_rtcga)) and it's pedigree restoration


{% highlight r %}
# This sets `silent=TRUE` in saveToRepo
# which is used by %a% . There will be 
# no warning printed about archiving 
# the same artifact or it's data twice.
aoptions('silent', TRUE)
aoptions('repoDir', 'RHero')
 # %a% archives to Local Repos, that's
# why we need 'repoDir'
{% endhighlight %}

{% highlight r %}
#
# information about genes' expressions
library(RTCGA.rnaseq); data(BRCA.rnaseq)
library(dplyr)
BRCA.rnaseq %a%
    select(`TP53|7157`, 
           bcr_patient_barcode) %a%
    # bcr_patient_barcode contains a key to 
    # merge patients between various datasets
    rename(TP53 = `TP53|7157`) %a%
    filter(substr(bcr_patient_barcode, 
                  14, 15) == "01" ) -> 
    # 01 at the 14-15th position tells 
    # these are cancer sample
    BRCA.rnaseq.TP53
{% endhighlight %}


{% highlight r %}
# knitr: results='asis'.
# give hooks to objects
ahistory(BRCA.rnaseq.TP53,
         format = "kable",
         alink = TRUE ) 
{% endhighlight %}

|   |call                                                |md5hash                                                                                                                                    |
|:--|:---------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------|
|4  |env[[nm]]                                           |[63678e012c5b7f40966c32eec91f828b](https://raw.githubusercontent.com/archivistR/RHero/master/gallery/63678e012c5b7f40966c32eec91f828b.rda) |
|3  |select(`TP53&#124;7157`, bcr_patient_barcode)       |[4a85ce61229dd743b911d7edab0310b3](https://raw.githubusercontent.com/archivistR/RHero/master/gallery/4a85ce61229dd743b911d7edab0310b3.rda) |
|2  |rename(TP53 = `TP53&#124;7157`)                     |[103f2b82c41956e9f6437b3a0cd68679](https://raw.githubusercontent.com/archivistR/RHero/master/gallery/103f2b82c41956e9f6437b3a0cd68679.rda) |
|1  |filter(substr(bcr_patient_barcode, 14, 15) == "01") |[1da5a026aae19e0d0467ba3773679e28](https://raw.githubusercontent.com/archivistR/RHero/master/gallery/1da5a026aae19e0d0467ba3773679e28.rda) |


{% highlight r %}
# it uses global user and repo
# if they are not specified
{% endhighlight %}

Column with `[[env]]` is the object before transformations. We are working on using original names for objects in [this issue](https://github.com/pbiecek/archivist/issues/269).

This operation does not archive objects automatically on GitHub as this is functionality from base archivist package. One have to upload objects with


{% highlight r %}
pushGitHubRepo()
{% endhighlight %}

{% highlight r %}
# one can check how many commits have been performed so far
length(
   jsonlite::fromJSON(
      rawToChar(
httr::GET('https://api.github.com/repos/archivistR/RHero/commits')$content)
      )$sha
)
{% endhighlight %}



{% highlight text %}
[1] 3
{% endhighlight %}

# Overload `print()` to use `archive()`

After global parameters specification (aoptions() function sets 'user', 'repo', and 'password' parameters for each archivist.github and archivist function globally) we don't have to use `archive` function after each call to provide hooks in rmarkdown reports. We can overload `print()` function for specific classes so that after printing objects will be also evaluated with `archive` function.


{% highlight r %}
addHooksToPrintGitHub(class = "lm")
{% endhighlight %}


{% highlight r %}
# knitr: results='asis'
pld <- aread("MarcinKosinski/Museum/04eb0bdc12")
(lm.D9 <- lm(weight ~ group,
             data = pld))
{% endhighlight %}

Load: [`archivist::aread('archivistR/RHero/2b639023bc41e289aa21d790d5876736')`](https://raw.githubusercontent.com/archivistR/RHero/master/gallery/2b639023bc41e289aa21d790d5876736.rda)

Call:
lm(formula = weight ~ group, data = pld)

Coefficients:
(Intercept)     groupTrt  
      5.032       -0.371  



{% highlight r %}
(lm.D90 <- lm(weight ~ group - 1,
              data = pld))
{% endhighlight %}

Load: [`archivist::aread('archivistR/RHero/a33c804ff1d0b652210a39e2071d1e14')`](https://raw.githubusercontent.com/archivistR/RHero/master/gallery/a33c804ff1d0b652210a39e2071d1e14.rda)

Call:
lm(formula = weight ~ group - 1, data = pld)

Coefficients:
groupCtl  groupTrt  
   5.032     4.661  

This is the GitHub equivalent for local archiving with [addHooksToPrint](http://pbiecek.github.io/archivist/staticdocs/addHooksToPrint.html)

# [Feedback and Notes](http://marcinkosinski.github.io/archivist.github/#6_feedback_and_notes)

If you have any comments or user request, please see [Feedback and Notes](http://marcinkosinski.github.io/archivist.github/#6_feedback_and_notes) section to be aware of our future plans.

More examples can be checked at [archivist.github Tutorial](http://r-addict.com/archivist.github/) or you can learn more during [@pbiecek]() talk [How to use the archivist package to boost reproducibility of your research](http://schedule.user2016.org/event/7BYx/how-to-use-the-archivist-package-to-boost-reproducibility-of-your-research) at [useR2016 Conference](schedule.user2016.org/).

If you'd like to meet more R Heroes then restore message that was archived for commisar O'Rdon with

{% highlight r %}
# library(archivist.github)
cat(aread("MarcinKosinski/archivist.github/eRum2016/010e596"))
{% endhighlight %}



{% highlight text %}

European R users meeting (eRum 2016) will take place between October 12th and 14th.

We already have confirmed great invited speakers such as: Rasmus Bååth, Romain François, Ulrike Grömping, Matthias Templ, and Heather Turner, as well as strong representation from Poland: Przemysław Biecek, Marek Gągolewski, Jakub Glinka, Katarzyna Kopczewska, and Katarzyna Stąpor. We are planning a meeting of more than 200 useRs from all across Europe working in different areas of the industry, academy, and government.

On behalf of organising committee, chaired by Maciej Beręsewicz, we want to invite you to be a part of this historical meeting by proposing a workshop, submitting a regular or lightning talk, presenting a poster, or just attending the activities we are preparing for the meeting.

You will find more details about the registration process on the website www.erum.ue.poznan.pl.

If you have any questions do not hesitate to ask through erum@konf.ue.poznan.pl.

See you in Poznań.

Source: http://www.r-bloggers.com/european-r-users-meeting-meeting-of-r-heroes-poznan-12-14-10-2016/
{% endhighlight %}

<img src="/images/fulls/eRum.png" class="fit image">


# Thanks

Paintings were made by [pedzlenie](https://pedzlenie.github.io).
