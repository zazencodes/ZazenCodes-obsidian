---
date: 2024-04-15
tags:
    - doc
hubs:
    - "[[machine-learning]]"
urls:
    - https://scikit-learn.org/stable/modules/ensemble.html
    - https://scikit-learn.org/stable/auto_examples/ensemble/plot_gradient_boosting_regression.html
    - https://towardsdatascience.com/all-you-need-to-know-about-gradient-boosting-algorithm-part-1-regression-2520a34a502
    - https://towardsdatascience.com/all-you-need-to-know-about-gradient-boosting-algorithm-part-2-classification-d3ed8f56541e
    - https://datascience.stackexchange.com/questions/26699/decision-trees-leaf-wise-best-first-and-level-wise-tree-traverse
---

# Gradient Boosting Algorithm

- General framework for many types of models, but most commonly used with decision trees
- Works like gradient descent, minimzes some loss function
- Can use any loss function that is differentiable

## Boosted trees

- In each stage n_classes_ regression trees are fit on the negative gradient of the loss function

- Ensemble methods improve generalizability & robustness compared to a single estimator

- Compared to random forests
    - Sequential boosting: each tree is trained to correct the errors made by the previous ones. This allows them to iteratively improve the model’s performance using relatively few trees. In contrast, random forests use a majority vote to predict the outcome, which can require a larger number of trees to achieve the same level of accuracy.

- An objective function based on the gradient is used to build the tree, rather than entropy/gini

- Learning rate
    - Scales down the updates in order to not overfit. Value should be small, e.g. 0.1 or 0.01

- Tree size is determined by `max_depth` or `max_leaf_nodes`
    - e.g. max depth of 4 = 2^4 = 16 leaf splits, if trees do not stop early due to some critera
    - Determines if trees are built "depth-first" (level-wise, all nodes evaluated at each level as tree is built) or "best-first" (leaf-wise, each node is carried out to full depth as tree is built)
        - Order matters: application of early stopping criteria and pruning methods can result in very different trees
        - Because leaf-wise chooses splits based on their contribution to the global loss and not just the loss along a particular branch, it often will learn lower-error trees "faster" than level-wise, i.e. for a given number of leaves, leaf-wise will probably out-perform level-wise

- Feature importance
    - The more often a feature is used in the split points of a tree the more important that feature is
    - Calcualted by averaging the impurity-based feature importance of each tree
    - Impurity-based feature importances can be misleading for high cardinality features (many unique values). As an alternative, the permutation importances of regressor can be computed on a held out test set


### Histogram-based gradient boosting

- e.g. Scikit-learn's [HistGradientBoostingClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.HistGradientBoostingClassifier.html#sklearn.ensemble.HistGradientBoostingClassifier) and [HistGradientBoostingRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.HistGradientBoostingRegressor.html#sklearn.ensemble.HistGradientBoostingRegressor)
   - Samples are binned into integer-value bins (typically 256 bins) and histograms are computed
       - Orders of magnitude faster than [GradientBoostingClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.GradientBoostingClassifier.html#sklearn.ensemble.GradientBoostingClassifier) and [GradientBoostingRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.GradientBoostingRegressor.html#sklearn.ensemble.GradientBoostingRegressor) when the number of samples is larger than tens of thousands of samples
       - Building trees is the bottleneck of gradient boosting
       - Typical (non-hisogram) trees require sorting at each node for each feature (in order to compute gain of a split efficiently) therefore each node split has a complexity of `O(n_features * n * log(n))` (where n i number of samples)
       - Histogram trees require a histogram to be built at each node for each feature therefore each node split has a complexity of `O(n_features * n)` since building a histogram is `O(n)`
       - Also only need to consider splits for discrete bins
    - Built-in support for missing values
    - The size of the trees can be controlled through the max_leaf_nodes, max_depth, and min_samples_leaf parameters.
    - Early-stopping is enabled by default if the number of samples is larger than 10,000




### Gradient boosting regression trees

- Method [from scikit-learn docs](https://scikit-learn.org/stable/auto_examples/ensemble/plot_gradient_boosting_regression.html)
    - Load data
    - Train-test split
    - Fit model, calculate MSE on test set
    - Plot training deviance (performance on training & test sets as function of boosting iterations)
    - Plot feature importance

- Scikit-learn's `GradientBoostingRegressor` uses the `squared_error` loss, which works by training regression trees with residuals as their target and then updating the ensemble prediction in such a way to minimize the residuals using the same ideas as gradient descent
    - "If we can build a tree that finds some patterns between x and r, we can reduce the residuals from the initial prediction p by utilizing those found patterns."

- Custom regressor class from [Masui's blog post](https://towardsdatascience.com/all-you-need-to-know-about-gradient-boosting-algorithm-part-1-regression-2520a34a502)
```python
class CustomGradientBoostingRegressor:

    def __init__(self, learning_rate, n_estimators, max_depth=1):
        self.learning_rate = learning_rate
        self.n_estimators = n_estimators
        self.max_depth = max_depth
        self.trees = []

    def fit(self, X, y):

        self.F0 = y.mean()
        Fm = self.F0

        for _ in range(self.n_estimators):
            r = y - Fm
            tree = DecisionTreeRegressor(max_depth=self.max_depth, random_state=0)
            tree.fit(X, r)
            gamma = tree.predict(X)
            Fm += self.learning_rate * gamma
            self.trees.append(tree)

    def predict(self, X):

        Fm = self.F0

        for i in range(self.n_estimators):
            Fm += self.learning_rate * self.trees[i].predict(X)

        return Fm
```

### Gradient bossting classification trees

- Use log loss (cross-entropy loss), train regression trees with residuals as their targets and then update ensemble predictions
    - Probability calculations, e.g. the ensemble prediction update is done using log(odds) `log(p/1-p)`, i.e. the probability is converted to a log(odds) and then updated by a value gamma that seeks to minimize (in a similar way to gradient descent) the logistic loss on each terminal node

- Custom regressor class from [Masui's blog post](https://towardsdatascience.com/all-you-need-to-know-about-gradient-boosting-algorithm-part-2-classification-d3ed8f56541e)
```python
class CustomGradientBoostingClassifier:

    def __init__(self, learning_rate, n_estimators, max_depth=1):
        self.learning_rate = learning_rate
        self.n_estimators = n_estimators
        self.max_depth = max_depth
        self.trees = []

    def fit(self, X, y):

        F0 = np.log(y.mean()/(1-y.mean()))  # log-odds values
        self.F0 = np.full(len(y), F0)  # converting to array with the input length
        Fm = self.F0.copy()

        for _ in range(self.n_estimators):
            p = np.exp(Fm) / (1 + np.exp(Fm))  # converting back to probabilities
            r = y - p  # residuals
            tree = DecisionTreeRegressor(max_depth=self.max_depth, random_state=0)
            tree.fit(X, r)
            ids = tree.apply(x)  # getting the terminal node IDs

            # looping through the terminal nodes
            for j in np.unique(ids):
              fltr = ids == j

              # getting gamma using the formula (Σresiduals/Σp(1-p))
              num = r[fltr].sum()
              den = (p[fltr]*(1-p[fltr])).sum()
              gamma = num / den

              # updating the prediction
              Fm[fltr] += self.learning_rate * gamma

              # replacing the prediction value in the tree
              tree.tree_.value[j, 0, 0] = gamma

            self.trees.append(tree)

    def predict_proba(self, X):

        Fm = self.F0

        for i in range(self.n_estimators):
            Fm += self.learning_rate * self.trees[i].predict(X)

        return np.exp(Fm) / (1 + np.exp(Fm))  # converting back into probabilities

```

