---
date: 2024-01-31
tags:
    - book
hubs:
    - "[[machine-learning]]"
---
# MLOps Engineering at Scale
Author: Carl Osipov
https://github.com/osipov/smlbook

# Index
- ML systems
- Dataset considerations
- Data quality
- Object storage on AWS
-  AWS Glue data catalog
- AWS Athena
- Well-informed train-test-split
- PyTorch fundamentals
- Neural Networks
- Autograd, SGD and Adam
- Gradient descent with out-of-memory dataset and GPUs
- Distributed gradient descent
- Feature selection
- PyTorch Lightning
- Hyperparameter optimization
- Optuna
- ML pipeline
- Kaen
- MLFlow

# ML systems
- ML code is a small part of the ML system (~5%)
![[Pasted image 20240618083312.png]]
# Dataset considerations
- What is the business use case? (the objective)
- What are the business rules? (how the data was generated)
- What is the schema? (data types and formats)
- What are the options for implementing the business service? (practical considerations, e.g. cost, privacy)
- What data assets are available for the business service? (e.g. internal/external APIs)

# Data quality
- Focus on validating essential data (columns, or maybe records) first
- VACUUM
	- Valid: matches schema data type, is not null unless field is nullable, is in set of expected values, numeric or date range, and adheres to any rule (e.g. valid email, IP address, etc..)
	- Accurate: matches with reference datasets
	- Consistent: same standards for validity and accuracy across all data systems
	- Uniform: data within each column should be recorded using the same measurement system
	- Unified: combines as much useful data as possible in one place
# Object storage on AWS
- Files are immutable, globally identifiable with a URL and redundant across multiple availability zones
- Designed to be accessible using HTTP

# AWS Glue data catalog
- Run batch jobs, e.g. data cleaning
- Schema discovery
	- A Glue crawler connects to s3, identifies the format used by the data and analyzes the schema
	- IAM process: create new security role for Glue (`aws iam create-role`), attach Glue policy to that role so that is can use Glue services like the database (`aws iam attach-role-policy`), then assign a policy to the role to enable crawling of your specific s3 bucket (`aws iam put-role-policy`)
	- Create, start and monitor the crawler and resulting database with CLI commands: `aws glue create-database`, `create-crawler`, `start-crawler`, `get-crawler`, `get-table`
- Running ETL jobs with Glue
	- Write job as python file and upload to s3, e.g. loading data with spark:
			
			df = spark.read.format("csv").load(bucket_path)
			df.createOrReplaceTempView("view_name")
			query_df = slark.sql("select col1, col2, avg(m1) from view_name group by col1, col2")
			query_df.write.parquet(output_bucket_path)
	- Create, run and monitor job with Glue: `aws glue create-job`, `start-job-run`, `get-job-runs`

# AWS Athena
- Run SQL queries on object storage, uses "schema on read", can attach to other DBs that implement the Athena connector
- Create "table" (schema) with SQL, e.g. 

			CREATE EXTERNAL TABLE table_name(
				col1 STRING,
				col2 DOUBLE
			)
			LOCATION 's3://bucket-name/folder/'
			TBLPROPERTIES ('skip.header.line.count'='1');
- Useful for data exploration and validation

# Well-informed train-test-split
- General heuristics: use smaller validation & test sets when data is big - e.g. as low as 1% for "Netflix-sized" data - 98% (train), 1% (validation), 1% (test)
- The author argues that you can look at the standard error of the mean, SEM = σ/sqrt(N), as the sample size N increases.
	- However this will always follow the same pattern (i.e. y=1/sqrt(x)) so I don't understand why he thinks it's a good idea.
	- He doesn't demonstrate the technique on other datasets so I don't trust that it's worthwhile approach.
	- He finds 7% split (~1M records) to be appropriate based on the point of diminishing returns on the SEM improvement

