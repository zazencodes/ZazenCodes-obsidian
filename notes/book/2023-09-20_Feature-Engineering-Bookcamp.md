---
date: 2023-09-20
tags:
  - book
hubs:
  - "[[data-engineering]]"
  - "[[machine-learning]]"
  - "[[python]]"
urls:
  - https://github.com/sinanuozdemir/feature_engineering_bookcamp/tree/main/notebooks
---
# Feature Engineering Bookcamp
Author: Sindan Ozdemir

- Feature engineering = manipulating data into a format the optimally represents the underlying problem (that an ML algorithm is trying to model) and mitigates complexities and biases
# Index
- Concepts
- General Feature Engineering
- Text Mining
- Image Mining
- Day Trading
- Feature stores
- Chapter code

# Concepts

## Five types
- Feature improvement e.g. fill missing data
- Feature construction e.g. calculate price per square foot
- Feature selection e.g. remove bad features
- Feature extraction e.g. Use BERT on text
- Feature learning e.g. GANs

## Levels of data
- Nominal
    - Categories with no order
    - e.g. blood type
- Ordinal
    - Categories with order
    - e.g. satisfaction level
- Interval
    - Like ordinal, but difference between numbers has consistent interpretation
    - e.g. temperature
- Ratio
    - Like interval, but has a zero-point
    - e.g. speed

## Null accuracy
- The accuracy if the model guesses the most common target every time
- e.g. if 90% of targets are category A, then the null accuracy to beat is 90%

## Transforming data to be more normal
- ML models work better on data that's normally distributed and don't have outliers
- Log transforms (x -> log(x+1)) can help with this
- Box-cox transforms work even better (`PowerTransformer` in sklearn)
- Min-max scaling (from 0-1) doesn't handle outliers as well as z-score standardization (mean of 0, variance of 1)

## Ordinal vs nominal data
- One-hot-encoding needed for nominal data
    - Do not arbitrarily assign numeric values to these categorical variables since it creates false dependence that model will learn

# General Feature Engineering

## Constructing categorical data
- Binning data generally reduces performance but helps avoid overfitting
- Use domain-specific knowledge to combine data and create new features
    - e.g. has_flu_symptoms = any(sore_throat, cough, fever, ...)

## Feature selection
- Can use "mutual information" to select top n features that best predict the target and remove the rest
- Same idea with chi squared test (measuring features with target) and decision tree feature importances

## Bias aware models
- Sensitive variables like race or gender should not factor into predictions (i.e. avoid disparate treatment)
- Random forest feature importance helps prove model unawareness of sensitive variables
- Use Dalex library to understand model bias using model_fairness method
- Mitigate bias with preprocessing
    - Can remove sensitive features directly
    - Transform features that depend on sensitive features, so model cannot learn them indirectly
        - e.g. Yeo-Johnson transformation separately on feature for each sensitive-feature segment
    - Can use algorithms e.g. AIF360 LFR (learning fair representation)
- Mitigate bias in processing
    - e.g. meta fiar classifier
- Mitigate bias post-processing
    - e.g. equalized odds (modifying labels)

## Grid search pipelines
- Makes hyperparameter choice trivial
- Should include feature selection / transformation
e.g.
```python
clf = LogisticRegression(max_iter=10000)
ml_pipe = Pipeline([
                    ("vectorizer", CountVectorizer()),
                    ("classifier", clf),
])
params = {
          "vectorizer__lowercase": [True, False],
          "vectorizer__stop_words": [None, "english"],
          "vlassifier__C": [1e-1, 1e0, 1e1],
}
advanced_grid_search(x_train, y_train, x_test, y_test, ml_pipe, params)
```


# Text Mining

## Cleaning text for "bag of words" modeling
- The `max_features` param of a text vectorizer helps strip out rare/useless characters
- Lowercase everything, remove stop words and "bad_regex" patterns, e.g. mentions on twitter, URLs
- Stem words, e.g. teeth -> tooth, ran -> run, waiting -> wait

## Latent feature engineering for text data
- PCA using SVD algorithm
    - Idea being to transform a high dimensional dataset with possibly correlated features into a lower dimensional dataset with uncorrelated features
