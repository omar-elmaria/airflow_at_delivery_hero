# airflow_at_delivery_hero
This repo contains a readme.md file that explains how to use Airflow at Delivery Hero

# Access Permissions
To start using Airflow at Delivery Hero, you need to obtain 'Edit' access to:
1. [datahub-airflow](https://github.com/deliveryhero/datahub-airflow) repo
    - A pre-requisite to this step is installing [Git](https://git-scm.com/), creating an GitHub account, and merging your account with your Delivery Hero account through [sailpoint](https://iam.deliveryhero.net/identity/workitem/commonWorkItem.jsf#/commonWorkItem/session)
    - To get access to the [datahub-airflow](https://github.com/deliveryhero/datahub-airflow) repo, you need to open a ticket on the datahub Jira domain similar to this one --> [DHUB-1301](https://jira.deliveryhero.com/browse/DHUB-1301)
    - The **Global Pricing Team's** user group (pricing@deliveryhero.com) already has "Admin" access to the datahub-airflow repo 

2. The **Airflow instance** that belongs to your business unit (e.g., [marketing](https://airflow-mkt.datahub.deliveryhero.net/home), [logistics](https://airflow-log.datahub.deliveryhero.net/home), etc.)
    - To get access to your business unit's Airflow instance, you need to submit a request to **designated person** or in the respective **data engineering channel**.
    - **Logistics** --> [#log-data-engineering](https://deliveryhero.slack.com/archives/CCTGZJAM8). Use the **"Shortcut"** function to submit a support request similar to the one shown below.
    ![image](https://user-images.githubusercontent.com/98691360/196386129-332db5ab-2954-4405-97e4-1535b2bda58c.png)
    - **Marketing** --> #mkt-data-eng
      - Keep in mind that this channel is private, so you will need to be invited to it. Contact @Sarmad Ahmed on Slack to get added
      - If this doesn't work, you can contact @Ersin Ihsan Ünkar or one of his direct reports

# Cloning the datahub-airflow repo
You need to clone the datahub-airflow repo to be able to commit to it. Follow the instructions below to interact with the repo correctly.

## Mac
Before cloning the repo on a Mac machine, you need to add a **SSH key**. Follow the instructions in this [video](https://drive.google.com/drive/u/0/folders/1P2Vbo5OBNDSDOFmS8-SjKjZ709boiJ7s) from **5:20** to **8:55**. After that follow **step 1** in the **"Windows"** section.

## Windows
1. **Clone the repo** by entering this command in your terminal ```git clone https://github.com/deliveryhero/datahub-airflow.git```
    - If you are cloning the repo to a location on **G-drive**, you will have to delete the **.ini** files for the cloning to work. The best way to do that is through a **.bat** file that you run in your **terminal** or by **double clicking it**
      - **Create a .bat file** anywhere on your G-drive and place the following command in it
      - ```del /s /q /f /a "{path_to_folder_containing_ini_files}\desktop.ini"```
2. ```cd [REPOSITORY]```
3. Enter the following **three commands** in your terminal
      - ```git config core.protectNTFS false``` --> Allows **format overrides**
      - ```git config core.sparsecheckout true``` --> Accept paths with **empty trailing or preceding spaces**. This is done automatically in **Mac**
      - ```git config --system core.longpaths true``` --> Accept paths **longer than 256 characters**
      - If you **face errors with Git commands due to long paths**, you might need to do two additional steps to prevent long path errors: 
        - **Step 1:**
          - Search for **Registry editor** in the Windows search box
          - Go to this path ```Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem```
          - Set the LongPath option to 1
        - **Step 2:**
          - Search for **gpedit.msc**, then press the Enter key. This launches the Local Group Policy Editor.
          - Navigate to ```Local Computer Policy > Computer Configuration > Administrative Templates > System > Filesystem```
          - Double click **Enable NTFS long paths**
      
# Creating a DAG
The best practice is to create a folder containing all **SQL** and **Python** scripts under your respective business unit. For logisitcs, use the [log](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/log) folder. For marketing, use the [mkt](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/mkt) folder. To see a good example of a folder structure, you can check out the "Loved Brands" [project](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/log/loved_brands_automation)

The **Datahub engineeering team** created many useful features that could help you spin up your DAG **without** the need for creating custom functions and operators. You can check a couple of DAG examples below:
- [Rider contact daily rate](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/log/rider_contact_rate_daily.py)
- [Loved Brands automation](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/log/loved_brands_automation/loved_brands_full_dag.py)

You can also check the [SQL files](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/log/loved_brands_automation/sql_queries) under the "Loved Brands" project to see how to **parametrize** your project IDs and datasets so that the correct one is picked based on the requirements of the task.

Keep in mind that you will need to install the Airflow Python package to utilize the commonly used Airflow operators such as the ```BigQueryExecuteQueryOperator``` and ```PythonOperator```. To do that, please run the following command in your terminal after creating and activating a Py virtual environment
- ```pip3 install "apache-airflow[celery]==2.3.4" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.3.4/constraints-3.7.txt"```

## Configuring your code to run on **Staging** and **Production** environments
The **Global Pricing** team currently shares the staging environment ([Stg-ds-pricing-demand](https://airflow-log-stg-ds-pricing-demand.datahub.deliveryhero.net/)) with the **Data Science** team. Please don't use any other environment. Parameterizing your code to run on **Staging** and **Production** is done via **YAML config files**. 

There is one base YAML file in the datahub-airflow repo called ```config.yaml```. It is found in this [here](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/log/configuration/yaml/config.yaml). This file contains the standard configurations that are used by default. ```config_production.yaml``` **overrides** these default configuration when the DAG runs on production. That way, you will **not** need to create two Python scripts for the DAG. You can only create one file and import the configurations from "config.yaml" using the ```get``` method as shown below.

```
from configuration import config
airflow_gcp_conn_id = config.get("global-pricing").get("bigquery_conn_id")
```
For the Global Pricing team's use cases, these are the parameters used in ```config.yaml```. They are configured based on the staging **environment** ([Stg-ds-pricing-demand](https://airflow-log-stg-ds-pricing-demand.datahub.deliveryhero.net/)) and staging **project** we use to dump the output tables ```log-data-science-staging```

```
global-pricing:
  project_id_read: "fulfillment-dwh-staging" # The GCP project used in queries for reading/pulling data
  project_id_destination: "log-data-science-staging" # The GCP project used in queries to store output tables
  bigquery_conn_id: "bigquery_data_science_flat" # The Airflow connection used to connect to BQ
  dataset_read: "cl" # The dataset used in queries for reading/pulling data
  dataset_staging: "staging" # The dataset to store the tables of intermediate steps
  dataset_destination: "global_pricing" # The dataset to store the table produced by the last task
```
The configurations above get overriden by the ones on ```config_production.yaml```
```
global-pricing:
  project_id_read: "fulfillment-dwh-production" # When the DAG runs on production, project_id_read becomes fulfillment-dwh-production 
  project_id_destination: "dh-logistics-product-ops" # When the DAG runs on production, project_id_destination becomes dh-logistics-product-ops 
  bigquery_conn_id: "bigquery_logistics_flat" # When the DAG runs on production, bigquery_conn_id becomes bigquery_logistics_flat
  dataset_destination: "pricing" # When the DAG runs on production, dataset_destination becomes pricing
```
If you have a configuration that does not change when switching environments, you do **not** need to specify it in the ```config_production.yaml``` file. To import these configurations to your DAG, use the ```get``` method after the instantiating your DAG as follows:
```
from configuration import config
with DAG(
    dag_id="loved-brands",
    schedule_interval="0 4 3 * *",  # At 04:00 on day-of-month 3
    template_searchpath=template_search_path, # See the full DAG (linked below) to see how we define this variable
    default_args=default_args, # See the full DAG (linked below) to see how we define this dictionary
    catchup=False,
    tags=["pricing", "loved_brands"],
) as dag:

    # Task parameters
    airflow_gcp_conn_id = config.get("global-pricing").get("bigquery_conn_id")
    project_id_read = config.get("global-pricing").get("project_id_read")
    # ... and so on
```
You can check the Loved Brands [DAG](https://github.com/deliveryhero/datahub-airflow/blob/main/dags/log/loved_brands_automation/loved_brands_full_dag.py) to see how the rest of the configurations are imported. The process is identical to what is shown above.

## Linting
After you place all your SQL and Python files in the right place and create your DAG, you will need to run some commands in the terminal so that the scripts **pass the automatic checks** that are run after you push your code to GitHub.

- In your terminal, run the following command after activating the virtual environment of your project
  - ```pip install sqlfluff, black, flake8, isort```
- For **SQL** files you will need to run this command in your terminal --> ```sqlfluff fix /local_path/to/sql_queries```. Replacing ```fix``` with ```lint``` will **display** the fixable formatting errors **without fixing** them
- For **Python** files, run the following commands in the specified order:
  - black /local_path/to/py_files
  - flake8 /local_path/to/py_files
  - isort /local_path/to/py_files

## Pushing code to GitHub
You cannot commit to the **main** branch right away. You need to **open a new branch** and create a **pull request**. This YouTube [video](https://www.youtube.com/watch?v=rgbCcBNZcdQ) explains how to create a pull request in under 3 minutes.

As a friendly reminder, the commands that you will need are as follows:
- ```git checkout -b [BRANCH_NAME]``` --> This **creates** and **switches** the HEAD to a new branch
- ```git add .```
- ```git commit -m "YOUR MESSAGE AFTER CHANGES HERE"```
- ```git push origin [BRANCH_NAME]```
- Open GitHub and click on ```Compare & Pull Request``` and follow the instructions to create your pull request

If you have been working on your branch for a while and some changes occured on the **main** branch that create conflicts with your code, you will need to **pull the changes** and merge the **base branch into the head branch**
- **Step 1:** Clone the repository or update your local repository with the latest changes --> ```git pull origin main```
- **Step 2:** Switch to the head branch of the pull request --> ```git checkout [BRANCH_NAME]```
- **Step 3:** Merge the base branch into the head branch --> ```git merge main```
- **Step 4:** Fix the conflicts manually (if any exist) and commit the result --> ```git commit -m "..."```
- **Step 5: Push the changes** --> ```git push -u origin [BRANCH_NAME]```

After the code is pushed to GitHub, some checks will run in the background and a docker image of your repo will be created. If there any **formatting** or **syntax** errors, you will need to **fix them** and **re-commit**. If **all the checks are successful**, you are ready to deploy the GitHub branch to the staging environment. If anything fails, check the failures before deploying to staging.

## Deploying GitHub Branches on Staging
- Depending on your business unit, you will need to open the right Airflow environment
  - [Logistics](https://airflow-log.datahub.deliveryhero.net/home). This works for the staging environment of **Data Science** (Stg-ds-pricing-demand) and Logistics Customer Analytics (Stg-bi-customer)
  - [Marketing](https://airflow-mkt.datahub.deliveryhero.net/home)
- Click on ```"Add-ons" > "Deploy airflow branch"```
- Select the right **environment** --> Currently, the one the Global Pricing team uses is **Stg-ds-pricing-demand**, although this might change in the future as this is a shared environment with the **Data Science** team
- Insert your **GitHub branch name**. You can copy it from the Pull Request
![image](https://user-images.githubusercontent.com/98691360/198082728-b13b725a-d497-4220-801c-7ce16a9f4fd9.png)
- Before clicking on **Submit**, go to the [#log-chapter-data](https://deliveryhero.slack.com/archives/C6R95H4TS) slack channel, check that **no one else is using the environment**, and **announce that you will deploy a GitHub branch** via the **"Integration"** functionality
![image](https://user-images.githubusercontent.com/98691360/196409017-abd7d588-e09e-4af5-acf9-90ed948425c9.png)
- Return to the Airflow instance and click on **Submit**
- A link will appear directly after you click **Submit**. **After 2-3 minutes**, your Airflow environment will be ready to trigger
- Before triggering the DAG, you need to promote yourself as an **Environment Admin**

![image](https://user-images.githubusercontent.com/98691360/196414868-45b06048-f64b-4ae5-83db-4d58d60f737d.png)
- After you're done with testing, share with the other channel members that the environment is **free** by adding an emoji to your first announcement message. This relevant to **logistics environments** only)
- If you are using marketing environments, make sure to **delete the environment** after you're done
- If your DAG fails, check the **Airflow logs** and correct the error. You will need to re-commit your changes to GitHub, wait until all the **checks are passed**, and re-deploy the branch to the staging environment

![image](https://user-images.githubusercontent.com/98691360/198083492-d0759028-3b42-446d-adf0-398742b10e68.png)

## Merging Code to Production
- **Logistics** --> Submit **support request** in the [#log-data-engineering](https://deliveryhero.slack.com/archives/CCTGZJAM8) channel to **review** and **approve** your PR. Feel free to replicate this [request](https://deliveryhero.slack.com/archives/CCTGZJAM8/p1666014873416579)
- **Marketing** --> Create a Jira ticket similar to this [one](https://jira.deliveryhero.com/browse/DAFO-2121)

## Slack Alerts
To integrate Slack alerts in your DAG so that you can be notified whenever a DAG fails or succeeds, you will need to do the following:
- Import the ```alerts``` module from ```datahub.common``` --> ```from datahub.common import alerts```
  - This gives you access to two callback functions that you can add to your Airflow tasks via the ```on_success_callback``` and ```on_failure_callback``` parameters
    - ```alerts.setup_on_success_callback()```
    - ```alerts.setup_on_failure_callback()```
  - An example of how to use the functions in an Airflow operator is shown below
  
  ![image](https://user-images.githubusercontent.com/98691360/198099135-d12d34a6-aa5e-4b4a-a9c2-a35c978a7a56.png)
- In the **default_args** of the DAG, define a team who will **own** the DAG as shown below

![image](https://user-images.githubusercontent.com/98691360/196412224-aa78ba09-f356-498b-aede-9e16bdde9459.png)
- In ```datahub-airflow\dags\log\configuration\yaml\config.yaml```, add the **name of the team** and their **Slack IDs**

![image](https://user-images.githubusercontent.com/98691360/196412361-81f4b764-4423-42fe-bc26-75d724e4c2e3.png)
- **Create a new channel** in Slack to send the alerts to and **enter its name** in ```datahub-airflow\dags\log\configuration\yaml\config_production.yaml```

![image](https://user-images.githubusercontent.com/98691360/196412706-8455d8b1-4094-47e5-be45-ff2b4bff7271.png)
- Add the **GDF Monitoring** bot to the channel

![image](https://user-images.githubusercontent.com/98691360/198098893-9a98b871-0473-4b79-add9-e36f07d2e80f.png)

- This channel will be used to send Slack alerts for DAGs running on the **production** environment. For DAGs running on staging, alerts will be automatically sent to [#log-airflow-st](https://deliveryhero.slack.com/archives/CBB1J65HA), albeit **without mentions**
