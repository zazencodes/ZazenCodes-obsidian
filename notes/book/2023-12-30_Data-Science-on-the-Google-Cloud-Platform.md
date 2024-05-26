---
date: 2023-12-30
tags:
  - book
hubs:
  - "[[gcp]]"
  - "[[data-science]]"
  - "[[machine-learning]]"
urls: https://github.com/GoogleCloudPlatform/data-science-on-gcp
---
# Data Science on the Google Cloud Platform
Author: Valliappa Lakshmanan

# Index
- Bash workflows
- Theory
- Cloud Scheduler
- ML dashboards
- Model training
- Exploratory analysis
- Dataproc
- Bayesian stats
- Feature engineering
- Practical considerations of ML
- TensorFlow
- Streaming ML
- Continuous training
- Sensitive data considerations

# Bash workflows

## Bash tricks

```bash
# make temp dir and store as variable
DIR=$(mktemp -d)

# bash loop
for i in `seq 1 12`; do
    echo i
done

# replace , with space using sed
cat [file] | sed 's/,/ /g'
```

## Google buckets

- Common pattern for Google cloud buckets is to append the project ID to the start of the bucket name, to guarantee global uniqueness.

- Cheaper and faster to localize to single region for data storage and analysis.

```bash
# make bucket
BUCKET=$(gcloud config get-value project)-mybucket
REGION=us-central1
gsutil mb -l $REGION gs://$BUCKET


# copy files to bucket (-m flag = multithread upload)
gsutil -m cp *.csv gs://$BUCKET/path
```

## BigQuery

Load tables with the `bq` CLI:
```bash
bq --project-id $(gcloud config get-value project) load \
    --time_partitioning_field=date \
    --time_partitioning_type=day \
    --source_format CSV \
    --ignore_unknown_values \
    --skip_leading_rows 1 \
    --schema "COL1:STRING,COL2:DATE,..." \
    dataset.tablename file.csv
```

Unload queries with the `bq` CLI:
```bash
bq query --destination_table dataset.table_name --replace --nouse_legacy_sql '[select_stmt]'

bq extract --destination_format=NEWLINE_DELIMITED_JSON dataset.table_name \
    gs://bucket/path/data.json
```

Avoiding SQL injection when using python library:
```python
start_time = # set start timestamp
end_time = # set end timestamp
bqclient = bq.Client(project)
querystr = """
select * from table_name
where event_time between @startTime and @endTime
"""
job_config = bq.QueryJobConfig(query_parameters=[
    bq.ScalarQueryParameter("startTime", "TIMESTAMP", start_time),
    bq.ScalarQueryParameter("endTime", "TIMESTAMP", end_time),
])
rows = bqclient.query(querystr, job_config=job_config)
```

Top rows for each group, equivalent of `groupby(col).head(1)` operation in pandas:
```sql
select
    airport,
    array_agg(
        struct(metric_1, metric_2, ...) order by order_metric limit 1
    ) a
from dataset.table
group by airport;
```

# Theory
- CAP theorem: can only guarantee 2/3 of Consistency, Availability and Partition resilience. In reality this means trade off between Consistency and Availability (b/c Partitions will always be prone to dying off and therefore Partition resilience is an absolute requirement)

- ROC plots true positive rate as a function of the false positive rate
	- True positive = how many y=1 labels are correct, higher = better
	- False positive = how many y=1 labels are incorrect, higher = worse
	- In a random classifier this is linearly increasing
	    (e.g. label no y=1, therefore have 0% TP and perfect FP at (0,0), progressing to label all y=1, therefore have perfect TP and 100% FP at (1,1))
	- Perfect ROC has area under the curve (AUC) of 1

- Precision is the percent of y=1 labels that are correct
- Recall is the percent of y=1 labels that were accurately predicted

- XGBoost is a random forest algorithm where trees are generated using information about how labels were misclassified by previously generated trees. The improved trees are "boosted"

- Generally speaking, outliers twist and warp the decision space in order for outliers to achieve the proper labels, thus resulting in large weights. L2 regularization solves this by imposing a penalty on large weight values, therefore reducing the impact of outliers


# Cloud Scheduler

## Invoke HTTP endpoint on Cloud Run
#### Flask API:

