---
date: 2024-07-21
tags:
    - podcast
hubs:
    - "[[machine-learning]]"
---

# MLOps Coffee Sessions 83 with Vincent Warmerdam   Better Use cases for Text Embeddings

Start simple. Build a model with simple if/else logic before building an ML model.

It helps to consider two models.

Let's say we have one model that is the very complex model. It's like the gradient boosted tree, deep learning, hyperparameter tuning, etc..

And then we have another model, which we'll just consider to be the relatively simple model. So, you know, maybe logistic regression, or like something with heuristics that I just mentioned, something with very much domain knowledge.

Now, what's very interesting to do is to train those two models and to see where they disagree. Like just look at the examples where both models are reasonably good, that both models have like a way of working and they arguably are not bad. But if two models that are good disagree, something interesting is happening.

And sometimes it's because you find some bad labels, which are good to fix, but it's also sometimes that the deep learning model has figured out something that my rule-based system hasn't.

Sometimes you can also look at the data and sort of say, oh, wait, there's another heuristic that I can add to my rule-based system.

By that time, you'll have better labels, you'll have better rules, and you'll have version two of your deep learning system and version two of your heuristics. And from there, you can once again see where they disagree.

And I've just noticed this is way better than grid search, just in terms of like getting a proper model that actually can do something meaningful in a business. Like these kinds of approaches, where it's a little bit more iterating on data.








