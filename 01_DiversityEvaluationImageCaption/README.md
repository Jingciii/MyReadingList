1. What did the author(s) try to accomplish?

The evaluation for image captioning should be based on two dimension: accuracy and diversity. With the surge of Deep Learning, there are lots of approaches reached state-of-the-art performances in terms of accuracy. As for diversity, however, there is not that many papers working on this part, and there is not a proper way evaluating the diversity. Since an image is worth a thousand words, diversity is an important indication for models to be human-like. In this paper, the authors define a new evaluation metric for diversity that is more correlated with human judgment comparing to existing method.

2. What were the key elements of the approach(s)?

- Use LSA method, in representing matrix for caption set, let the singular values indicate the strength of each latent topic (so it is based on pairwise similarity rather than one vs. the aggregation of the rest). If all the singular values are in similar scales, captions in this set should be considered different from each other, while if all the singular values are zeros except the first one, then only one topic dominates the whole set resulting in less diversity.

- Kernelize the above method using CIDEr score so that it considers phrases and sentence structures


3. List the concepts/techniques that are new to me and highlight whatever I feel necessary to put into my skill set if there is any.

- *Image Captioning* which is not introduced in too much depth in this paper, but it would be good to be familiar with
- *Reinforcement Learning* which I've known but not been familiar with for years

4. Thoughts and questions

- I think the way the authors use to evaluate diversity is pretty interesting to me. As I've played around with PCA, Topic Modeling and a lot of things involving matrix decomposition, it feels putting all the maps together when I saw it uses strength of latent topic to denote diversity across a set of captions
- I feel the overall ideas of this paper are very clear. But since the authors tried different combinations of models settings and generators in an extensive experiment (it's not a bad thing though), when I was doing presenting and also for the preparation, I kept feeling things not strict enough. I might be wrong, but I think if using less number of models would make things clearer.