Dockerfile
```Dockerfile
FROM python:3.8-slim
END APP_HOME /app
WORKDIR $APP_HOME
COPY . ./
RUN pip install --no-cache-dir -r requirements.txt
# Run API with timeouts disabled
CMD exec gunicorn --build :$PORT --workers 1 --threads 8 --timeout 0 main:app
```

main.py
```python
from flask import Flask

app = Flask(__name__)

...
```

#### Deploy to cloud run:

Create service account
```bash
gcloud iam service-accounts create account-name --display-name "Display name"
SVC_PRINCIPAL=serviceAccount:account-name@$(gcloud config get-value project).iam.gserviceaccount.com
```

Attach to bucket
```bash
BUCEKT=$(gcloud config get-value project)-bucket-name
gsutil mb -l us-central1 gs://$BUCKET
gsutil uniformbucketlevelaccess set on gs://$BUCKET
gsutil iam ch serviceAccount:${SVC_PRINCIPAL}:roles/storage.admin gs://$BUCKET
```

Attach to bigquery dataset
```bash
PROJECT_ID=$(gcloud config get-value project)
bq --project_id $PROJECT_ID query --nouse_legacy_sql \
"GRANT `roles/bigquery.dataOwner` ON SCHEMA table_name TO '$SVC_PRINCIPAL'"
gcloud projects add-iam-policy-binding $PROJECT_ID --member $SVC_PRINCIPAL --role roles/bigquery.jobUser
```

Test
- Download new JSON key
- Impersonate service account: `gcloud auth activate-service-account --key-file key.json`
- Run app

Deploy cloud run
```bash
SVC_EMAIL=account-name@$PROJECT_ID.iam.gserviceaccount.com
gcloud run deploy app-name --region us-central1 --source=$(pwd) --platorm=managed --timeout 12m --service-account $SVC_EMAIL --no-allow-unauthenticated
```

Invoke cloud run
```bash
curl -X POST $URL \
    -H "Content-Type:application.json" \
    -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
    --data '{"year": 2020, ...}'
```

Schedule
```bash
gcloud scheduler jobs create http appname \
--description "App description" \
--schedule "$CRON_SCHEDULE" \
...
--uri=$SVC_URI \
--http-method POST \
--oidc-service-account-email $SVC_EMAL \
--oidc-token-audience $SVC_URL \
...
```


# ML Dashboards

- The aim of dashboard creation is to crowd-source insight into the working of models from end users - it's about presenting an explanation of the models.

- Why build it?
	- Show users how model behaves
	- Observability into data that powers model
	- Model explainability

- Design this first, in order to
	- Get user feedback
	- Exploring and monitoring data yourself
	- Eventual use in production

- What to visualize?
	- Confusion matrix
	- Threshold optimization (accuracy VS some parameter)


# Exploratory data analysis

- The aim of EDA is for the data engineer to develop insights about the data before developing models

- You should do the following
	- Test underlying assumptions
	- Identify important variables using logic and understand them
	- Discover relationships between variables and other patterns (structures)
	- Develop simple models with explanatory power - understand where and why they succeed and (more importantly) fail
	- Detect outliers and anomalies
	- Detect data quality problems
	
- GCP's Vertex AI = fully managed Jupyter notebooks


# Model training

- When splitting data, don't always want to do random samples. e.g. we have correlated data within each day, therefore the data should be split into training and testing *days* (no intraday splitting)

# Dataproc

- Often used for legacy reasons. On-prem Hadoop workloads are lifted and shipped to Dataproc and optimized to take advantage of cloud computing (e.g. ephemeral clusters). Modernized over time to BigQuery & Dataflow jobs

- Dataproc offers more control over resources than Dataflow

- Secondary workers are available at significant (~80%) discount. These are made available for your cluster but can be taken away within a minute notice. It is economical to use them because they can drastically increase the speed of jobs and therefore the time you need to pay for compute

# Bayesian stats

- Most basic Bayes classification approach:

	- Discretize dataset along select features, e.g. table below, where xi and yi represent feature splits (e.g. x=gender, y=age bucket)
	- Compute probability of target for each bin
	- Determine classification (and % likelihood) using lookup table of select features
	- This does not scale well (exponentially with number of features)

