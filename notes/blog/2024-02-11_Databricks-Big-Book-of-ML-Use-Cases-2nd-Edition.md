---
date: 2024-02-11
tags:
    - blog
hubs:
    - "[[machine-learning]]"
---

# Databricks Big Book of ML Use Cases 2nd Edition


# Out-of-stock Modeling
- Challenges of retailers
    - Phantom inventory
        - Inventory is reported as being non-zero, but sales are zero - indicating there is no inventory in reality
    - Safety stock violations
        - When stock levels reach some threshold, new stock is ordered to replenish. This may not be an effective threshold (i.e. not early enough)
    - Zero-sales events
        - Determine if probability of consecutive zero-sales days is high for indicating phantom inventory, or reflective of another problem, rather than regular shopper behavior
    - On-shelf availability
        - Compare actual to expected sales units over time. If graph looks different this could indicate that inventory is not being displayed as expected

# Detecting financial fraud at scale with decision trees and MLflow
- Common pattern is to train a model and "express it" as a set of SQL conditions that are transparent in their logic
- Data engineer / data analyst / data scientist teams are often siloed
- Synthetic labels can be generated using rules determined by SMEs in order to create a benchmark model on sufficiently large data and identify valuable features
- Decision tree attractive model because of explainability, and it tends to perform great at expressing rules (i.e. rules used to create synthetic data)
- Data loaded is accessed using SQL in Databricks notebook, and converted to pyspark dataframe for ML
- Using confusion matrix to analyze - especially interested in false negatives (when you predict it is not fraud, but it is)
- Downsampling negative labels (non-fraud) to get balanced problem improves FN rate
- Model output in production should be evaluated by collecting human labels to identify if predictions are correct or not
    - Low confidence predictions are the best candidates to manual review
- MLflow helps reduce siloing since every team has access and visibility to the models and their metrics

# Fine-grained time series forecasting at scale with prophet and spark
- Forcaseting important for product and service demand
- Pyspark can apply a function to each groupby set of a dataframe across a distributed cluster of nodes
- Allows for timely predictions (i.e. update model as new data arrives)

# Applying image classification with PyTorch Lightning
- e.g. defect inspection for manufacturing, recommendation systems
- Scale out on GPUs and use MLflow for deep learning systems
- Main approach to training distributed deep learning models is via data parallelism, a copy of the model is sent to each GPU and they each train on different shards of the data
- Horovod (Linux foundation project) helps with this distributed training
    - Scale up learning rates with number of GPUs
    - Auto-scaling works on a single node, but not on multi node setups
- Standard way to prevent against overfitting a neural network is early stopping
    - e.g. set minimum loss improvement that we need to see in order to keep training (along with a minimum number of epochs, e.g. 10 to ensure training has plateaued and not just a few unproductive bad weight updates)




# Processing Geospatial data at scale
- Due to massive scale of data, vertically scaled deodatabases are being unloaded onto data lakes where storage is cheaper and schemas can be more flexible
- Few companies have the technology to make this data accessible for analytics, and ML rarely makes it to production
-





















