1. What did the author(s) try to accomplish?

 Get better word representation considering context. Make it adaptive to series downstream task.

2. What were the key elements of the approach(s)?

 - For the encoder part, using *transformers*
 - For the fine-tuing part, using two sub-tasks: *Masked LM* and *Next Sentence Prediction*
 - In this way, it not only enables parallel training with multi-head attention mechanism, but also considers bi-directional information without revealing to each other.


3.  List the concepts/techniques that are new to me and highlight whatever I feel necessary to put into my skill set if there is any.

- One concept that needs to be brought up for some mix confusion: *fine-tuning* method in this paper represent tuning all parameters in the downstream task including all the weights from BERT, while *feature-based* method refers to only tune the parameters in downstream tasks, since the word representations are just the outputs from last layer of BERT and they are fixed. 

4.  Thoughts and questions

 - Apparently it is the new "ImageNet" in NLP field as it says. One thought of mine is that there are a lot more can be studying on in terms of using the information from which layer? As far as I know, the base BERT has 12 layers and each of them contains some information. I recall that there are people finding that different combination of layer outputs can benefit different tasks depending on situations. So I'm wondering if there could be some standardization on how to find these combinations or just mindlessly trying.