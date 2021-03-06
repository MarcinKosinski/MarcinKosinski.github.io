---
layout: post
title: When You Went too Far with Survival Plots During the survminer 1st Anniversary
comments: true
published: true
date: 2017-01-15 11:00:00
author: Marcin Kosiński
categories: [Biostatistics]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/uber-survminer.png" class="fit image"> We are celebrating the 1st anniversary of the survminer's release on CRAN! Due to that fact I have prepared the most (uber platinum) customized survival plot that I could imagine. I went too far because it took over 30 parameters to create a graph.. 

[survminer](https://cran.r-project.org/web/packages/survminer/index.html), a package for drawing easily beautiful and 'ready-to-publish' survival curves with the 'number at risk' table and 'censoring count plot', has became very popular at the end of a previous year. It happend due to 3 great methodology posts about Cox Proportional Hazards model and Survival Analysis basics by Alboukadel Kassambara ([kassambara](https://github.com/kassambara/)) published on [STDHA](http://www.sthda.com/english/)

- [Survival Analysisc Basics](http://www.sthda.com/english/wiki/survival-analysis-basics)
- [Cox Proportional Hazards Model](http://www.sthda.com/english/wiki/cox-proportional-hazards-model)
- [Cox Model Assumptions](http://www.sthda.com/english/wiki/cox-model-assumptions)

If you are working in the survival analysis field, you might be interested in reading the post covering the newest features in survminer: [survminer 0.2.4](http://www.sthda.com/english/wiki/survminer-0-2-4)

Right now, after a year of the first CRAN release, survminer has been downloaded [11k times](https://cranlogs.r-pkg.org/badges/grand-total/survminer) and has been used and cited in several research articles:

- [Primary resistance to PD-1 blockade mediated by JAK1/2 mutations](http://cancerdiscovery.aacrjournals.org/content/candisc/early/2016/12/01/2159-8290.CD-16-1223.full.pdf)
- [Methods for Evaluating Respondent Attrition in Web-Based Surveys](https://www.jmir.org/2016/11/e301)
- [Antiretroviral therapy improves survival among TB-HIV co-infected patients who have CD4+ T-cell count above 350cells/mm3](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5067898/#CR18)
- [The serological response against foot and mouth disease virus elicited by repeated vaccination of dairy cattl](https://www.ncbi.nlm.nih.gov/pubmed/27576078)
- [The serological response against foot and mouth disease virus elicited by repeated vaccination of dairy cattle](https://www.researchgate.net/publication/307559129_The_serological_response_against_foot_and_mouth_disease_virus_elicited_by_repeated_vaccination_of_dairy_cattle)
- [ActiviTeen: A Protocol for Deployment of a Consumer Wearable Device in an Academic Setting](ActiviTeen: A Protocol for Deployment of a Consumer Wearable Device in an Academic Setting)
 
Due to the celebration of it's 1st anniversary on CRAN, I have prepared a long survival graph of Kaplan-Meier estimates of survival curves, where each option is described below. I think I went too far to present the possible customization of the graph because this example takes over 30 parameters!

<p>
   <script src="https://gist.github.com/MarcinKosinski/47b0a38a4c812b66b324255e4a9877ba.js"></script>
  <noscript>
    <pre>
       {% highlight r %}
   ggsurvplot(
   fit,                     # survfit object with calculated statistics.
   risk.table = TRUE,       # show risk table.
   pval = TRUE,             # show p-value of log-rank test.
   conf.int = TRUE,         # show confidence intervals for 
                            # point estimaes of survival curves.
   xlim = c(0,500),         # present narrower X axis, but not affect
                            # survival estimates.
   xlab = "Time in days",   # customize X axis label.
   break.time.by = 100,     # break X axis in time intervals by 500.
   ggtheme = theme_light(), # customize plot and risk table with a theme.
  risk.table.y.text.col = T,# colour risk table text annotations.
  risk.table.y.text = FALSE,# show bars instead of names in text annotations
                            # in legend of risk table.
  ncensor.plot = TRUE,      # plot the number of censored subjects at time t
  conf.int.style = "step",  # customize style of confidence intervals
  surv.median.line = "hv",  # add the median survival pointer.
  legend.labs = 
    c("Male", "Female"),    # change legend labels.
  palette = 
    c("#E7B800", "#2E9FDF"),# custom color palettes.
  main = "Survival curves",                       # specify the title of the plot
  submain = "Based on Kaplan-Meier estimates",    # the subtitle of the plot 
  caption = "created with survminer",             # the caption of the plot
  font.main = c(16, "bold", "darkblue"),          # font for titles of the plot, the table and censor part
  font.submain = c(15, "bold.italic", "purple"),  # font for subtitles in the plot, the table and censor part
  font.caption = c(14, "plain", "orange"),        # font for captions in the plot, the table and censor part
  font.x = c(14, "bold.italic", "red"),           # font for x axises in the plot, the table and censor part
  font.y = c(14, "bold.italic", "darkred"),       # font for y axises in the plot, the table and censor part
  font.tickslab = c(12, "plain", "darkgreen"),    # font for ticklabs in the plot, the table and censor part
  ########## risk table #########,
  risk.table.title = "Note the risk set sizes",          # the title of the risk table
  risk.table.subtitle = "and remember about censoring.", # the subtitle of the risk table
  risk.table.caption = "source code: website.com",       # the caption of the risk table
  risk.table.height = 0.35,                              # the height of the risk table
  ########## ncensor plot ######
  ncensor.plot.title = "Number of censorings",           # as above but for the censoring plot
  ncensor.plot.subtitle = "over the time.",
  ncensor.plot.caption = "data available at data.com",
  ncensor.plot.height = 0.35)
       {% endhighlight %}
    </pre>
  </noscript>
</p>