|     | p   | ... | ... | ... |
| --- | --- | --- | --- | --- |
|     | x1  | x2  | x3  | ... |
| y1  |     |     |     |     |
| y2  |     |     |     |     |
| ... |     |     |     |     |

- Naive Bayes approach
	- Scalable solution where features are treated in isolation. It requires (mathematically) that they are independent
	- Discretize dataset along select features, e.g. table below, where xi and yi represent feature splits (e.g. x=gender, y=age bucket) 
	- Compute probability of target for each bin (one-dimensional)
	- Determine probabilities of feature combinations by multiplying conditional probabilities of each (the bin values)
	- Determine classification using lookup table of select features
	- % likelihood determined by Bayes rule normalization (multiply by `P(target) / P(features)` = `P(target) / (P(x1) * P (y1) ...)`)
	- Scales linearly

|     | p   |
| --- | --- |
| x1  |     |
| x2  |     |
| x3  |     |
| ... |     |
| y1  |     |
| y2  |     |
| y3  |     |
| ... |     |

# Logistic Regression

- Algorithm
	- Use a function `L = b + x1*w1 + x2*w2 + ...` , where `xi` are features, `wi` are weights and `b` is an intercept
	- Probabilities between 0 and 1 are given by transforming `L` using the logistic function: `P = 1 / (1 + e^-L)`
	- Pick weights minimizing the cross entropy or equivalent "loss" function (e.g. `sum(ln(1+e^(-y*L)))` for L-BFGS optimization used in pyspark)

# Feature engineering

- Common practices
	- Standardize variables, e.g. from -1 to 1 (min/max) or set mean as 0 and standard deviations as -1 / 1
	- Cut variables (set outliers to max value)

- Time of day transformation trick
	- Times like e.g. 22 and 2 are only 4 hours apart. But ML model will interpret them as 20 apart
	- Solve this in two ways
	    - 1. Bucket into logical groups, e.g. Morning, Mid-day, Evening, Night, etc..
	    - 2. Transform as `sin(theta)` and `cos(theta)` , where `theta` = angle on clock (`rad(360 * hour / 24*)`)

# Practical considerations of ML

- Having complex library code carry out feature transformations makes doing predictions hard. Not possible to run on devices - needs to go through specialized server and back
    - BigQuery ML (managed compute) and Vertex AI (managed compute, REST API) solve this issue
	
- Spark is not the best framework for ML - better to use XGBoost, Pytorch or TensorFlow in Vertex AI

- BigQuery ML useful for training models and making batch predictions, models can be exported as TensorFlow objects and run elsewhere to make online predictions

- Use AutoML and BigQuery ML whenever you can (low-code solutions) confident that you can drop down to Vertex AI and TensorFlow if necessary. Don't overcorrect and use only TensorFlow code - you will be sacrificing a lot of productivity if you thumb your nose at low-code and no-code tools

# BigQuery ML

Train logistic regression model:
```sql
create or replace model dataset.model
options(input_label_cols=["ontime"],
       model_type="logistic_reg",
       data_split_method="custom",
       data_split_col="is_eval_day")
as select ontime, dep_delay, taxi_out, distanct, is_eval_day
from dataset.training_data
```

BigQuery useful for making batch predictions:
```sql
select * from ml.predict(model dataset.model, (
    select dep_delay, taxi_out, distance
    from dataset.new_data
))
```

- For online models (making single predictions as needed) extract the model as a TensorFlow model and deploy to Vertex AI

## Hyperparameter tuning

- Good to adjust amount of L2 regularization, as default arbitrary value is used

Hyperparameter searching with Vertex AI:
```sql
create or replace model dataset.model
options(input_label_cols=["ontime"],
       model_type="boosted_tree_classifier",
       num_trials=5,
       l2_reg=hparam_range(0.5, 3.0),
       max_tree_depth=hparam_range(2,10),
       data_split_method="custom",
       data_split_col="is_eval_day")
```

- Note that "is_eval_day" has 3 values: train, eval and test (test set is withheld from hyperparam search)

## Going from BQ to Tensorflow