# PyTorch fundamentals
- Author used Tensorflow for years and prefers Torch
- Heavy use of tensors, watch for overloaded arithmetic operators that do matrix operations (e.g. broadcast additions in high-dimensions)
- PyTorch tensors are more efficient than Python lists
	- C integers (like PyTorch and NumPy data types) are stored directly as assessable memory location chunks
	- Python integers are stored as address references to a generic PyObject_HEAD structure which specified the data type and the data value
	- Elements of a Python list might be scattered in non-continuous memory locations
	- Example to illustrate why having data in a continuous chunk is beneficial: a loop in C that adds a series of numbers stored in an array
		- The CPU starts the loop and needs to access the first number in the array. It fetches that data (and potentially a few surrounding numbers) from RAM and loads it into the cache.
		- As the loop continues, the CPU needs the next number in the array. Ideally, this data is already in the cache because it's located right next to the previously accessed number. The CPU can grab it much faster than going back to RAM.
		- This process continues throughout the loop, with the CPU accessing data from the cache whenever possible. This significantly improves the performance of the loop compared to fetching everything from RAM every time.

# Neural Networks
- Neural networks are a collection of nested functions that are executed on some input data. These functions are defined by _parameters_ (consisting of weights and biases), which in PyTorch are stored in tensors.
- Training involves
	- Forward Propagation: Calculate the output of the NN
	- Backward Propagation: Adjust parameters proportionate to the error of the result,  moving backwards through the NN and calculating the gradients (derivatives of the error with respect to the parameters of the functions) and optimizing the parameters using gradient descent.

# Autograd, SGD and Adam
- Automatic differentiation engine (autograd) powers neural network training in PyTorch - it calculates the gradients needed for learning.
- SGD and Adam are the algorithms that use those gradients to update the model's parameters and improve its performance.
- SGD is a simpler approach, while Adam is a more sophisticated optimizer that builds on SGD concepts.
- SGD
	- Minimizes a function (often the loss function in neural networks) by iteratively updating the model's parameters in the direction of the negative gradient.
	- Considers a subset of data (minibatch) to compute the gradient in each iteration.
	- A foundational algorithm, but it can be slow to converge and requires careful tuning of the learning rate.
- Adam
	- Similar to SGD, minimizes a function by updating parameters.
	- Updates weights and biases based on gradients, but also incorporates information from past gradients to adapt the learning rate for each parameter.
	- Aims to be more efficient than SGD and less sensitive to learning rate tuning.
    
# Gradient descent with out-of-memory dataset and GPUs

Code explanation: https://chatgpt.com/share/942cdc50-168e-4f2b-bc08-6a22403be1d1

```python
import os
import torch
from torch.utils.data import DataLoader
from kaen.torch import ObjectStorageDataset as osds

torch.manual_seed(0);
torch.set_default_dtype(torch.float64)

# Use GPU if it's available
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

BATCH_SIZE = # set batch size (e.g. 2**20)
BUCKET_PATH = # set bucket path
FEATURE_COUNT = # set number of features
LEARNING_RATE = # set learning rate (e.g. 0.03)
GRADIENT_NORM = # set gradient norm (e.g. 0.5)
ITERATION_COUNT = # set iteration count (e.g. 50)

# Create out-of-memory dataset
train_ds = osds("s3://{BUCKET_PATH}/*.csv", batch_size=BATCH_SIZE)
train_dl = DataLoader(train_ds)

# Initialize weights and biases
w = torch.nn.init.kaiming_uniform_(torch.empty(FEATURE_COUNT, 1, requires_grad=True, device=device))
b = torch.nn.init.kaiming_uniform_(torch.empty(1, 1, requires_grad=True, device=device))

def batch_to_xy(batch):
    batch = batch.squeeze_().to(device)
    return batch[:, 1:], batch[:,0]

def forward(X):
	y_pred = X @ w + b
	return y_pred.squeeze_()

def loss(y_est, y):
	mse_loss = torch.mean((y_est - y)**2)
	return mse_loss

optimizer = torch.optim.SGD([w, b], lr=LEARNING_RATE)

for iter_idx, batch in zip(range(ITERATION_COUNT), train_dl):
	start_ts = time.perf_counter()

	# Get the training data for this batch
	X, y = batchToXy(batch)

	# Calculate the predicted values `y_est` using the linear model (`y_est = X @ w + b`)
	y_est = forward(X)

	# Compute the mean squared error between the predicted values `y_est` and the actual target values `y`
	mse = loss(y_est, y)

	# Calculate the gradients of the loss with respect to the weights (`w`) and biases (`b`)
	mse.backward()

	# Clip the gradients to prevent them from becoming too large and causing instability during training (exploding)
	torch.nn.utils.clip_grad_norm_([w, b], GRADIENT_NORM)

	# Updates the model parameters (`w` and `b`) using the computed gradients and the learning rate
	optimizer.step()
    
    # Clears the gradients (by default, gradients in PyTorch accumulate after each backward pass)
	optimizer.zero_grad()

	sec_iter = time.perf_counter() - start_ts
	print(f"Iteration: {iter_idx}")
	print(f"Seconds/Iteration: {sec_iter}")
	print(f"MSE: {mse.data.item()}")
```


