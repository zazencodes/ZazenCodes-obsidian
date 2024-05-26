---
date: 2024-01-31
tags:
    - book
hubs:
    - "[[machine-learning]]"
---
# Designing Machine Learning Systems
Author: Chip Huyen

# Index
- Overview of ML systems
- ML system design
- Training data
- Feature engineering
- Model development and offline evaluation
- Model deployment and prediction service
- Data distribution shifts and monitoring
- Continual learning and test in production
- Infrastructure and tooling for MLOps
- The human side of ML

# Overview of ML systems

- Different stakeholders have different objectives, eg
    - ML engineers: want a model that recommends restaurants users are most likely to order from
    - Sales team: wants a model that recommends most expensive restaurants that have best service fees
    - Product team: wants a model thats fast because latency causes drop in orders
    - ML platform team: wants to delay the next model because platform should be upgraded first
    - Manager: wants to maximize margin and is considering laying off ML team
	
- Different objectives (eg ML and product teams above) can be handled with two separate models whose predictions are combined

- Its common to classify latency of serving responses in terms of percentile, eg 90th percentile or 99.9th percentile must be below a certain threshold. Higher percentiles are important because they are more likely to represent your top customers

- Examples of potential bias-introducing features: zip code, spelling of your name, credit score. Think: guilty by association. Not good. We want guilty by proof.

# ML system design
- Problem framing
    - Eg Predict app the user is going to open- better to treat it as a regression problem where each app is scored separately, rather than a single classification problem that outputs a vector of size N, where N is the number of apps

- Objective functions
    - Most commonly used: RMSE for regression, log loss for binary classification, cross entry for multi-class classification

- Decoupling
    - When there are multiple objectives, eg minimize quality loss of content censoring and minimize engagement loss, it’s a good idea to decouple them. It makes model maintenance easier. One model may need frequent re-raining but the other might be more static. You can combine the model outputs (with parametrized weighting, manually set or optimized somehow) to serve the final prediction, eg the order of posts

Hierarchy of needs:
![[Pasted image 20240114095921.png]]

# Training data

- sampling causes bias, but is usually unavoidable
    - eg models trained on more available real-world data (llms, self driving cars)
    - Random sampling may completely exclude certain rare classes

- Reservoir sampling useful for streaming data (maintain group of randomly sampled records, swapping in new ones with some probability as they come in)

- Data lineage
    - Track the origin of each of your data samples, as well as its labels
    - Training data from multiple sources can cause models to fail mysteriously

- Feedback loops
    - e.g. long feedback loop = fraudulent transactions, 1-3 months
    - e.g. short feedback loop = digital ad, measure clicks within minutes
    - frequent events can be a source of more feedback data (e.g. product view VS purchase) but are weaker signal

- Handling sparsely labeled data
    - Weak supervision
        - Use labeling function e.g. regex
    - Semi-structured
        - Self-training
            - Train model on what labels you do have, then apply model to more, then train another model on that expanded set, etc..
        - Similar characteristics
            - Extend labels to samples that have similar characteristics, e.g. using kNN
        - Perturbation
            - Apply small perturbations to labeled training samples to generate new ones
            - Assumption that small perturbations should not change the label
    - Transfer learning
        - Use models trained for some task on a different one, e.g. LLMs
    - Active learning
        - The model selects unlabeled samples that are sent back to the user to be labeled by annotators (usually humans)
        - Much more efficient than labelling random samples
        - Common metric is uncertainty measurement - label the samples that the model is mot uncertain about
        - Another metic is disagreement from multiple models - label the samples that each model disagrees about the most
        - Highly useful/applicable for realtime systems

- Class imbalance
    - Different percentiles might have different accuracy requirements, e.g. small error on very high costs = a very high cost difference!
    - ML generally works poorly on imbalanced data
        - Not much information about minority class known
        - Model gets stuck in nonoptimal solution by exploiting a simple heuristic, e.g. classify none as the minority class, get very high accuracy still
    - Techniques
        - Use F1, recall and precision instead of accuracy
        - Asymmetric metrics that depend on which class is selected as the "positive" class
        - Precision = true positive / (true positive + false positive)
        - Recall = true positive / (true positive + false negative)
        - F1 = 2 * precision * recall / (precision + recall)
    - Data-level methods
        - Undersample majority class - may result in loss of important data
        - Oversample minority class- may result in overfitting
        - There are many algorithms to address these biases, eg two-phase learning and dynamic sampling
        - SMOTE and Tomek can be  used to oversample
        - Never evaluate your model on resampled data- it will cause it to fit to that distribution. Only use it to train
    - algorithm-level methods
        - Many involve adjustments to the loss function
        - Eg cost-sensitive learning, class-balanced loss, focal loss

