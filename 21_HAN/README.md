1. What did the author(s) try to accomplish?

In the paper, the authors design a hierarchical attention mechanism for documentation classification. One simple illustration is that when doing sentiment analysis, say for tweets, what makes the predictor do the classification should not equally weighted across each word, and the predictor should pay more attention to words or sentences server as stronger indicator. 

2. What were the key elements of the approach(s)?

Based on the *Attention Mechanism* in previous work, this paper designed a hierarchical network in which the first block looks at the word embedding and performs attention algorithm, then using the output of the first block, the second block performs attention mechanism towards sentence level. The encoders are all *GRU* in each of the above block. 


3.  List the concepts/techniques that are new to me and highlight whatever I feel necessary to put into my skill set if there is any.

 - Most important of all, revisit *Attention Mechanism*. This time, I finally figured it out what it's going and why it makes improvement. 
 - Documentation Classification
 - Sentiment Analysis


4.  Thoughts and questions

 - I had really bad experience struggling with the concepts and rationale and everything behind what's going on with those text encoders such GRU. Because of those I almost gave up for this field. But when I revisit them, I feel it not that necessary to get to know 100% from the idea to derivation if just looking for application. Not to say that's unnecessary at all, I mean I still got to know those things much better than those who are new to the field. But as long as having a sense of general idea and concepts, it would be okay to step forward and let those were abstract be digested gradually. 
 - One technical part of confusion is that I did not feel clear about the hierarchical structure from the words but understand that from figures in this paper. Perhaps it would be better if the paper explain a little more about how the word-level attention connect with sentence-level encoder. 
