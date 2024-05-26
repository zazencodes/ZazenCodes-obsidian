---
date: 2024-03-15
tags:
    - video
hubs:
    - "[[machine-learning]]"
urls:
    - https://www.youtube.com/watch?v=G5F-L7hdqSQ
---

# Building an ML System for Southeast Asia's Largest Hospital

Goal: predict hospital bill costs

- Data flow
    - Receive data in staging area (Azure Blob)
    - Transform and load into data storage (Postgres)
    - Data prep, feature engineering
    - Train model and store in ML model storage
    - Publishing server
        - e.g. model wrapper in Docker container or binary in cloud storage
    - Deployment server
        - Each hospital has a model deployed on a different API

![2024-03-15_852.png](assets/imgs/2024-03-15_852.png)

- Source code organization
```
.
├── src
│   ├── dataprep
│   │   ├── prep_case.py
│   │   ├── prep_doctor.py
│   │   ├── prep_patient.py
│   │   ├── ...
│   │  
│   ├── feateng
│   │   ├── feat_endoding.py
│   │   ├── ...
│   │  
│   ├── ml
│   │   ├── train.py
│   │   ├── serve
│   │   │   ├── api.py
│   │   │   ├── package_api.py
│   │   │   ├── ...
│   │   └── validation
│   │       ├── train_val_split.py
│   │       ├── validate.py
│   │       ├── ...
│   └── utils
│       ├── io.py
│       ├── notification.py
│       ├── ...
│
└── tests
    ├── dataprep
    ├── feateng
    ├── ...
```
- Data validation and ingestion
    - Once data is ingested it can be very difficult to fix it
    - How to avoid ingestion problems
        - Schema and format of incoming data must be checked
        - Duplicate, null value checks must be performed
        - Volume checks - how big is the data incoming? is it far larger or smaller than expected?

- Data preparation
    - Categoricals need to be standardized
    - Numerics need to fill missing values or handle outliers
        - Note: remove outliers from trining data, but do not remove outliers from evaluation set during modeling, that way production metrics can be compared to eval metrics during training!
    - Handle text, images, etc...
    - Merge multiple datasets
    - Augment data with ecternal data (e.g. doctor data), internal features (e.g. diseaase embeddings)

- Validation
    - Random train-test split is almost never a good idea
        - Using future data to predict past data is extremely dangerous and often not what you want
    - Instead we need to do time-based split:
        - Only train on data from period leading up to validation period

![2024-03-15-909.png](assets/imgs/2024-03-15-909.png)

- Feature engineering
    - Some categorigal fields are grouped, e.g. rare instances are grouped
    - Encoding (e.g. one-hot, binary) - now people tend to just create embeddings!
    - Numerics are normalized and skewed data is transformed
    - Date or time of day of data point can reveal a lot
    - Feature crosses (e.g. male, over 50)
    - Conditional probabilites (i.e. bayesian approach, e.g. prob Y given X1, X2)

- Machine learning
    - Model & objective function choice
    - Feature selection
    - Data weights
        - e.g. Recency (decay weights linearly or exponentially so that recent sampes have a larger impact)
        - domain knowledge important here, e.g. can exclude data recommended to ignore from client
    - Parameter tuning
        - Random / grid search
        - Bayesian optimization

- Deployment
    - Train -> Validate -> Deploy -> Feedback -> Optimize -> Train ...
    - Cycle above is automated, e.g. monthly
    - Models are versioned and can be rolled back

- Domain expertise is incredibly valuable - talk to people!

- More data does not necessarily improve models

- Trainig separate models for each hospital was effective because each hospital works differently
    - I've heard the same advice for other ML use cases re: geo, e.g. cities

- Invest in proper orchestration, logging, etc..

- The ML is only <20% of the effort - the methodology and engineering process is more important


