---
layout: post
title: Why R? Webinar - Recent changes in R spatial and how to be ready for them
comments: true
published: true
date: 2020-04-21 13:00:00
author: Marcin Kosiński
categories: [webinars]
tags: [2020, Webinars]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/webinars/spatial.jpg" class="fit image"> April 23rdth (8:00pm GMT+2) is the next time we'll be hosting a webinar at Why R? Foundation YouTube channel - https://youtube.com/c/WhyRFoundation This time we will have a joint presentation by Robin Lovelace and Jakub Nowosad (authors of [Geocomputation with R](https://geocompr.robinlovelace.net/)). Jakub is an assistant professor in the Institute of Geoecology and Geoinformation at the Adam Mickiewicz University, Poznan, Poland. He is a computational geographer working at the intersection between geocomputation and the environmental sciences. Robin is associate professor at the Institute for Transport Studies (ITS) and Leeds Institute for Data Analytics (LIDA), University of Leeds, UK. His research focuses on geocomputation and reproducible data science for evidence-based policy-making.

Full abstract and biograms are below.

See you on the [Webinar](https://youtu.be/Va0STgco7-4)!

# Details

- donate: [whyr.pl/donate/](https://whyr.pl/donate/)
- channel: [youtube.com/c/WhyRFoundation](https://www.youtube.com/c/WhyRFoundation)
- date: every Thursday 8:00 pm GMT+2
- format: one 45 minutes long talk streamed on YouTube + 10 minutes for Q&A 
- comments: ask questions on YouTube live chat

# Abstract

Currently, hundreds of R packages are related to spatial data analysis. They range from ecology and earth observation, through hydrology and soil science, to transportation and demography. These packages support various stages of analysis, including data preparation, visualization, modeling, or communicating the results. One common feature of most R spatial packages is that they are built upon some of the main representations of spatial data in R, available in the sf, sp, raster, terra, or stars package. Those packages are also not entirely independent. They are using external libraries, namely GEOS for spatial data operations, GDAL for reading and writing spatial data, and PROJ for conversions of spatial coordinates.

Therefore, R spatial packages are interwoven with each other and depend partially on external software developments. This has several positives, including the ability to use cutting-edge features and algorithms. On the other hand, it also makes R spatial packages vulnerable to changes in the upstream packages and libraries.

In the first part of the talk, we will showcase several recent advances in R packages. It includes the largest recent change related to the developments in the PROJ library. We will explain why the changes happened and how they impact R users.
The second part will focus on how to prepare for the changes, including computer set-up and running R spatial packages using Docker. We will outline important considerations when setting-up operating systems for geographic R packages. To reduce set-up times you can use geographic R packages Docker, a flexible and scalable technology containerization technology. Docker can run on modern computers and on your browser via services such as Binder, greatly reducing set-up times. Discussing these set-up options, and questions of compatibility between geographic R packages and paradigms such as the tidyverse and data.table, will ensure that after the talk everyone can empower themselves with open source software for geographic data analysis in a powerful and flexible statistical programming environment.

# Biograms

Jakub is an assistant professor in the Institute of Geoecology and Geoinformation at the Adam Mickiewicz University, Poznan, Poland. He is a computational geographer working at the intersection between geocomputation and the environmental sciences. His research has focused on developing and applying spatial methods to broaden our understanding of processes and patterns in the environment. Another vital part of his work is to create, collaborate on, and improve geocomputational software. He is a co-author of the Geocomputation with R book and many R packages, including landscapemetrics, sabre, and climate.

Robin is associate professor at the Institute for Transport Studies (ITS) and Leeds Institute for Data Analytics (LIDA), University of Leeds, UK. His research focuses on geocomputation and reproducible data science for evidence-based policy-making. Decarbonising the global economy while improving health and environmental outcomes is a major problem solving challenge. Robin’s research supports solutions by generating evidence and tools enabling evidence-based investment in efficient and healthy modes of transport at local, city and national scales. Robin is the Lead Developer of the award-winning Propensity to Cycle Tool (see www.pct.bike ), convenor of the Transport Data Science module and workshop series, and co-author of popular packages, papers, and books, including Geocomputation with R.


# Previous talks

<img src="/images/fulls/webinars/heidi.jpg" class="fit image">

[Heidi Seibold](https://www.researchgate.net/profile/Heidi_Seibold), [Department of Statistics (collaboration with LMU Open Science Center)](https://statsatlmu.tumblr.com/) (University of Munich) - **Teaching Machine Learning online**. [Video](https://www.youtube.com/watch?v=jPQJTVa-GsQ)

<img src="/images/fulls/webinars/olgun.jpg" class="fit image">

[Olgun Aydin - PwC Poland](https://www.linkedin.com/in/olgun-aydin/) - **Introduction to shinyMobile**. [Video](https://www.youtube.com/watch?v=TJsu0S9_WY4)

<img src="/images/fulls/webinars/achim.jpg" class="fit image">

[Achim Zeileis from Universität Innsbruck](https://eeecon.uibk.ac.at/~zeileis/) - **R/exams: A One-for-All Exams Generator - Online Tests, Live Quizzes, and Written Exams with R**. [Video](https://www.youtube.com/watch?v=PnyCR7q4P4Q)


# Stay up to date

- subscribe to YouTube channel [youtube.com/c/WhyRFoundation](https://www.youtube.com/c/WhyRFoundation)
- join Why R? Slack [whyr.pl/slack/](http://whyr.pl/slack/)
- join [Meetup](https://www.meetup.com/Spotkania-Entuzjastow-R-Warsaw-R-Users-Group-Meetup/events/269589118/)