# Distributed gradient descent  
- Sharding is when data is split into chunks and stored or processed in separate places  
- Consider an example where we update the model weights once per full pass over the training data  
	- Sharding works with mini-batch gradient descent as well, but this example is easier to explain  
- Model params (weights) are stored in one node and updated once per full pass of the training data  
- Training data shards are sent to separate notes and the losses are computed in parallel 
- Cumulative loss function gradients are computed in parallel and sent to the parameter node  
- The parameter node computes an update to the weights once it receives all the gradients from that epoch (pass over the training data) and then it sends out the new weight values to each training node  
- This method is limited by bandwidth, both sending weights to the parameter node and sending weights back to the training nodes  
	- Consider that we want to have many training nodes (hundreds) and many weights (millions)  
- More advanced parallel processing algorithms solve this bandwidth problem, eg “Horovod” which uses logical ring-based gradient descent  
  
  
  
# Feature selection  
- Adding a large number of features can be costly (curse of dimensionality)  
- Over-engineering features can hurt performance (overfitting or adding bias)  
- Guiding principles  
	- Related to label  
	- Recorded before inference time  
	- Supported by abundant examples  
	- Expressed as a number with a meaningful scale  
	- Based on expert insights about the project  
  
## Related to label  
- Can you explain why the feature is related to the label?  
- Prioritize features with a “counterfactual causation”  
	- ie would you expect the label (or label probability) to have been different if the feature value had of been different  
	- In this way we can attempt to select for causal features over confounding features (eg age over shoe size, for predicting reading ability of children)  
	
## Recorded before inference time  
- Only use features that will be available at prediction time
	- Eg weight of baby at birth cannot use pregnancy duration  
	
## Supported by abundant examples  
- Make sure there are not too many distinct values or missing values  
	- Are the number of distinct values comparable to the number of records?  
	- Is the feature 95% NaN?  

## Expressed as a number with a meaningful scale  
- Eg text or image embeddings  

## Based on expert insights about the project  
- Work with subject matter experts (SMEs)  
- The most productive way to encourage the SMEs to propose useful features is to ask them,  
	- "What information would you share with your team to help them better estimate the label value?"
	- "How can this information be gleaned from the existing data in the project data set?"  
- Want them to think about the problem in natural, human-centric terms
  
  
# PyTorch Lightning  
- Reduce boilerplate code e.g.  
	- Iteration over training batches and epochs  
	- Checkpointing model weights to storage  
	- Training, validation and testing  
	- Logging features like CVSLogger

## PyTorch Lightning example

```python
model, trainer = train(
	DcTaxiModel(**{
		"seed": "1686523060",
		"num_features": "8",
		"num_hidden_neurons": "[3, 5, 8]",
		"batch_norm_linear_layers": "1",
		"optimizer": "Adam",
		"lr": "0.03",
		"max_batches": "1",
		"batch_size": str(2 ** 18),
	}),
	**data_paths
)
	
print(trainer.callback_metrics)
```

```python
import torch as pt
import pytorch_lightning as pl
from torch.utils.data import DataLoader

        
def train(model, train_glob, val_glob, test_glob = None):
    trainer = pl.Trainer(
        max_epochs = 1,
		...
	)
    
    train_dl = DataLoader(...)
    val_dl = DataLoader(...)
    test_dl = DataLoader(...)

    trainer.fit(
	    model, 
		train_dataloaders = train_dl,
		val_dataloaders = val_dl,
	)

	trainer.test(model, dataloaders=test_dl)

    return model, trainer
```