- Data augmentation
    - Make models more robust to noise and adversarial attacks
    - Label-preserving
        - Alter photos or text (e.g. flip, crop, replace words, sentence structure, etc..)
        - Perturbation
            - Add noisy samples to training dataset
            - For images, prevents against adversarial attacks where altering one pixel can change the prediction dramatically
    - Data synthesis
        - Can use templates to generate training data, e.g. "find me a [food category] restaurant within [x] miles of [location]"

# Feature engineering

- Filling missing data
    - Delete / input

- Scaling
    - Standardization or min/max (0-1)
    - Log transforms good for skewed data, to make it more normal

- Handling new categorical variables (not seen during training)
    - e.g. new brand on Amazon lists products, model does not know if it should show those products or not
    - Solution is he hashing trick
        - Hash category names in fixed space
        - Unseen categories (e.g. brands) will get assigned position in that space
        - Also helps reduce size of high-dimensional data sets

- Feature crossing
    - Makes learning non-linear relationships between variables easier

- Embeddings
    - Maps variable-length value onto fixed-length vector space
    - Positional embeddings involve e.g. the position of words in a sentence

- Data leakage
    - When some form of the label sneaks into the feature in a way that's unrelated to inference
        - e.g. different CT scans used for different patients, high-res machine used on patience at higher risk of cancer, model learns that high-res images more likely to be cancer, regardless of test result
        - e.g. patients lying down more likely to have COVID, so model learns that images of lying down patients more likely to be COVID regardless of test result
    - Where to watch for data leakage
        - Split time-correlated features by time instead of randomly
            - Prevents information from test/validation domain from leaking into the training set
            - e.g. use first 4 weeks as training sets, and split 5th week into validation and test sets
        - Split before scaling - use the training set to define the scaling
        - Remove duplication (watch for near-duplication) - can cause data to appear in both training and test/validation
        - Group leakage - e.g. photos taken a millisecond apart, should not include one image in training and another in test/validation
        - Leakage from data generation - e.g. examples above with CT scans, requires domain-expertise to think through and identify these sources of bias
    - General tips for avoiding
        - Be skeptical about features that the model determines to be the most important
        - Be careful about how you do train/test splits

- Good features
    - Too many features can be bad
        - More opportunities for data leakage
        - Easier to overfit
        - Model operational costs of training/serving, inference latency, memory overflows, etc..
        - Technical debt
    - Use feature importance or regularization to reduce
    - How well does the feature generalize? Will future samples actually have that feature available as a predictor?

# Model development and offline evaluation

- Knowledge of common ML tasks and typical approaches is essential
    - Classification: assign label, e.g. spam detection
        - SVM, random forest, gradient boost *(objective e.g. Minimize cross-entropy loss)*
    - Regression: predict number, e.g. house price
        - Linear regression, ridge regression, lasso regression *(objective e.g. Minimize MSE)*
    - Clustering: group similar records, e.g. customer segmentation for targeted marketing
        - K-means, DBSCAN, *(objective e.g. Minimize variance or silhouette score)*
    - Dimensionality reduction: reduce number of features while preserving information, e.g. image compression
        - PCA, t-SNE, *(objective e.g. Maximize explained variance)*
    - NLP: understand language *(objective e.g. sentiment analysis)*
        - Transformer models (objective e.g. Minimize cross-entropy loss)
    - Anomaly detection: identify outliers, e.g. fraud
        - Isolation forest, one-class SVM, autoencoder *(objective e.g. Minimize reconstruction error)*
    - Reinforcement learning: make sequential decisions to maximize rewards, e.g. game playing, robotic control
        - DQN, policy gradient *(objective e.g. Reward maximization)*
    - Recommendation systems: suggest items, e.g. movie recommendations
        - Collaborative filtering, content-based filtering *(objective e.g. Ranking-based loss)*

