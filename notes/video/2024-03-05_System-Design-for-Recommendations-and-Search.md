---
date: 2024-03-05
tags:
    - video
hubs:
    - "[[machine-learning]]"
urls:
    - https://www.youtube.com/watch?v=lh9CNRDqKBk
---

# System Design for Recommendations and Search

## Benefits

- Batch benefits
    - Pre-computed
    - Compute and serving decoupled
    - Lower operational load

- Real time benefits
    - Responsive to time-sensitive content (better UX)
    - No wasted cost on non-visitng users

## Aspects

- Offline aspect
    - Training, index/graph building
    - Loading data into feature stores

- Online aspect
    - Use artifacts from offline environment to serve requests
    - Need to do candidate retrieval and ranking

## Retreival VS ranking

- Retrieval
    - Fast but coarse
    - Filters millions of items down to hundreds
    - Approx nearest neighbors (ANN), graphs, etc..

- Ranking
    - Slower but more precise
    - Ranks hundreds of candidates
    - Involves more features

## Offline (batch) aspect

- Offline retrieval
    - Train embedding model
    - Compute embeddings of items from catalog
    - Build ANN index

- Offline ranking
    - Build feature store (e.g. item, transactions, user data)
    - Train ranking model (given item and user, what is probability that user will click on or purchase item)

- Result
    - Artifacts for real-time prediction:
        - embedding model
        - ANN index of embedded catalog items
        - feature store of item, tx and user data
        - ranking model

## Online example

- User searches "the matrix"
- Compute embedding of query
- Look up against ANN catalog of movie embeddings to get top k candidates
- Add features to candidates from feature store
- Rank top k candidates using ranking model

![2024-03-05-738.png](assets/imgs/2024-03-05-738.png)


## When to use real time?

- Start with batch because it's relatively easy
- When you need recommendations to react quickly to changes
- Behaviours that don't change quickly dont need real time
    - e.g. grocery selection, clothing preference


## Building an MVP from scratch (simple)

- Use word-to-vec to create embeddings ("self-supervised")
    - e.g. Alibaba's item-to-vec: embed the results of random walks on an item graph construction based on user behaviour with those items
- Put embeddings in approximage nearest neighbors ANN index
- Use ANN to retreive candidates
- Combine user and item features into logistic regression model for ranking
- Serve with SageMaker (can be load balanced as you scale)