```python
import sys
import json
import time
import torch as pt
import pytorch_lightning as pl
from distutils.util import strtobool

class DcTaxiModel(pl.LightningModule):
    def __init__(self, **kwargs):
		super().__init__()
		self.save_hyperparameters()

		# Initialize neural network layers
		...

	# More functions
	...

    def forward(self, X):
	    y_est = self.layers(X)
		return y_est.squeeze_()
        
    def training_step(self, batch, batch_idx):
        self.step += 1
        X, y = self.batchToXy(batch) #unpack batch into features and label
        y_est = self.forward(X)
        loss = pt.nn.functional.mse_loss(y_est, y)
        self.train_val_rmse = loss.sqrt()

        return loss

    def validation_step(self, batch, batch_idx):
        X, y = self.batchToXy(batch) 

        with pt.no_grad():
            loss = pt.nn.functional.mse_loss(self.forward(X), y)
          
        return loss
      
    def test_step(self, batch, batch_idx):
        X, y = self.batchToXy(batch) 

        with pt.no_grad():
            loss = pt.nn.functional.mse_loss(self.forward(X), y)

    def configure_optimizers(self):
        optimizers = {'Adam': pt.optim.AdamW,
                      'SGD': pt.optim.SGD}
        optimizer = optimizers[self.hparams.optimizer]

        return optimizer(self.layers.parameters(), 
                            lr = float(self.hparams.lr))
```

# Hyperparameter optimization
- Hyperparameter optimization techniques are used to find the best hyperparameters to model the data
- Unlike model params like weights and biases, which are optimized using e.g. SGD, hyperparams sit a level above this, e.g. learning rate, batch size, num hidden neurons, etc
- Find optimal neural net architecture, as opposed to creating an arbitrary one

# Optuna
- Optuna is a PyTorch framework for doing this, it uses the concept of a `trial` which is a training instance (i.e. an outer loop on the epochs)
- A `study` is used to run a set of `trials`, e.g.

			def objective(trial):
				hparams = {
					"num_features": 8,
					"optimizer": trial.suggest_categorical('optimizer', ['Adam', 'SGD'])
					"lr": trial.suggest_loguniform('lr', 0.009, 0.07),
				}

			import optuna
			from optuna.samplers iport TPESampler
			study = optuna.create_study(direction="minimze', sampler=TPESampler(seed=42))
			study.optimize(objective, n_trials=100)
			study_df = study.trials_dataframe()

- Use log-normal sampling when hyperparam values can span orders of magnitude
	- e.g. `{"lr": trial.suggest_loguniform('lr', 0.001, 0.1)}`,
	
- Set custom hyperparam ranges with `trial.suggest_categorial`, e.g.

		{
			"num"hidden_layers": [
				trial.suggest_categorical(
					f'num_hidden_{layer}_neurons', [7, 11, 13, 19, 23]
				)
				for layer in range(
					trial.suggest_categorical('num_layers', [11, 15, 19])
				)
			]
		}

# ML pipeline  
- Goal: input data and output model (feature eng, serving, etc.. are out of scope)
- Must be able to manage experiments and hyperparameter optimization
- Docker containers facilitate packaging, deployment, and integration of ML pipeline with cloud services
 
## Tools  
- MLFlow: Open source experiment management  
- Optuna: Hyperparameter optimization  
- Docker: Pipeline code packaging 
- PyTorch Lighting: Model training and validation  
- Kaen: Integration with AWS or other cloud providers


# Kaen
- Python library and CLI for running ML pipelines
- Local provider for testing and prototyping
- Cloud providers for production model training
- Maintained by book author but only a few stars on GitHub, so I would search for a more popular alternative
	- https://github.com/CounterFactualAI/kaenai


# Mlflow  
- Kaen provides a `BaseMLFlowClient` class, which can be used to implement an MLFlow-managed experiment
```python
from model_v1 import DcTaxiModel
from trainer import train
from kaen.hpo.client import BaseMLFlowClient

class DcTaxiExperiment(BaseMLFlowClient):
    
    def on_run_start(self, run_idx, run):

        #create a set of default hyperparameters
        default_hparams = {"seed": "1686523060",
                        "num_features": "8",
                        "num_hidden_neurons": "[3, 5, 8]",
                        "batch_norm_linear_layers": "1",
                        "optimizer": "Adam",
                        "lr": "0.03",
                        "max_batches": "1",
                        "batch_size": str(2 ** 18),}        
        
        #fetch the MLFlow hyperparameters if available
        hparams = run.data.params if run is not None \
                    and run.data is not None else \
                    default_hparams
        
        #override the defaults with the MLFlow hyperparameters
        hparams = {**default_hparams, **hparams}

		untrained_model = DcTaxiModel(**hparams)
        model, trainer = train(untrained_model)

```