- Go from BigQuery to TensorFlow
	- Load as pandas dataframe and use `tf.convert_to_tensor` if data will fit into memory
	- Export subset of BigQuery data to Google Storage (GCS)
	- Train model in BigQuery ML using `model_type=dnn_classifier` or `automl_cassifier` and export to use in Vertex AI

Export to GCS using terminal
```bash
bq extract --destination_format CSV dataset.table gs://bucket/key/table.csv
```

## Going from BQ to Vertex AI

Export model
```bash
EXPORT_PATH=gs://bucket/model_key/
bq extract -m dataset.model_name $EXPORT_PATH
```

Deploy (using e.g. docker image for tensorflow runtime)
```bash
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
MODEL_NAME=flights-${TIMESTAMP}
CONTAINER=us-docer.pkg.dev/vertex-ai/prediction/tf2-cpu.2-6:latest
gcloud ai models upload --region=$REGION \
    --display-name=$MODEL_NAME \
    --container-image-uri=$CONTAINER \
    --artifact-uri=$EXPORT_PATH
```

- Nearly all BQ ML models can be exported as tensorflow models and run with the container above.
	- Container e.g. for XGBoost: `us-docer-pkg.dev/vertex-ai/prediction/xgboost-cpu.0-82:latest`

Deploy endpoint: 
```bash
gcloud ai endpoints create --region=${REGION} --display-name=${ENDPOINT_NAME}
```

- Note that ENDPOINT_NAME will not change even though above MODEL_NAME will change as different models are deployed

Deploy model to endpoint:
```bash
gcloud ai endpoints deploy-model $ENDPOINT_ID \
    ...
    --machine-type=n1-standard-2 \
    --min-replica-count=1
    --max-replica-count=1
    --traffic-split=0=100
```

- If deploying a new model want to test, can specify how much traffic to send its way with e.g. `--trafic-split=0=10,OLD_DEPLOYED_MODEL_iD=90`

HTTP post invokation example
```bash
curl -X POST \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    -H "Content-Type: application/json" \
    -d @post_data.json
    "https://..."
```

# MLOps

- A production ML model must do the following
	- Be in version control
	- Entirely code driven - can automatically re-trigger training with a GitHub Action when new code is checked in
	- Invokable from a single entry point - can re-train by non-code changes such as new data arriving in a storage bucket
	- Be easy to monitor resource consumption (e.g. are GPUs getting saturated? -> get more GPUs) and model performance (e.g. has the accuracy dropped off? -> re-train)

- Model explanation
	- Global feature importance and local explanation (explaining an individual prediction) are different. There are features that are unimportant in the aggregate but can make a lot of difference to individual samples. e.g. when most globally important features are close to average, then other features can become more important

# Streaming ML

Making predictions with VertexAI:
```python
# get endpoint stub: a local proxy for the remote object
# (that abstracts away the details of the HTTP request -
# like data serialization and authentication)
endpoint = aiplatform.Endpoint.list(
    filter=f'display_name="{ENDPOINT_NAME}"',
    order_by='create_time desc',
)[0]
preds = endpoint.predict(input_data)
probs = [p[0] for p in preds.predictions]
```

e.g. of `input_data`:
```json
[
    {
        "dep_hour": 2, "is_weekday"; true, ...
    },
    ...
]
```

Re-using stubs across all Beam workers (useful if loading model from saved binary, which is time consuming and costs memory)
```python

class FlightsModelInvoker(beam.DoFn):
    def __init__(self, shared_handle):
        self._shared_handle = shared_handle
    def process(self, input_data):
        def create_endpoint():
            from google.cloud import aiplaform
            endpoint_name = 'flights-ch11'
            endpoint = aiplatform.Endpoint.list(
                filter=f'display_name="{ENDPOINT_NAME}"',
                order_by='create_time desc',
            )[0]
        endpoint = self._shared_handle.acquire(create_endpoint)
        preds = endpoint.predict(input_data).predictions
        for ids, input_instance in enumerate(input_data):
            res = input_instance.copu)_
            res["prob"] = preds[idx][0]
            yield res

handle = beam.utils.shared.Shared()
preds = features | 'pred' >> beam.ParDo(FlightsModelInvoker(handle))

```