- Autoencoders
    - Neural networks that learn how to transform the feature set to reduce noise
- Transfer learning with BERT text vectorizer
    - Takes in raw, variable-length text and output a fixed-length representation of that text
    - BERT has learned grammar, context and tokens from billions of words of Wikipedia and million of words of BookCorpus data
    - It's trained to fill in the blanks and predict next sentence - tasks designed to teach BERT the basics of language modeling
    - Especially useful for few-shot learning, where we have very little training data e.g. 10-100 samples

## Overall approach to text-data learning
- Use count vectorization and TF-IDF for baseline, interpretable models
- Add dimensionality reduction like PCA with SVD or an autoencoder
- Use transfer learning like BERT for few-shot learning or when pre-trained features are valuable

# Image Mining

## Image recognition with VGG-11
- Like BERT but for images - VGG = "visual geometry group"
- Convolutional neural network that has learned what filters to apply on images in order to classify the ImageNet dataset (1k different labels)
- Use VGG-11 to extract features from images
    - Pre-trained model is loaded with PyTorch and used to transform pixels into vectors of length 1,000
    - Can instead fine-tune model to output desired number of features to represent your classification problem, e.g. 10

# Day Trading

## General approach to timeseries modeling
- What is the response variable going to be? e.g. a future variable
- How can we construct basic date/time features, e.g. rolling windows
- What domain-specific features can we construct? e.g. MACD
- Are there other sources of data we can include? e.g. twitter
- Can we automatically extract feature interactions? e.g. polynomials

## Day trading basic features and target
- Start with `date`, `close`, `volume`
- Add columns `date`, `day_close_price`, `pct_change_eod` to get the response variable: `stock_price_rose`
- Feature categories:
    - date/time, e.g. `dayofweek`, `morning`
    - lag, e.g. `30_min_ago_price`, `7_day_ago_price`
    - rolling-window, e.g. `rolling_close_mean_60_min`,`rolling_close_std_60_min`,  `rolling_volume_mean_60_min`, `rolling_volume_std_60_min`
        - have fixed timeframe (forgets long-term)
    - expanding window, e.g. `expanding_close_mean`, `expanding_volume_mean`
        - have expanding timeframe (remembers long-term)

## Cross validation for timeseries
- Training set can only have data from before test set
- Split data into N chunks, e.g. n1, n2, n3, n4. Then train/test on splits as follows: (n1, n2), (n1+n2, n3), (n1+n2+n3, n4)
- Can be done with TimeSeriesSplit from sklearn

## Baseline model pipeline with standard scalar
```python
clf = RandomForestClassifier()
pipe = Pipeline([
    ('scale', StandardScaler()),
    ('clf', clf),
])
params = {
    'clf_criterion': ['gini', 'entropy'],
    'clf_in_samples_split': [2, 3, 5],
    'clf_max_depth': [10, None],
    'clf_max_features': [None, 'auto'],
}
train_df, test_df, train_X, train_y, test_X, test_y = split_data(price_df)
# ^ uses TimeSeriesSplit
best_miodel, test_preds, test_probas = advanced_grid_search(
    train_X, train_y, test_X, test_y,
    pipe, params, cv=tscv, include_probas=True
)
```
- Check results
    - against null accuracy (outcome of random guessing) before calling it a win
    - in terms of cumulative gains
        - 1. by listening to first prediction of the day
        - 2. by listening to all preductions
        - 3. by listening to bullish predictions
        - 4. by listening to bearish predictions (not sure how this would yield gains...)

