---
date: 2024-03-04
tags:
  - video
hubs:
  - "[[machine-learning]]"
urls:
  - https://www.youtube.com/watch?v=9vBRjGgdyTY
  - https://github.com/logicalclocks/hopsworks-tutorials/tree/master/advanced_tutorials/recommender-system
  - https://app.hopsworks.ai/
---

# Real Time Search and Recommendation at Scale Using Embeddings and Hopsworks

- The road to value with data and AI
    - Get data warehouse in order, run some BI analysis, build play models
    - Create batch prediction service
        - e.g. Spotify discover weekly
    - Create streaming prediction service
        - e.g. tiktok

- Well known online recommender services
    - Collaborative filtering batch
        - Facebook posts
        - Netflix movies
        - Spotify weekly
    - Real-time
        - Alibaba products
        - Youtube clips
        - Tiktok feed

- Batch architecture
    - Raw data comes from feature store
    - Batch program to fetch model from model registry and compute preidctions
    - Upload predictions to database
    - Served for reports / users

- Stream architecture
    - Offline API for training
        - Pipelines to feed the feature store and keep it updated
            - e.g. User data, items available, transactions
            - Sourced from various places, e.g. data warehouse, kafka transactions, etc..
        - Requires logging of user behaviour
    - Online API for serving

- Embeddings
    - Map a variable to a fixed-size vector (it's representation in the embedding's space)

- Ranking and retrieval
    - User query embedding: has information about search term (e.g. red dress) and user, e.g. age, location ,etc.. and recent items viewed / purchases
    - Retrieve candidates using two-tower network architecture (compare with item embeddings)
    - Enrich candidates with feature store (needs to be quick, feature store should therefore be in-memory database)
    - Batch scoring of candidates using ranking model
    - Log outcomes back to feature store (what users clicked on, what they didn't)



