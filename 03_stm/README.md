1. What did the author(s) try to accomplish?

To incorporate corpus structure or document metadata into the standard topic model affecting both topic prevalence (the frequency with which a topic is discussed) and topical content (the words used to discuss a topic)

2. What were the key elements of the approach(s)?

- The central idea is to specify the priors as generalized linear models through which we can condition on arbitrary observed data. This allows us to directly estimate the quantities of interest in applied problems
- The approach to including corpus structure reflects their interest in making inference about *observed* covariates rather than predicting covariate values in unseen text

3.  List the concepts/techniques that are new to me and highlight whatever I feel necessary to put into my skill set if there is any.

A bunches of different kinds of topic models and prior distributions (the way that data/parameters are sampled)

4.  Thoughts and questions

Honestly I don't fully understand what this paper is talking about, especially the rationales (like why this prior, why that topic model, what's the difference). But obviously the authors have combined everything into their *R* package so I feel it's not necessary to be expert at (most of) concepts in this paper and the essiential thing is that this method allows us to flexibly estimate a topic model that includes document-level meta-data and estimation is accomplished through a fast variational approximation, according to the offical [R document](https://cran.r-project.org/web/packages/stm/vignettes/stmVignette.pdf). I don't think it's worth that much time to dig into too detailed since apparently the paper itself has only 4 pages.