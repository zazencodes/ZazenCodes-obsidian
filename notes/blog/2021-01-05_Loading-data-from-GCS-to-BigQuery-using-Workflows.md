---
date: 2021-10-05
tags:
    - blog
hubs:
    - "[[gcp]]"
    - "[[bigquery]]"
urls:
    - https://cloud.google.com/architecture/serverless-orchestration-loading-data-from-cloud-storage-to-biquery-using-workflows?hl=en
---

# Loading data from GCS to BigQuery using Workflows

- Case
    - Retail platform collects sales events from various sources and writes them to google cloud storage
    - Events need to be imported into bigquery
    - Want serverless solution in 2 parts
        - File list to track unprcessed gs files
            - Cloud function triggered by object finalize storage event
            - Filename appended to array in Firestore collection
        - Workflow runner to import files into bigquery
            - Cloud scheduler triggers workflows
            - Workflows are YAML-based
            - Cloud functions transform data in bigquery
                - Create bq load job
                - Check load job status
                - Create transform query job
                - Poll status of transform job

- Setup
    - Create Firestore DB from console
    - Clone [project repo](https://github.com/GoogleCloudPlatform/bigquery-workflows-load)
    - Create resources from cloud shell using terraform
        - a [terraform file](https://github.com/GoogleCloudPlatform/bigquery-workflows-load/blob/main/main.tf) describes roles and resources to be created. Can be run with `terraform apply ...`
    - Create workflow from cloud shell
        - a [workflow](https://cloud.google.com/workflows) is created with [this yaml file](https://github.com/GoogleCloudPlatform/bigquery-workflows-load/blob/main/workflow.yaml). Can be done with `gcloud workflows deploy NAME ...`
    - Create python virtual env in cloud shell

            sudo apt-get install -y python3-venvpython3 -m venv env. env/bin/activatecd generatorpip install -r requirements.txt

    - Handling new files added to gs
        - Cloud function defined in terraform like this

                resource "google_cloudfunctions_function" "change_handler" {
                  name = "file_add_handler"
                  description = "Add file name to Firestore new collection"
                  ...
                  event_trigger {
                    event_type = "google.storage.object.finalize"
                    resource = google_storage_bucket.demo_bucket.name
                  }
                  ingress_settings = "ALLOW_INTERNAL_ONLY"
                  entry_point = "handle_new_file"
                }

    - Running and scheduling the workflow
        - It can be run in cloud shell `gcloud workflows execute NAME`
        - It can be scheduled in cloud scheduler

