# Setup ML Environment
## Requirements
* *RHODS on OCP on AWS with NVIDIA GPUs* RHDP Cluster
* OpenShift Data Science operator
* OpenShift Pipelines operator
* oc logged in to cluster

## Setup
* Install Codeflare operator (for simple object storage)
* Clone https://github.com/mamurak/os-mlops.git
* Navigate to *manifests* folder
* Run *oc apply -f projects.yaml*
* Run *oc apply -k .* (this uses the kustomization.yaml file)

Once the manifests have been deployed, your environment contains some content needed for demos (check for yourself):

* A Minio instance as a lightweight S3 storage provider. You can manage the S3 buckets through the Minio UI through the minio-ui route URL in project minio. Use minio and minio123 for logging in.

* A Data Science Project fraud-detection for running the fraud detection demo. The pipeline server is instantiated and cluster storage and data connections are configured.

* A Data Science Project object-detection for running the object detection demo. The pipeline server is instantiated and cluster storage and data connections are configured. The OVMS model server is instantiatend for model deployment.

* A Data Science Project huggingface-demo for running the Huggingface demo. Cluster storage is configured.

* A number of community workbench images.

* A number of custom serving runtimes.

# Start
## Instantiate Workbench

The demo content includes a number of usecases, we'll use the object detection scenario.

* In your OpenShift cluster, find the route to the RHODS dashboard (*rhods-dashboard*) and open the dashboard (login with cluster credentials)

* Enter the object-detection project.

* Create a new workbench with an arbitrary name and these parameters:
  * Image: Object detection
  * Existing cluster storage: development
  * Existing data connection: object-detection
  * Click **Create Workbench**

* Wait for the workbench to be started, click *Starting...* near *Status* and watch the *Event Log*
* Open the workbench, in the left menu go to *Git*, click *Clone a Repository* and clone the repo https://github.com/mamurak/os-mlops.git
* Navigate to *notebooks/object-detection* and follow the instructions in *notebooks/object-detection-example/demo-setup.ipynb*.

## S3 Bucket setup

Before proceeding, create the *object-detection* bucket in your S3 storage, e.g. in your Minio instance that is referenced in the *object-detection* data connection:

* Call the Minio UI route URL (OCP console: Networking -> Routes -> Project: minio -> minio-ui).
* Log in (use **minio** and **minio123**).
* Create the *object-detection* bucket in the Buckets tab.

## GPU-accelerated inferencing

If you have enabled GPUs in your cluster, configure GPU usage in your Triton model server instance:

* In the RHODS dashboard, navigate to **Data Science Project** *object detection*
* Scroll down to *Models and model servers* and enter the context menu of the **Triton** model server instance (three dots on the right).
* Select **Edit model server**. Increase the count of *Model server GPUs* to **1**.

## Model training
Now create and run the model training as a pipeline with Elyra and Data Science Pipelines. The training pipeline uses images from the *Open Images Dataset V7*. (https://storage.googleapis.com/openimages/web/index.html)

We'll use three classes in our training: *Ball*, *Tennis ball* and *Fedora*. Access the image database website, click **Explore**, search for the classes and have a look at the images. To configure the pipeline, open the file *configuration.yaml* in the Jupyter Notebook and replace the class names with the classes listed above. Hit *Ctrl-s* to save.

* In the Jupyter Notebook navigate to the model-training folder
* Open the appropriate Elyra pipeline, i.e. model-training-gpu.pipeline if you have enabled GPUs in your cluster, else model-training-cpu.pipeline.
* Click **Run Pipeline** to submit the pipeline to the Data Science Pipelines backend.
* Track the pipeline progress in the RHODS dashboard: Data Science Pipelines->Runs->Triggered
* Once the pipeline run has completed, check for the trained model in the *object-detection* bucket in Minio using the **Object Browser**.

## Online scoring

You now have a trained model for object recognition. To use the model we deploy it into RHODS Model Serving:

* Access the RHODS dashboard, in **Data Science Projects** choose *object-detection*
* Under **Models and model servers** click **Deploy model** for the appropriate model server, i.e. Triton if you enabled GPUs in your cluster, else OVMS.
* In the form enter
  * Model Name: demo-model
  * Model framework: ONNX
  * Choose *Existing data connection*: object-detection
  * *model-latest.onnx* as Folder path.
* Select Deploy
* Copy the Inference endpoint URL
* Open the *online-scoring* notebook.
* Paste the inference endpoint URL into the placeholder of the *prediction_url* variable.
* Run the notebook.