Above approach is not actually useful for us here since the model is hosted on VertexAI. Instead we want a per-worker instance as follows:
```python
class FlightsModelInvoker(beam.DoFn):
    def __init__(self):
        self.endpoint = None
    def setup(self):
        from google.cloud import aiplaform
        endpoint_name = 'flights-ch11'
        self.endpoint = aiplatform.Endpoint.list(
            filter=f'display_name="{ENDPOINT_NAME}"',
            order_by='create_time desc',
        )[0]
    def process(self, input_data):
        preds = self.endpoint.predict(input_data).predictions
        for ids, input_instance in enumerate(input_data):
            res = input_instance.copu)_
            res["prob"] = preds[idx][0]
            yield res
```

(note we instantiate the endpoint in the setup method to hook into the lifecycle of the DoFn properly - the first time it's executed on the worker)

## Batching predictions in Beam

- Batching is more efficient (cheaper, faster).

Beam will tune the batch sizes with this code:
```python
preds = (
    features
    | 'batch_instances' >> beam.BatchElements(
        min_batch_size=1, max_batch_size=64,
    )
    | 'model_pred' >> beam.ParDo(FlightsModelInvoker())
)
```

## Streaming sinks

- Data can stream to a table or flat file
	- Bigquery
	    - Stream directly is good solution is latency requirements of a few ms are OK and throughputs are less than 1 GBps
	- Cloud BigTable
	    - Massively scalable NoSQL, good solution if needing extreme throughputs and low latency
	- Google storage (text files)
	    - Ideal for analytics. Not good if use case is primarily for serving result to user
	    - If periodic batch update to DB are OK, then this is a great streaming solution because it's cost efficient
	        - e.g. stream to files sharded by hour of day, batch write hourly files to bigquery
	- Cloud SQL
	    - If transactions are needed, or records need to be updated frequently. Medium scale
	- Cloud Firestore
	    - Allows for much larger datasets in NoSQL schemas
	- Cloud Spanner
	    - Massively scalable, extreme guarantees on availability, latency, etc. SQL-queryable

# Continuous training

- A necessary part of ML

- Fine-tuning
    - Start from a saved model and update it according to new data
    - Slowly adapt without losing accumulated knowledge
    - Can freeze certain layers and only train others
    - Not as effective as training from scratch
        - Once new data exceeds ~5% of total data, best to re-train from scratch to get optimal accuracy and model benefits

# Sensitive data considerations

- Sensitive data in columns
    - Decide which columns have sensitive data, decide how to secure them and document your decisions
    - Consider that fields on their own may be OK, but when combined can become sensitive. In these cases need to remove one or more to mitigate
	
- LLMs
    - For redacting information such as credit cards from training data, regex can be used. Google's Data Loss Prevention API can help here
	
- Free-form unstructured data
    - Cloud Natural Language API can help identify entities, email addresses, etc..
    - Speech-to-text can help for audio recordings
    - Cloud Vision API
        - Detect text
        - Mask faces

- Protecting data
    - Removal of sensitive info from ML team often not ideal - lessens model performance
    - Create a view on data that redacts sensitive fields
    - Remove and replace sensitive fields with a generic string

- Masking sensitive data
    - Substitution cipher: replace all occurrences of plan-text identifier with hashed and/or encrypted values (e.g. SHA-256 hash or AES-256 encryption)
    - Tokenization adds an extra layer by pushing the hashed/encrypted values into a separate database (lookup using token)
    - PCA and other dim. reductions can obfuscate fields, but sometimes at the cost of accuracy and explainability
    - Consider that data is still vulnerable to frequency analysis attacks (e.g. most commonly occurring name token in the US is likely the most common name in the US)

- Coarsening sensitive data
    - Locations, numeric quantities, IPs, etc..

- Governance policy
    - Establish secure location for governance docs
    - Exclude encryption keys from docs
    - Document
        - Sources of incoming data
        - Locations of sensitive data internally and steps taken to protect it
        - Instances where remediation steps are difficult or impossible, e.g. frequency attack vector
        - Roles and employees granted access
        - Process by which employees request access
    - Establish processes to
        - Continually scan for new sensitive data
        - Review who has access to what data
        - Communicate and enforce policies

