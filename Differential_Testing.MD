# Differential Testing -  "good for testing unknowed unknowns" 

Differential testing (also known as "back to back" testing) - A type of testing that compares the differences in execution from one ML model version to the next when the inputs are the same.  These are things like bugs in our feature engineering code which result in significantly poor performance but which don't raise any particular exceptions.**For example:** if I make an error in my config and forget to include a particular feature if it's an important feature that's gonna really reduce the effectiveness of my model.But the pipeline will still run there'll be no problem with training and there'll be no problem withactually calling the prediction.So a traditional software test would assume that everything was working fine. 

Tuning your differential tests refers to the sensitivity of precision how big a change from one version to another is a problem. If you're working with a well-established model and you're expecting that a new model is going to make a subtle small improvement then any significant change detected in a differential test would be cause for concern. On the other hand if you're prototyping or introducing a radically different model perhaps then you're going to tune your differential test to be more flexible depending on our implementation.

Differential test can fall into different test categories: 
  - if we're running them via some kind of continuous integration job on a fully setup up sandbox testing environment then they could be considered end to end tests 
  - if they're operating on multiple docker containers or across many different components they could be integration tests but it is also possible to run them as simply unit tests checking against a specific function call to get predictions from our model given the same input arguments
  - could unit tests