## Domain-specific features for day trading
- Daily price (related to the market's close/open windows): `overnight_change_close`, `monthly_pct_change_close`, `expanding_average_close`
- Technical indicators: `macd`
- Twitter: `rolling_7_day_total_tweets`, `rolling_1_day_verified_tweets`

## Feature selection in model pipeline
- Using `SelectFrommodel` or `RFE` to filter out unimportant features
```python
rf = RandomForestClassifier(n_estimators=20, max_depth=None, random_state=0)
lr = LogisticRegression(random_state=0)
pipe = Pipeline([
    ('scale', StandardScalar()),
    ('select_from_model', SelectFromModel(estimator=rf)),
    ('clf', clf),
])
params.update({
    'select_from_model__threshold': ['0.5 * mean*', 'mean', '0.5 * median*, 'median],
    'select_from_model__estimator': [rf, lr],
})
```
- Only useful when we have many features (did not yield improvements in day trading models of book)
- Useful in conjunction with polynomial features

## Feature extraction for day trading
- Polynomials
```python
p = PolynomialFeatures(3)
poly_features = p.fit_transform(df[features])
print(f"Created features: {p.get_feature_names()}")
```

# Feature stores

## Overview
- System to enable effective machine learning: data input, tracking, governance. Ensure features are accurate, up to date, versioned, consistent.
- Centralized solution can be used by many teams and people
- Middle man between raw data stores (databases) and ML practitioners

## MLOps
- Data ingestion and aggregation pipelines from multiple data sources
- Model training, testing and version
- Ability to continually integrate and deploy pipelines

## Hopsworks
- Open source python implementation available
- Can create AWS clusters automatically for compute

## Feature groups
- Similar sets of features which are batched
- Cached - stored in Hopsworks
    - Online - only recent data is cached
    - Offline - all historical data is cached
- On-demand - stored only in raw database

# Chapter code

## Chapter 3

### Utility functions
```python
import numpy as np
np.random.seed(0)
import random
random.seed(0)


from sklearn.ensemble import ExtraTreesClassifier
from sklearn.model_selection import GridSearchCV
from sklearn.metrics import classification_report, mean_squared_error
from sklearn.pipeline import Pipeline
import time


def simple_grid_search(x_train, y_train, x_test, y_test, feature_engineering_pipeline):
    '''
    simple helper function to grid search an ExtraTreesClassifier model and
    print out a classification report for the best param set.
    Best here is defined as having the best cross-validated accuracy on the training set
    '''

    params = {  # some simple parameters to grid search
        'max_depth': [10, None],
        'n_estimators': [10, 50, 100, 500],
        'criterion': ['gini', 'entropy']
    }

    base_model = ExtraTreesClassifier()

    model_grid_search = GridSearchCV(base_model, param_grid=params, cv=3)
    start_time = time.time()  # capture the start time
    if feature_engineering_pipeline:  # fit FE pipeline to training data and use it to transform test data
        parsed_x_train = feature_engineering_pipeline.fit_transform(x_train, y_train)
        parsed_x_test = feature_engineering_pipeline.transform(x_test)
    else:
        parsed_x_train = x_train
        parsed_x_test = x_test

    parse_time = time.time()
    print(f"Parsing took {(parse_time - start_time):.2f} seconds")

    model_grid_search.fit(parsed_x_train, y_train)
    fit_time = time.time()
    print(f"Training took {(fit_time - start_time):.2f} seconds")

    best_model = model_grid_search.best_estimator_

    print(classification_report(y_true=y_test, y_pred=best_model.predict(parsed_x_test)))
    end_time = time.time()
    print(f"Overall took {(end_time - start_time):.2f} seconds")

    return best_model


def advanced_grid_search(x_train, y_train, x_test, y_test, ml_pipeline, params, cv=3, include_probas=False, is_regression=False):
    '''
    This helper function will grid search a machine learning pipeline with feature engineering included
    and print out a classification report for the best param set.
    Best here is defined as having the best cross-validated accuracy on the training set
    '''

    model_grid_search = GridSearchCV(ml_pipeline, param_grid=params, cv=cv, error_score=-1)
    start_time = time.time()  # capture the start time

    model_grid_search.fit(x_train, y_train)

    best_model = model_grid_search.best_estimator_

    y_preds = best_model.predict(x_test)

    if is_regression:
        rmse = np.sqrt(mean_squared_error(y_pred=y_preds, y_true=test_set['pct_change_eod']))
        print(f'RMSE: {rmse:.5f}')
    else:
        print(classification_report(y_true=y_test, y_pred=y_preds))
    print(f'Best params: {model_grid_search.best_params_}')
    end_time = time.time()
    print(f"Overall took {(end_time - start_time):.2f} seconds")

    if include_probas:
        y_probas = best_model.predict_proba(x_test).max(axis=1)
        return best_model, y_preds, y_probas

    return best_model, y_preds
```


### Feature engineering pipeline
```python
from sklearn.preprocessing import FunctionTransformer
from sklearn.pipeline import Pipeline, FeatureUnion

# deal with risk factors

risk_factor_pipeline = Pipeline(
    [
        ('select_risk_factor', FunctionTransformer(lambda df: df['RiskFactors'])),
        ('dummify', DummifyRiskFactor()),
        ('tree_selector', SelectFromModel(max_features=20, estimator=DecisionTreeClassifier()))
    ]
)

# deal with binary columns

binary_pipeline = Pipeline(
    [
        ('select_categorical_features', FunctionTransformer(lambda df: df[binary_features])),
        ('fillna', SimpleImputer(strategy='constant', fill_value=False))  # assume missing values are not present
    ]
)

# deal with numerical columns

numerical_pipeline = Pipeline(
    [
        ('select_numerical_features', FunctionTransformer(lambda df: df[numerical_columns])),
        ('impute', SimpleImputer(strategy='median')),
    ]
)

simple_fe = FeatureUnion([
    ('risk_factors', risk_factor_pipeline),
    ('binary_pipeline', binary_pipeline),
    ('numerical_pipeline', numerical_pipeline)
])

best_model = simple_grid_search(x_train, y_train, x_test, y_test, simple_fe)
```

## Chapter 5

### Twitter NLP feature engineering pipeline
```python
# clean tweets https://pypi.org/project/tweet-preprocessor

import preprocessor as tweet_preprocessor

# remove urls and mentions
tweet_preprocessor.set_options(
    tweet_preprocessor.OPT.URL, tweet_preprocessor.OPT.NUMBER
)

tweet_preprocessor.clean(
    '@United is #awesome 👍 https://a.link/s/redirect 100%'
)

Out[29]:

'@United is #awesome 👍 %'

In [30]:

tweet_preprocessor.set_options(
    tweet_preprocessor.OPT.URL, tweet_preprocessor.OPT.NUMBER
)

ml_pipeline = Pipeline([
    ('vectorizer', TfidfVectorizer()),  # TfidfVectorizer gave us better results
    ('classifier', clf)
])

params = {
    'vectorizer__lowercase': [True, False],
    'vectorizer__stop_words': [None, 'english'],
    'vectorizer__max_features': [100, 1000, 5000],
    'vectorizer__ngram_range': [(1, 1), (1, 3)],

    'classifier__C': [1e-1, 1e0, 1e1]

}

print("Tweet Cleaning + Log Reg\n=====================")
advanced_grid_search(
    # apply cleaning here because it does not change given the training data
    train['text'].apply(tweet_preprocessor.clean), train['sentiment'],
    test['text'].apply(tweet_preprocessor.clean), test['sentiment'],
    ml_pipeline, params
)
```

### TF-IDF + SVD feature engineering pipeline
```python
from sklearn.decomposition import TruncatedSVD  # A

ml_pipeline = Pipeline([
    ('vectorizer', TfidfVectorizer()),  # B
    ('reducer', TruncatedSVD()),
    ('classifier', clf)
])

params = {
    'vectorizer__lowercase': [True, False],
    'vectorizer__stop_words': [None, 'english'],
    'vectorizer__max_features': [5000],
    'vectorizer__ngram_range': [(1, 3)],

    'reducer__n_components': [500, 1000, 1500, 2000],  # number of components to reduce to

    'classifier__C': [1e-1, 1e0, 1e1]

}

print("SVD + Log Reg\n=====================")
advanced_grid_search(
    train['text'], train['sentiment'],
    test['text'], test['sentiment'],
    ml_pipeline, params
)

# A feature extraction / dimension reduction with SVD
# B our custom tokenizer didn't work out so well so we will remove it
```

## Chapter 7

### Basic timeseries model pipeline
```python
from sklearn.pipeline import Pipeline  # A
from sklearn.ensemble import RandomForestClassifier
from sklearn.preprocessing import StandardScaler

clf = RandomForestClassifier(random_state=0)

ml_pipeline = Pipeline([  # B
    ('scale', StandardScaler()),
    ('classifier', clf)
])

params = {  # C
    'classifier__criterion': ['gini', 'entropy'],
    'classifier__min_samples_split': [2, 3, 5],

    'classifier__max_depth': [10, None],
    'classifier__max_features': [None, 'auto']
}

# A Import the scikit-learn Pipeline object
# B Create a pipeline with feature scaling and our classifier
# C Create the base gridsearch parameters

In [18]:

from sklearn.model_selection import TimeSeriesSplit  # A

tscv = TimeSeriesSplit(n_splits=2)  # A

# A This splitter will give us train/test splits optimized for time data

In [19]:

for i, (train_index, test_index) in enumerate(tscv.split(price_df)):
    train_times, test_times = price_df.iloc[train_index].index, price_df.iloc[test_index].index
    print(f'Iteration {i}\n-------------')
    print(f'''Training between {train_times.min().date()} and {train_times.max().date()}. Testing between {test_times.min().date()} and {test_times.max().date()}\n'''
    )

Iteration 0
-------------
Training between 2020-01-09 and 2020-07-01. Testing between 2020-07-01 and 2020-12-22

Iteration 1
-------------
Training between 2020-01-09 and 2020-12-22. Testing between 2020-12-22 and 2021-07-08

In [20]:

price_df.filter(regex='feature__').corrwith(price_df['stock_price_rose']).sort_values()

Out[20]:

feature__rolling_volume_mean_60   -0.003444
feature__rolling_volume_std_60    -0.002263
feature__expanding_volume_mean    -0.002219
feature__dayofweek                -0.001477
feature__morning                   0.025632
feature__expanding_close_mean      0.033297
feature__rolling_close_mean_60     0.040732
feature__lag_30_min_ago_price      0.040766
feature__lag_7_day_ago_price       0.041196
feature__rolling_close_std_60      0.055433
dtype: float64
```

### Feature selection for timeseries data
```python
from sklearn.feature_selection import SelectFromModel
from sklearn.linear_model import LogisticRegression

rf = RandomForestClassifier(n_estimators=20, max_depth=None, random_state=0)  # A
lr = LogisticRegression(random_state=0)

ml_pipeline = Pipeline([
    ('scale', StandardScaler()),
    ('select_from_model', SelectFromModel(estimator=rf)),
    ('classifier', clf)
])

params.update({
    'select_from_model__threshold': [
        '0.5 * mean', 'mean', '0.5 * median', 'median'
    ],
    'select_from_model__estimator':  [rf, lr]
})

print("Feature Selection (SFM) \n==========================")
best_model, test_preds, test_probas = advanced_grid_search(
    train_X, train_y,
    test_X, test_y,
    ml_pipeline, params,
    cv=tscv, include_probas=True  # C
)

del params['select_from_model__threshold']
del params['select_from_model__estimator']

plot_gains(test_df.copy(), test_y, test_preds)

# The feature importances in this random forest will dictate which features to select
```


### Polynomial feature extraction on timeseries data
```python
from sklearn.preprocessing import PolynomialFeatures

ml_pipeline = Pipeline([
    ('poly', PolynomialFeatures(1, include_bias=False)),
    ('scale', StandardScaler()),
    ('select_from_model', SelectFromModel(estimator=rf)),  # A
    ('classifier', clf)
])

params.update({
    'select_from_model__threshold': ['0.5 * mean', 'mean', '0.5 * median', 'median'],
    'select_from_model__estimator':  [rf, lr],
    'poly__degree': [2],
})

print("Polynomial Features \n==========================")
best_model, test_preds, test_probas = advanced_grid_search(
    train_X, train_y,
    test_X, test_y,
    ml_pipeline, params,
    cv=tscv, include_probas=True  # C
)

del params['poly__degree']
del params['select_from_model__threshold']
del params['select_from_model__estimator']

plot_gains(test_df.copy(), test_y, test_preds)
```



