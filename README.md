
--------

# 🤔 1. What I wanted to show in this project <a name=learn></a>

* design a batch-serving architecture
* use Hopsworks as a feature store
* design a feature engineering pipeline that reads data from an API
* build a training pipeline with hyper-parameter tunning
* use W&B as an ML Platform to track your experiments, models, and metadata
* implement a batch prediction pipeline
* use Poetry to build your own Python packages
* deploy your own private PyPi server
* orchestrate everything with Airflow
* use the predictions to code a web app using FastAPI and Streamlit
* use Docker to containerize your code
* use Great Expectations to ensure data validation and integrity
* monitor the performance of the predictions over time
* deploy everything to GCP
* build a CI/CD pipeline using GitHub Actions





# 💵 2. Costs <a name=costs></a>

To reproduce the project will cost you  about 25$ run end-to-end on Google Cloud Pltaform

# ❔ 3. Ask Questions <a name=ask-questions></a>

If you have any questions or issues , please create an issue.  I will do my best to respond.

Also, you can contact me directly on [gmail](junioralexio607@gmail.com).

-------

# 📊 6. Data <a name=data></a>
I used an open API that provides hourly energy consumption values for all the energy consumer types within Denmark.

They provide an intuitive interface where you can easily query and visualize the data. You can access the data [here](https://www.energidataservice.dk/tso-electricity/ConsumptionDE35Hour).

The data has 4 main attributes:
* **Hour UTC**: the UTC datetime when the data point was observed. 
* **Price Area**: Denmark is divided into two price areas: DK1 and DK2 - divided by the Great Belt. DK1 is west of the Great Belt, and DK2 is east of the Great Belt.
* **Consumer Type**: The consumer type is the Industry Code DE35, owned and maintained by Danish Energy.
* **Total Consumption**: Total electricity consumption in kWh

**Note:** The observations have a lag of 15 days! But for the project  use case, that is not a problem, as  the same steps can b simulated as it would be in real time.

### IMPORTANT OBSERVATION

The API will become obsolete during 2023. Its latest data points are from June 2023, and the API will become unavailable during 2023. We created a copy of the data from 2020-07-01 and 2023-06-30 to bypass this issue. Thus, there are 3 years of data to play with. More than enough for the purpose of this course. The file is stored in Google Drive accessible [at this link](https://drive.google.com/file/d/1y48YeDymLurOTUO-GeFOUXVNc9MCApG5/view?usp=drive_link).

Thus, instead of querying the API, there is need to   mock the same behavior by loading the data from the file. Therefore, you don't have to download the file yourself. The code will download it and load the data from the file instead of the API, simulating 100% the same behavior.

**---> All Rights Reserved to: www.energidataservice.dk**


<p align="center">
  <img src="images/forecasting_demo_screenshot.png">
</p>

The data points have an hourly resolution. For example: "2023–04–15 21:00Z", "2023–04–15 20:00Z", "2023–04–15 19:00Z", etc.

We will model the data as multiple time series. Each unique price area and consumer type tuple represents its unique time series. 

Thus,the model  built will  independently forecasts the energy consumption for the next 24 hours for every time series.



----------

# 🧬 6. Code Structure <a name=structure></a>

The code is split into two main components: the `pipeline` and the `web app`.

The **pipeline** consists of 3 modules:
- `feature-pipeline`
- `training-pipeline`
- `batch-prediction-pipeline`

The **web app** consists of other 3 modules:
- `app-api`
- `app-frontend`
- `app-monitoring`

**Also,** we have the following folders:
- `airflow` : Airflow files | Orchestration
- `.github` : GitHub Actions files | CI/CD
- `deploy` : Build & Deploy
<br/>
<br/>

To follow the structure in its natural flow, read the folders in the following order:
1. `feature-pipeline`
2. `training-pipeline`
3. `batch-prediction-pipeline`
4. `airflow`
5. `app-api`
6. `app-frontend` & `app-monitoring`
7. `.github`


<br/>
<br/>

-------

# 🔧 7. Set Up Additional Tools <a name=tools></a>

**The code is tested only on Ubuntu 20.04 and 22.04 using Python 3.9.**

 A `.env` file  was used  to store all our credentials. Every module that needs a `.env` file has a `.env.default` in the module's main directory that acts as a template. Thus, you have to run:
```shell
cp .env.default .env
```


## Poetry
##### ``<< free usage >>``

**Note:** In the project used `Poetry 1.4.2`  was used. To avoid potential issues when installing the dependencies using Poetry, It is  recommended to  use the same version (or if there are any errors & you have a different version, you can delete and regenerate the `poetry.lock` file).

Install Python system dependencies:
```shell
sudo apt-get install -y python3-distutils
```
Download and install Poetry:
```shell
curl -sSL https://install.python-poetry.org | python3 -
```
Open the `.bashrc` file to add the Poetry PATH: 
```shell
nano ~/.bashrc
```

Add `export PATH=~/.local/bin:$PATH`

to `~/.bashrc`

Check if Poetry is installed:
```shell
source ~/.bashrc
poetry --version
```

[If necessary, here are the official Poetry installation instructions.](https://python-poetry.org/docs/#installation)


#### macOS M1/M2 Poetry Issues
**!!!** If you have issues creating Poetry environments on macOS M1/M2 devices, [Hongnan Gao](https://github.com/gao-hongnan) implemented a script that will solve all the dependency issues. Just run the following before creating a Poetry environment:
```shell
bash scripts/install_poetry_macos_m1_chip.sh
```


## Docker
##### ``<< free usage >>``

In the project `Docker version 24.0.5`  was used.

* [Install Docker on Ubuntu.](https://docs.docker.com/engine/install/ubuntu/)
* [Install Docker on Mac.](https://docs.docker.com/desktop/install/mac-install/)
* [Install Docker on Windows.](https://docs.docker.com/desktop/install/windows-install/)

## Configure Credentials for the Private PyPi Server
##### ``<< free usage >>``

**<br/>The private PyPi server   was run using Docker down the line. But it will already expect the credentials configured.<br/>**

Create credentials using `passlib`:
```shell
# Install dependencies.
sudo apt install -y apache2-utils
pip install passlib

# Create the credentials under the energy-forecasting name.
mkdir ~/.htpasswd
htpasswd -sc ~/.htpasswd/htpasswd.txt energy-forecasting
```

Set `poetry` to use the credentials:
```shell
poetry config repositories.my-pypi http://localhost
poetry config http-basic.my-pypi energy-forecasting <password>
```

Check that the credentials are set correctly in your poetry `auth.toml` file:
```shell
cat ~/.config/pypoetry/auth.toml
```

## Hopsworks 
##### ``<< free usage >>``

[Hopsworks](https://www.hopsworks.ai/)  was used  as the serverless feature store. Thus, you can   create an account and a project on Hopsworks. 

 But long story short, you can go to your Hopsworks account settings and get the API Key from there. Afterward, you must create a new project (or use the default one) and add these credentials to the `.env` file under the `FS_` prefix.

**!!!** Be careful to name your project differently than **energy_consumption,** as Hopsworks requires unique names across its serverless deployment.

[Click here to start with Hopsworks](https://www.hopsworks.ai/).

**Note:**In the project  only the Hopsworks freemium plan  was used, making it free of charge to replicate the code within the series. 


## Weights & Biases
##### ``<< free usage >>``

In the project   Weights & Biases   were used as the serverless ML platform. Thus, you must create an account and a project on Weights & Biases.
 But long story short, you can go to your W&B and create an entity & project. Afterward, you must navigate to user settings and create the API Key from there. In the end, you must add these credentials to the `.env` file under the `WANDB_` prefix.



[Click here to start with Weights & Biases](https://wandb.ai/).

**Note:**In the project   only the W&B freemium plan will used, making it free of charge to replicate the code within the series. 

## GCP

First, you must install the `gcloud` GCP CLI on your machine.

[Follow this tutorial to install it.](https://cloud.google.com/sdk/docs/install)

**If you only want to run the code locally, go straight to the "Storage" section.**<br/>

As before, you have to create an account and a project on GCP. Using solely the bucket as storage will be free of charge.



### Storage
##### ``<< free usage >>``

At this step, you have to do 5 things:
- create a project
- create a non-public bucket
- create a service account that has admin permissions to the newly created bucket
- create a service account that has read-only permissions to the newly created bucket
- download a JSON key for the newly created service accounts.

Your `bucket admin service account` should have assigned the following role: `Storage Object Admin`<br/>
Your `bucket read-only service account` should have assigned the following role: `Storage Object Viewer`<br/>

![Bucket Creation](images/gcp_gcs_screenshot.png)

* [Docs for creating a bucket on GCP.](https://cloud.google.com/storage/docs/creating-buckets)<br/>
* [Docs for creating a service account on GCP.](https://cloud.google.com/iam/docs/service-accounts-create)<br/>
* [Docs for creating a JSON key for a GCP service account.](https://cloud.google.com/iam/docs/keys-create-delete)<br/>



**NOTE:** Don't forget to add the GCP credentials to the `.env` file under the `GOOGLE_CLOUD_` prefix:
* *GOOGLE_CLOUD_PROJECT*: your project name (e.g., "energy_consumption")
* *GOOGLE_CLOUD_BUCKET_NAME*: your bucket name (e.g., "hourly-batch-predictions")
* *GOOGLE_CLOUD_SERVICE_ACCOUNT_JSON_PATH*: absolute path to your JSON key file. (e.g., "/absolute/path/to/your/service-account.json")


### Deployment
##### ``<< ~20$ >>``

This step must only be finished if you want to deploy the code on GCP VMs and build the CI/CD with GitHub Actions.

Note that this step might result in a few costs on GCP. It won't be much. While developing this course, we spent only ~20$.

Also, you can get some free credits if you create a new GCP account (I created a new account and received 300$ in GCP credits). Just be sure to delete the resources after you finish the course.

See [this document](/README_DEPLOY.md) for detailed instructions.

-------

# 🔎 8. Usage <a name=usage></a>

**The code is fully tested on Ubuntu 20.04 & 22.04 using Python 3.9 and Poetry 1.4.2.**

**Note:** If you are working on macOS M1/M2, be sure to check the [macOS M1/M2 Poetry Issues](https://github.com/iusztinpaul/energy-forecasting/tree/main#macos-m1m2-poetry-issues) section.

## The Pipeline

#### Run 
You will run the pipeline using Airflow (`free usage`). Don't be scared. Docker makes everything very simple to set up.

**Note:**also hook the **private PyPi server** in the same docker-compose.yaml file with Airflow. Thus, everything will start with one command.


Run:
```shell
# Move to the airflow directory.
cd airflow

# Make expected directories and environment variables
mkdir -p ./logs ./plugins
sudo chmod 777 ./logs ./plugins

# It will be used by Airflow to identify your user.
echo -e "AIRFLOW_UID=$(id -u)" > .env
# This shows where our project root directory is located.
echo "ML_PIPELINE_ROOT_DIR=/opt/airflow/dags" >> .env
```

Now from the `airflow` directory move to the `dags` directory and run:
```shell
cd ./dags

# Make a copy of the env default file.
cp .env.default .env
# Open the .env file and complete the FS_API_KEY, FS_PROJECT_NAME and WANDB_API_KEY credentials 

# Create the folder where the program expects its GCP credentials.
mkdir -p credentials/gcp/energy_consumption
# Copy the GCP service credetials that gives you admin access to GCS. 
cp -r /path/to/admin/gcs/credentials/admin-buckets.json credentials/gcp/energy_consumption
# NOTE that if you want everything to work outside the box your JSON file should be called admin-buckets.json.
# Otherwise, you have to manually configure the GOOGLE_CLOUD_SERVICE_ACCOUNT_JSON_PATH variable from the .env file. 
```

Now go back to the `airflow` directory and run the following:
```shell
cd ..

# Initialize the Airflow database
docker compose up airflow-init

# Start up all services
# Note: You should set up the private PyPi server credentials before running this command.
docker compose --env-file .env up --build -d
```


Wait a while for the containers to build and run. After access `127.0.0.1:8080` to login into Airflow.<br/>
Use the following default credentials to log in:
* username: `airflow`
* password: `airflow`

<p align="center">
  <img src="images/airflow_login_screenshot.png">
</p>

Before starting the pipeline DAG, you must deploy the modules to the private PyPi server. Go back to the `root folder` of the `energy-forecasting` repository and run the following to build and deploy the pipeline modules to your private PyPi server:
```shell
# Set the experimental installer of Poetry to False. For us, it crashed when it was on True.
poetry config experimental.new-installer false
# Build & deploy the pipelines modules.
sh deploy/ml-pipeline.sh
```
Airflow will know how to install the packages from the private PyPi server. <br/>

One final step is to configure the parameters used to run the pipeline. Go to the `Admin` tab, then hit `Variables.` There you can click on the `blue` `+` button to add a new variable.
These are the three parameters you can configure with our suggested values:
* `ml_pipeline_days_export = 30`
* `ml_pipeline_feature_group_version = 5`
* `ml_pipeline_should_run_hyperparameter_tuning = False`
<br/>

<p align="center">
  <img src="images/airflow_variables_screenshot.png">
</p>

Now, go to the `DAGS/All` section and search for the `ml_pipeline` DAG. Toggle the activation button. It should automatically start in a few seconds. Also, you can manually run it by hitting the play button from the top-right side of the `ml_pipeline` window.

<p align="center">
  <img src="images/airflow_ml_pipeline_dag_overview_screenshot.png">
</p>

That is it. You can run the entire pipeline with a single button if all the credentials are set up correctly. How cool is that?

Here is what the DAG should look like 👇

<p align="center">
  <img src="images/airflow_ml_pipeline_dag_screenshot.png">
</p>


#### Clean Up
```shell
docker compose down --volumes --rmi all
```

#### Backfil Using Airflow

Find your `airflow-webserver` docker container ID:
```shell
docker ps
```
Start a shell inside the `airflow-webserver` container and run `airflow dags backfill` as follows (in this example, we did a backfill between `2023/04/11 00:00:00` and `2023/04/13 23:59:59`):
```shell
docker exec -it <container-id-of-airflow-webserver> sh
airflow dags backfill --start-date "2023/04/11 00:00:00" --end-date "2023/04/13 23:59:59" ml_pipeline
```
If you want to clear the tasks and run them again, run these commands:
```shell
docker exec -it <container-id-of-airflow-webserver> sh
airflow tasks clear --start-date "2023/04/11 00:00:00" --end-date "2023/04/13 23:59:59" ml_pipeline
```


### Run Private PyPi Server Separately

The private PyPi server is already hooked to the airflow docker compose file. But if you want to run it separately for whatever reason, you can run this command instead:
```shell
docker run -p 80:8080 -v ~/.htpasswd:/data/.htpasswd pypiserver/pypiserver:v1.5.2 run -P .htpasswd/htpasswd.txt --overwrite
```

------

## The Web App



Copy the bucket read-only GCP credentials to the root directory of your `energy-forecasting` project:
```shell
# Create the folder where the program expects its GCP credentials.
mkdir -p credentials/gcp/energy_consumption
# Copy the GCP service credetials that gives you read-only access to GCS. 
cp -r /path/to/admin/gcs/credentials/read-buckets.json credentials/gcp/energy_consumption
# NOTE that if you want everything to work outside the box your JSON file should be called read-buckets.json.
# Otherwise, you have to manually configure the APP_API_GCP_SERVICE_ACCOUNT_JSON_PATH variable from the .env file of the API.
```

Go to the API folder and make a copy of the `.env.default` file:
```shell
cd ./app-api
cp .env.default .env
```

**NOTE:** Remember to complete the `.env` file with your own variables. 

That is it!

Go back to the root directory of your `energy-forecasting` project and run the following docker command, which will build and run all the docker containers of the web app:
```shell
docker compose -f deploy/app-docker-compose.yml --project-directory . up --build
```

If you want to run it in development mode, run the following command:
```shell
docker compose -f deploy/app-docker-compose.yml -f deploy/app-docker-compose.local.yml --project-directory . up --build
```

**Now you can see the apps running at:**
* [API](http://127.0.0.1:8001/api/v1/docs)
* [Frontend](http://127.0.0.1:8501/)
* [Monitoring](http://127.0.0.1:8502/)

-----

## Deploy the Code to GCP

[Check out this section.](./README_DEPLOY.md)

## Set UP CI/CD with GitHub Actions

[Check out this section.](./README_CICD.md)


------


# 🧑‍💻 9. Installation & Usage for Development <a name=installation></a>

All the modules support Poetry. Thus the installation is straightforward.

**Note 1:** Just ensure you have installed Python 3.9, not Python 3.8 or Python 3.10.

**Note 2:** In the project, we used `Poetry 1.4.2` was used. To avoid potential issues when installing the dependencies using Poetry,  use the same version (or if there are any errors & you have a different version, you can delete and regenerate the `poetry.lock` file).



## The Pipeline

**We support Docker to run the whole pipeline. Check out the [Usage](#usage) section if you only want to run it as a whole.**<br/>

If Poetry is not using Python 3.9, you can follow the next steps:
1. Install Python 3.9 on your machine.
2. `cd /path/to/project`, for example, `cd ./feature-pipeline`
3. run `which python3.9` to find where Python3.9 is located
4. run `poetry env use /path/to/python3.9`




**See here how to install every project individually:**
- [Feature Pipeline](/feature-pipeline/README.md)
- [Training Pipeline](/training-pipeline/README.md)
- [Batch Prediction Pipeline](/batch-prediction-pipeline/README.md)
<br/>

### Set Up the ML_PIPELINE_ROOT_DIR Variable

**Important:** Before installing and running every module individually, **one key step** is to set the `ML_PIPELINE_ROOT_DIR` variable to your root directory of the `energy-forecasting` project (or any other directory - just make sure to set it):
Export it to your `~/.bashrc` file:
```shell
gedit ~/.bashrc
export ML_PIPELINE_ROOT_DIR=/path/to/root/directory/repository/energy-forecasting/
```
Or run every Python script proceeded by the `ML_PIPELINE_ROOT_DIR` variables. For example:
```shell
ML_PIPELINE_ROOT_DIR=/path/to/root/directory/repository/energy-forecasting/ python -m feature_pipeline.pipeline
```

By doing so, all the 3 pipeline projects (feature, training, batch) will load and save the following files from the same location:
* `.env` configuration;
* JSON metadata files;
* logs & plots.

**NOTE:** This step is **critical** as every pipeline component needs to access the JSON metadata from other pipeline processes. By setting up the **ML_PIPELINE_ROOT_DIR** variable, all the metadata JSON files will be saved and accessed from the same location between different processes. For example, the batch prediction pipeline will read the model version it needs to use to make predictions from a JSON file generated by the training pipeline. Without settings the **ML_PIPELINE_ROOT_DIR**, the training and batch processes won't share the same output directory. Thus, they won't know how to talk to each other. When running the project inside `Airflow`, it is defaulted to `/opt/airflow/dags`; thus, you must set this variable only when running it outside Airflow.  
<br/> 

## The Web App
**We support Docker to run the web app. Check out the [Usage](#usage) section if you only want to run it as a whole.**<br/>

**See here how to install every project individually:**
- [API](/app-api/README.md)
- [Frontend](/app-frontend/README.md)
- [Monitoring](/app-monitoring/README.md)

You can also run the whole web app in development mode using Docker:
```shell
docker compose -f deploy/app-docker-compose.yml -f deploy/app-docker-compose.local.yml --project-directory . up --build
```

------

# 🏆 10. Licensing & Contributing <a name=licensing></a>

The code is under the MIT License. Thus, as long as you keep distributing the License, feel free to share, clone, or change the code as you like.

Also, if you find any bugs or missing pieces in the documentation, I encourage you to add an issue on GitHub or a PR. Based on your support, I will adapt the code and docs for future use Furthermore, you can contact me directly  if you have any questions.



-----

### Let's connect if you want to level up in designing and productionizing ML systems:

<br/>
<br/>
<br/>



-----

<br/>Thank you ✌🏼 !<br/>

