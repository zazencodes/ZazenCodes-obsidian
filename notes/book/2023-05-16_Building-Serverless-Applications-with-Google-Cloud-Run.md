---
date: 2023-05-16
tags:
    - book
hubs:
    - "[[gcp]]"
---

# Building Serverless Applications with Google Cloud Run


### Using a Cloud SQL backend for your app

Provision service
```bash
gcloud services enable sqladmin.googleapis.com
gcloud services enable sql-component.googleapis.com

gcloud sql instances create sql-db \
    --tier db-f1-micro \
    --database-version MYSQL_8_0 \
    --region us-central1
```

Delete root user and restrict to "Cloud SQL Client" IAM role
```bash
gcloud sql users delete root --host % instance sql-db
gcloud sql users create root --host "cloudsqlproxy~%" --instance sql-db
```

Disable direct connections (only allow cloud SQL proxy)
```bash
gcloud sql instances patch --require-ssl sql-db
# now nobody can connect unless you generate client certificates for them
```

### Web App Session Storage
- Web app frameworks tend to store session storage on local server, e.g. logged in status, form data, etc..
- It's better to persist this on disk e.g. Redis using MemoryStore cloud service, Firestore cloud service
    - Reads and writes happen in between request and response and must be very fast
    - Session data indexed with session ID as key, with cryptographic signature to prevent users from manually swapping the session ID in their cookie

### Policy Bindings
- Binds a member to a role, e.g. a Google account to Cloud SQL Client, All users (anyone on the internet) to Cloud Run Invoker, a service account to BigQuery Admin, etc..
- A policy binding can be added (scoped) to a project: the member can connect to all specified resources in the project
- A policy binding can be added (scoped) to a resource: the member can connect to only that resource
    - e.g. `gcloud run services add-iam-policy-binding [SERVICE] --member serviceAccount:[SERVICE-ACCOUNT] --role roles/run.invoker`
    - this allows a service account to be a cloud run invoker on only one specific cloud run service

### Authentication using identity tokens
```bash
ID_TOKEN=$(gcloud auth print-identity-token)
curl -H "Authorization: Bearer $ID_TOKEN" [PRIVATE_ENDPOINT_URL]
```

### Deploying a web app with backend API
```bash
PROJECT=$(gcloud config get-value project)
FRONTEND_IMAGE=us-docker.pkg.dev/$PROJECT/repo/image_id

gcloud builds submit frontent --tag $FRONTEND_IMAGE
gcloud run deploy frontend --image $FRONTEND_IMAGE --allow-unauthenitcated

PRODUCT_IMAGE=us-docker.pkg.dev/$PROJECT/repo/image_id

gcloud builds submit product-api --tag $PRODUCT_IMAGE
gcloud run deploy product-api --image $PRODUCT_IMAGE --no-allow-unauthenticated

# tell the frontend about the API
# the two can communicate using the default service credentials
gcloud run services update frontend --set-env-vars PRODUCT_API=[CLOUD_RUN_API_URL]

# add custom service accounts (principle of least permission)
gcloud iam service-accounts create frontend
gcloud iam service-accounts create product-api
gcloud run services update frontend --service-account frontend@$PROJECT.iam.gserviceaccount.com
gcloud run services update product-api --service-account product-api@$PROJECT.iam.gserviceaccount.com

# add IAM policy binding
# add frontend service account (which has been added to the frontend cloud run service) as a member on the API cloud run service with permission to invoke it
gcloud run services add-iam-policy-binding product-api --member serviceAccount:frontend@$PROJCT.iam.gserviceaccount.com --role roles/run.invoker
```

### Scheduling jobs with Cloud Tasks Queue
- Alternate to pub/sub message queues which are less attuned to the use case of scheduling tasks
- Cloud Workflows service is designed to handle multiple tasks that must be chained together
```bash
gcloud services enable cloudtasks.googleapis.com

# create task queue named myjobs
gcloud tasks queues create myjobs

# allow cloud run app to enqueue tasks
gcloud tasks queues add-iam-policy-binding [QUEUE_NAME] \
    --member serviceAccount:task-app@$PROJECT.iam.gserviceaccount.com \
    --role roles/cloudtasks.enqueuer
```