- Model-selection tips
    - Avoid state of the art (research doesn't always translate effectively to real-world)
    - Start simple (Easier to deploy for MLops validation, benchmark, easy to understand and debug as you develop it)
    - Understand model assumptions (e.g. input data is independent and identically distributed (neural network assumption), linear boundaries (linear regression), conditionally independent (features for naieve bayes))

- Ensembles
    - Motivation
        - Capture different patterns
        - Tend to perform better, when taking majority vote
        - Reduce variance, improve robustness (guard against overfitting)
    - Methods
        - Bagging: random sample with replacement (e.g. random forest)
        - Boosting: future models trained on samples that get misclassified by previous iterations (e.g. boosted tree)
        - Stacking: average vote from different base models

- Experiment tracking
    - Loss curve corresponding to the training and validation data
    - Model performance (e.g. MSE, F1)
    - Log of corresponding sample, prediction and ground truth label
    - Speed of training (steps/second, or tokens/second)
    - System performance metrics (CPU/GPU, memory)
    - Hyperparameter values over time

- AutoML
    - Never use test split to tune hyperparameters, use the validation split

- Phases of ML model development
    - 1. Before ML
        - Take a stab at the problem without any ML. Benchmark for how well you can do.
    - 2. Simplest ML
        - Train a simple model and get it working in production
    - 3. Optimize
        - Optimize the simple ML models with different objective functions, hyperparemeter search, more data, ensembles, etc.. Determine how fast your model decays in the wild and build out infrastructure for periodically re-training.
    - 4. Complex (optional)
        - Experiment with more complex models (e.g. neural networks).

- Model offline evaluation
    - Compare to baselines
        - Random baseline (e.g. MSE, F1 for uniform & task-label distribution)
        - Simple heuristic (e.g. phase #1 of ML model dev. above)
        - Zero rule baseline (when model always predicts the most common class)
        - Human baseline (compared to humans)
    - Methods
        - Perturbations
            - The more sensitive your model is to small perturbations on training data, the less well it will perform in the real world
        - Invariance
            - Change sensitive features such as race, name, etc.. and make sure model still outputs the same result
            - (this is meant to avoid bias, I feel like it's silly.. those features simply should not be in the model in the first place)
        - Directional expectation
            - Does the prediction directionally change how you expect?
        - Model calibration
            - How well does the predicted probability match the distribution
            - Use calibration curve (predicted probability on X-axis, actual fraction of positive labels on Y-axis (binned)) to test, want diagonal for perfect calibration
            - Neural networks tend to have poor calibration, can use Platt Scaling or Isotonic Regression to adjust
        - Confidence measurement
            - How confident do you need your model to be in order to trust the results it's showing?
        - Slice-based evaluation
            - Slice your test data and see how the model performs on each
            - Do this for subgroups of the data (not random)

# Model deployment and prediction service

- Batch deployments make periodic batches of predictions to be able to serve pre-computed predictions to users

- Online deployments make realtime predictions

- Companies investing in going from batch to online
    - Need real-time pipeline that can extract streaming features from incoming data, input into model and return a prediction. Should have latency on the order of milliseconds

- Historically, Spark (mapreduce) has dominated batch prediction extremely effectively, but realtime requires different solution

- On cloud VS on the edge
    - The edge = on device; browser, phone, car, smartwatch, etc..
    - Cloud is easier, but costly
    - Edge can run without stable internet, avoids transit of potentially sensitive data

- Model optimization
    - ML-powered compiler, e.g. autoTVM, adapts model training to hardware architecture
        - Can be slow at first (e.g. for days)
        - Ideal when you have a model ready for production and target hardware to run inference on
- ML in browsers
    - Models can be compiled into JS, e.g. TensorFlow.js, but JS is slow and the language is limited for complex logics such as extracting features from data
    - WebAssemply (WASM) is an open standard that allows you to run executable programs in browsers
        - Compile model to WASM, run with JS
        - Still relatively slow because it's limited by browser

# Data distribution shifts and monitoring

- Models decay in production

- Failure can be operational (is it returning results) or performance (accuracy)

- In google study, > 50% of ML failures stemmed from distributed systems, e.g. data pipeline error (bad data)

- Production data differing from training data
    - Data constantly changing, can be sudden or gradual
    - It can also be seasonal

- Edge cases
    - Outliers that cause performance to be significantly worse (not all outliers are edge cases)
    - It's tempting to remove outliers when training data, but that could lead to poor performance on edge cases

- Degenerate feedback loops
    - When ML predictions themselves influence the next iteration of training
    - e.g. recommendation systems
    - Measured by diversity metrics (specifically in the long tail)
    - Simple correction is to use randomization, which hurts user experience

- Data distribution shifts
    - Research classifications
        - Covariate shift
            - The distribution of input data X changes i.e. P(X), but P(y|X) stays the same, e.g. start marketing to older audiences (user base change)
        - Label shift
            - The distribution of labels y changes i.e. P(y), but P(X) stays the same
        - Concept drift
            - The conditional distribution of the output changes i.e. P(y|X), but P(X) stays the same, e.g. house prices post-covid (same houses on the market, now they are worth more/less)
    - Real-world examples
        - Feature change
            - New feature added / old feature removed, or set of all possible feature values changes
        - Label change
            - e.g. schema (POS/NEG strings instead of integers, new label option NEUTRAL), range of regression labels
    - Detecting distribution shifts
        - Monitor models accuracy metrics - MSE, F1 score, recall, AUC, etc.. in production (if you have access to labels)
        - Monitor distributions P(X), P(y), P(y|X)
            - Statistical methods, e.g. 5th, 25th, 75th, 95th percentile, skewness, mean, median, min, max, variance
            - "two-sample" drift detection algorithms, e.g. K-S test, MMD - tests to determine if two populations samples are statistically different
            - Timeseries data needs special consideration around seasonality to avoid incorrectly identifying drift
    - Correcting for distribution shifts
        - Typically train a new model, or update model on new data (stateful training)
        - Address possibility of distribution shift early, e.g.
            - trade off accuracy by binning feature that tends to change frequently, such as app ranking in the app store (1-10, 11-100, 101-1000, etc..)
            - train different models for different areas, e.g. house prices in SF vs Arizona - SF model might need frequent updates

- Monitoring and observability
    - User feedback can be valuable in measuring accuracy
    - Monitoring predictions is straightforward - changes in prediction distributions P(y) tend to indicate changes in underlying input distribution P(X)
    - Feature validation: min, max, median within acceptable range, regex validation, feature values belong to expected set, pairwise relationships between features are satisfied
        - Concerns: can be very expensive to monitor every feature for every model constantly, useful for debugging but not extremely helpful for detecting performance degradation, feature transformations might be missed because monitoring is done downstream
    - Monitoring raw inputs often not possible by ML team directly, since they are managed by an upstream team

- Logging
    - Want to make it as easy as possible to find logged events later
    - Distributed tracing (microservice architecture idea) - each process has a unique ID that can be used to pull all related logs for that process
    - Good realtime solutions for log monitoring are using message queues like Kafka or Amazon Kinesis and search for specific characteristics using streaming SLQ such as KSQL or Flink SQL
    - Telemetry - logs and metrics collected form remote components such as cloud services or applications run on customer devices

# Continual learning and test in production

- Continual learning
    - Update a challenger model (a replica of the "champion" model - the one in production) every ~1024 samples, if it performs better than the champion then replace the champion
    - Significantly cheaper than training from scratch
    - Ecom use case: adapt quickly to user behavior, e.g. during black friday event, or even during the course of a single session

- Difficulties of continual learning
    - Fresh data is hard to get, one solution is pull direct from streaming before it hits the data warehouse - the data needs to be transformed (labeled) which is difficult to do in a timely way
        - e.g. Search for product, click on one = label the search query with the clicked product ID, hard to do with streaming
    - Evaluating the challenger is hard, need to watch for adversarial attacks, sometimes collecting enough data to test properly is hard e.g. fraud use case - need to wait weeks to collect enough fraud data
    - Algorithmic challenge when transforming features (e.g dim reduction) or doing an update is not well performant for small samples, this only affects extremely fast updating models, eg. hourly.
        - Neural networks relatively easy to update compared to matrix or tree-based models
        - Feature reuse can help here: features may have already been extracted for production predictions, which can be re-used to train the challenger model, this is called "log and wait"

- Stages of continual learning
    - Manual, stateless retraining
        - Painful place to be, most companies without an ML platform team are here
        - Models are running but not being updated, some have been running for months or years without updates
        - Updating is manual and ad hoc, data is passed through multiple teams and pre/post performance is not carefully tested
    - Automated retraining
        - You have a script that runs a batch retraining process, most companies with mature ML infrastructure are at this stage
        - Often retraining frequency is based on gut feel, not optimally tested through experiments
    - Automated stateful retraining
        - Same as above but stateful, incremental updates
        - Model versioning and data lineage important to track
    - Continual learning
        - Model automatically updated based on requirements, rather than fixed time interval
        - e.g. time-based (5 minutes), performance based (performance degradation), volume-based (new data), drift-based (data distribution change)
        - Can be done on the edge, e.g. ship device with base model and update it without needing centralized servers

- How often to update your models
    - Train multiple models, withholding various periods of previous data.
        - e.g. to simulate going a week without training, then test a model that's been trained on data up to a week before the last week, and compare that to performance from a time window shifter up by a week

- Test in production
    - Shadow deployment
        - Deploy challenger model alongside champion, watch the outputs of the challenger but do not serve to users
        - Downside is cost
    - A/B testing
        - Select winner based on predictions, business objectives and user feedback
        - For some cases, may need to A/B test temporally, e.g. serve model A one day and model B the next, rather and 50/50 split of realtime traffic
    - Canary release
        - Gradual roll out, if key metrics degrade significantly then release is aborted
        - Doesn't have to work like an A/B test - can roll out model to sub-market, e.g. one that is less critical
    - Interleaving
        - Expose results of both models to a single user to determine preference
        - e.g. user to recommendations from combination of models - which ones does the user click on?
        - Need to correct for bias based on how the user sees the result, e.g. for recommendations the order matters
    - Bandits
        - Simplest example is the e-greedy model: say e = 0.9, then you serve the best model 90% of the time, and pick a random different one 10% of the time.

# Infrastructure and tooling for MLOps

- Development environment
    - "if you can only set up one piece of infrastructure well, make it the dev environment for data scientists" - it improves productivity
    - Version control, e.g. Git (code), DVC (data version control), Weights & Biases (track experiments), MLflow (model artifacts)
    - CI/CD, e.g. github actions
    - Remote compute, e.g. GitHub codespaces, AWS EC2, GCP instance
        - Developers SSH into this via IDE, so all devs work with standard dependencies. Cost is about $70/month for constantly running instance with 4vCPUs and 8GB memory

- Production
    - Docker compose limited to one host, kubernetes solves that by creating a network of containers that communicate, share resources and automatically scale up or down
    - Kubeflow and Metaflow (which should be used alongside a proper scheduler like AIrflow) allow for seamless work between dev and prod in the same script or notebook, parameterized data size/compute resource on a per-function basis

- ML platform
    - Model deployment
    - Model storage
        - Along with model binary, need as much data as possible - e.g. to help understand model failures
        - Model definition, e.g. loss function
        - Model parameters
        - Featurize and predict functions
        - Code versions and dependencies
        - Data, e.g. commit that generated data with DVC
        - Model generation code, e.g. notebook
        - Experiment artifacts, e.g. raw numbers of performance on test sets
        - Tag, e.g. person who created model, business problem, etc..
    - Feature store
        - e.g. Feast (open-source), SageMaker, Databricks
        - Management, i.e. feature catalog - Help teams share and discover features
        - Computation, performing and storing result
        - Consistency, unification of logic cross languages, for training and inference and for batch and streaming

# The human side of ML

- Better UX
    - Consistency-accuracy tradeoff
        - Users want consistency. Delivering this may impact accuracy
    - Human-in-the-loop
        - Combat "mostly right" predictions, e.g. GPT
        - Strategy: show multiple prediction results to user and they can pick the better one
    - Smooth failing
        - e.g. models can take a long time to run
        - Strategy: have a simpler backup model that is triggered if the model is taking over X milliseconds to run a prediction

- Responsible AI framework
    - Discover sources for model bias
        - Training data
            - Is representative of real world?
        - Labeling
            - Are these accurate? Subject to human error/bias
        - Feature engineering
            - Does your model use sensitive information? e.g. ethnicity, gender, religion. Or features that correlate with these?
            - Does it have a disparate impact on a subgroup of people? (widely different outcomes based on subgroup)
        - Model objective
            - Is it fair for all users? Are you biasing towards objectives that benefit some user over others?
        - Evaluation
            - Are you going fine-grain to determine impact on all user groups?
    - Create model cards
        - https://arxiv.org/pdf/1810.03993.pdf
        - A document for each model about how they are trained and evaluated
        - Model details, use case, sensitive factors, metrics, evaluation data, training data, etc..



