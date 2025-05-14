# Nash DataOps Pipeline

This repository contains the ETL (Extract, Transform, Load) pipeline code for processing NYC For-Hire Vehicle (FHV) trip data using AWS Glue, S3, and Redshift.

## Repository Structure

- **job-scripts/fhvhv_etl_job.py**: AWS Glue job script for transforming raw FHV trip data
- **job-scripts/fhvhv_redshift_load_job.py**: AWS Glue job script for loading processed data into Redshift
- **job-scripts/fhvhv_redshift_schema_from_catalog_job.py**: AWS Glue job script for setting up Redshift schema based on Glue Data Catalog
- **.github/workflows/s3-deploy.yml**: GitHub Actions workflow for CI/CD deployment to S3

## Pipeline Components

### 1. S3 to S3 ETL (fhvhv_etl_job.py)

This job processes the raw taxi data:
- Reads Parquet files from S3 using Glue Data Catalog
- Joins with taxi zone lookup data to add geographic context
- Derives time-based dimensions (year, month, day, hour)
- Adds data quality transformations and handles null values
- Calculates trip metrics like trip duration
- Writes processed data back to S3, partitioned for efficient querying

### 2. S3 to Redshift ETL (fhvhv_redshift_load_job.py)

This job loads processed data into Redshift:
- Reads processed CSV data from S3 via Glue Data Catalog
- Adds load timestamp for data lineage tracking
- Loads data into Redshift tables using JDBC connection
- Sets up Redshift schemas if they don't exist

### 3. Redshift Schema Management (fhvhv_redshift_schema_from_catalog_job.py)

This job manages Redshift database structure:
- Reads schema from Glue Data Catalog
- Creates or updates Redshift tables to match the catalog schema
- Uses idempotent operations for safe execution

## CI/CD with GitHub Actions

This repository includes a GitHub Actions workflow that automatically deploys ETL job scripts to an S3 bucket whenever changes are pushed to the main branch:

### Workflow Features

- **Automatic Deployment**: All Python scripts are deployed to S3 when changes are pushed to main branch
- **Environment Selection**: Manual workflow runs allow selecting dev or prod environment
- **Testing**: Runs tests before deployment (when tests are added)
- **AWS Integration**: Updates AWS Glue ETL jobs with new script locations (optional)
- **Packaging**: Creates a zip archive containing all ETL scripts

### Setup Instructions

To enable the GitHub Actions workflow, you need to configure the following:

1. **Add AWS Credentials**:
   - Go to your GitHub repository settings
   - Navigate to "Secrets and variables" > "Actions"
   - Add the following repository secrets:
     - `AWS_ACCESS_KEY_ID`: Your AWS access key with S3 and Glue permissions
     - `AWS_SECRET_ACCESS_KEY`: Your AWS secret key

2. **Set AWS Region** (Optional):
   - In the same "Actions" variables section, add a variable:
     - `AWS_REGION`: Your preferred AWS region (defaults to us-east-1)

3. **Configure Bucket Name**:
   - If you need to deploy to a different S3 bucket, edit the `.github/workflows/s3-deploy.yml` file

### Manual Workflow Execution

You can manually trigger the workflow:
1. Go to the "Actions" tab in your GitHub repository
2. Select "Deploy ETL Scripts to S3" workflow
3. Click "Run workflow"
4. Select the environment (dev/prod)
5. Click "Run workflow"

This will deploy the scripts and update the Glue jobs for the selected environment.

## Running the Pipeline

The pipeline is designed to run as an AWS Glue workflow with the following steps:

1. Raw data ingestion into S3
2. S3 to S3 ETL transformation (fhvhv_etl_job.py)
3. Redshift schema creation/verification (fhvhv_redshift_schema_from_catalog_job.py)
4. S3 to Redshift data load (fhvhv_redshift_load_job.py)

## Job Parameters

Each job requires specific parameters:

### fhvhv_etl_job.py
- `data_bucket_name`: S3 bucket containing the data
- `database_name`: Glue Data Catalog database name
- `fhvhv_table_name`: Table name for FHV data in Glue Catalog

### fhvhv_redshift_load_job.py
- `data_bucket_name`: S3 bucket containing the data
- `database_name`: Glue Data Catalog database name
- `redshift_connection`: Glue connection name for Redshift
- `redshift_database`: Redshift database name
- `redshift_schema`: Redshift schema name
- `redshift_table`: Target table in Redshift
- `redshift_host`: Redshift host (with port)
- `redshift_username`: Redshift username
- `redshift_password`: Redshift password

## Infrastructure as Code

The infrastructure for this pipeline is managed by Terraform in the companion repository `nash-dataops-terraform`.
