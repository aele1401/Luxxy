# Luxxy

## Description
This project provides an automated solution for deploying a COVID health result testing system using infrastructure as code to manage, migrate, and provision resources from multiple cloud providers. It facilitates scalable and resilient processing and storage of health data using infrastructure as code to create and provision S3 buckets, SQL cloud instances, clusters, and kubernetes to deploy the infrastructure.

## Features
- Automated multi-cloud deployment
- Scalable architecture
- Secure handling of health data
- Integration with multiple cloud providers (AWS & Google Cloud)

## Architecture
![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/solution_architecture.png)

## Getting Started

### Prerequisites:
- Cloud provider accounts (AWS & Google Cloud)
- Terraform
- Kubernetes
- Cloud Shell
- Mission Files

## Usage & Deployment

- Mission #1 - Setup and configuration of cloud instances
- Mission #2 - IAM and cluster deployment
- Mission #3 - Data migration

## Mission #1 - Getting Started

- Access AWS console and create a new programmatic user *"terraform-en-1"* 
    * Set permissions
    * Review details
    * Create new user

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/users.png)

- Obtain and download access key for created user and mission1 files
- **GCP Console:**
    * Upload access keys and mission 1 files into GCP Cloud Shell
    * Set all applicable files in mission1 directory as executables
        - `chmod +x *.sh`
    * AWS and GCP environment preparation:
        - Run `./aws_set_credentials.sh accessKeys.csv`
        - Run `gcloud config set project <project_id>`
        - Execute `./gcp_set_project.sh`
    * Enable container registry API, Kubernetes Engine API, and Cloud SQL API by running:
        - `gcloud services enable containerregistry googleapis.com` 
        - `gcloud services enable container.googleapis.com` 
        - `gcloud services enable sqladmin.googleapis.com` 
    * Update *"tcb_aws_storage.tf"* file to contain unique bucket name required for S3
    * Run the commands to finalize provisioning:
        - `cd ~/mission1_en/mission1/en/terraform/`
        - `terraform init`
        - `terraform plan`
        - `terraform apply`
            * `Type Yes and go ahead`
        
    * After instance has been created, go to *"Cloud SQL Instance"*
        - Configure SQL network
        - Navigate to SQL Instance Cluster
        - In the left menu, select *"Connections"*
        - Under *"Connections,"* select:
            * *"Private IP"* 
            * *"Network: default"*
            * In *"Authorized networks,"* name will be *"Public Access (Testing Only)"*
            * *"Network: 0.0.0.0/0"*
               - *Note: This configuration is set to allow anybody to connect through the web. In a real-word environment, this is not recommended. In production environments, it is strongly recommended to only use private network for database access.*

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/creation_complete.png)

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/bucket.png)


## Mission #2: IAM & Cluster Deployment

- **AWS:**
    * Access AWS console and go to *"IAM service"*
    * In *"Access Management,"* create a new programmatic user named *" luxxy-covid-testing-system-en-app1"*
        - Set permissions for user
        - Review details
        - Create new user
        - Go to *"Access Keys"* section
        - Download access keys

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/users.png)

- **GCP:**
    * Navigate to *"Cloud SQL"* instance and create new user
    * Connect to Google Cloud Shell
    * Download and unzip Mission2 files
    * Connect to DB running on Cloud SQL using user and password just created
        - `mysql --host=<public_ip_cloudsql> --port=3306 -u app -p`
    * After logged into DB, create products table for testing purposes
        - `use dbcovidtesting;`
        - `source ~/mission2_en/mission2/en/db/create_table.sql;`
        - `show tables;`
        - `exit;`
    ![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/sql_instance.png)
    * Enable Cloud Build API via Cloud Shell
        - `gcloud services enable cloudbuild.googleapis.com`
    * Build Docker image and push it to Google Container Registry. Replace *PROJECT ID*.
        - `cd ~/mission2_en/mission2/en/app`
        - `gcloud builds submit --tag gcr.io <PROJECT_ID>/luxxy-covid-testing-system-app-en`
    * Open cloud editor and update *"luxxy-covid-testing-system.yaml"* file under *"kubernetes"* directory to contain bucket name, access key, secret key, and DB host name *(private IP of SQL instance)*
    * Connect to GKE cluster shell and deploy infrastructure
        - `kubectl apply -f luxxy-covid-testing-system.yaml`

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/clusters.png)

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/workloads.png)

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/service_ingress.png)

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/luxxy_system.png)

## Mission #3: Data Migration

### GCP Database Migration
- Connect to **Google Cloud Shell**
- Download dump
    * `wget https://tcb-public-events.s3.amazonaws.com/icp/mission3.zip`
- Connect to **Cloud SQL database instance**
    * `mysql --host=<public_ip_address> --port=3306 -u app -p`
- Import dump on Cloud SQL
    * `use dbcovidtesting;`
    * `source ~/mission3_en/mission3/en/db/db_dump.sql`
- Verify data was imported correctly
    * `select * from records;`

### AWS - PDF File Migration

- Connect to **AWS Cloud Shell**
- Download PDF files
    * `mkdir mission3_en`
    * `cd mission3_en`
    * `wget https://tcb-public-events.s3.amazonaws.com/icp/mission3.zip`
    * `unzip mission3.zip`
- Synch PDF files with S3 COVID-19 Testing Status System and test application
    * `cd mission3/en/pdf_files`
    * `aws s3 sync . s3://luxxy-covid-testing-system-pdf-en-xxxx`

### Successful Deployment of Luxxy COVID-19 Testing System Result

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/luxxy_system.png)

![Diagram](https://github.com/aele1401/Luxxy/blob/main/Images/get_results.png)

After application testing is complete and successful deployment of system, empty S3 buckets and use terraform destroy to terminate managed resources.

### Deleting Multicloud Resources

- **Access AWS console and S3 service**
- Select and empty bucket
   * Confirm emptying contents in bucket
- Open *"tcb_gcp_database.tf"* file under *"mission1"* directory using Google Editor
    * Update file on line 11 setting database deletion protection to false
- Run
    * `cd ~/mission1_en/mission1/en/terraform/`
    * `terraform apply`
        - *`yes`*
    * `terraform destroy`
        - *`yes`*
   
 # 
 ## *Special thanks to Jean Rodrigues and The Cloud Bootcamp*
    



