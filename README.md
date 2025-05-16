# Fraud-detection-Mlops-1-ModelMoniter

This repository is the monitoring component of the fraud detection MLOps pipeline, designed to oversee the performance, data quality, bias, and explainability of a deployed fraud detection model. It leverages Amazon SageMaker Model Monitor and SageMaker Clarify to configure and deploy monitoring schedules for both staging and production environments. Additionally, it integrates with a Lambda function that triggers real-time inference on streaming data, storing predictions in Amazon DynamoDB for further analysis.

## Features

- **Data Quality Monitoring**: Detects data drift and anomalies in input data.
- **Model Quality Monitoring**: Tracks prediction performance metrics (requires ground truth labels).
- **Bias Monitoring**: Identifies potential biases in predictions (requires ground truth labels).
- **Explainability Monitoring**: Analyzes feature importance for model interpretability.
- **Environment-Specific Configurations**: Supports distinct monitoring setups for staging and production.
- **Automated Deployment**: Uses AWS CodeBuild and CloudFormation for infrastructure as code.
- **Lambda Trigger Integration**: Processes streaming data via a Lambda function, invoking the SageMaker endpoint and storing results in DynamoDB.
- **Model Registry Integration**: Retrieves baselines from the SageMaker Model Registry for monitoring.

## File Structure

The repository is organized as follows:

```
Fraud-detection-Mlops-1-ModelMoniter/
├── pipelines/
│   └── __version__.py          # Version information for the pipeline module
├── README.md                   # Project documentation (this file)
├── __init__.py                 # Python package initialization
├── buildspec.yml               # AWS CodeBuild specification for building and deploying
├── get_baselines_and_configs.py # Script to retrieve baselines and prepare configs
├── model-monitor-template.yml  # CloudFormation template for monitoring schedules
├── prod-monitoring-schedule-config.json  # Production monitoring configuration
├── staging-monitoring-schedule-config.json # Staging monitoring configuration
└── utils.py                    # Utility functions for configuration and S3 operations
```

**Note**: The Lambda function (`lambda_function.py`) is not part of this repository but is a critical trigger mechanism in the broader pipeline. Its details are described below under "Lambda Trigger Integration."

## File Descriptions

### `pipelines/__version__.py`
- **Purpose**: Stores version metadata for the pipeline module.
- **How It Works**: Defines a version string (e.g., "1.0.0") for tracking changes.
- **Input**: None.
- **Output**: Version string accessible via imports.

### `buildspec.yml`
- **Purpose**: Defines the AWS CodeBuild process for deploying monitoring schedules.
- **How It Works**: Specifies the Python 3.11 environment, installs dependencies (e.g., `sagemaker==2.93.0`), executes `get_baselines_and_configs.py`, and packages the CloudFormation template.
- **Input**: Source code and environment variables (e.g., `MODEL_MONITOR_ROLE_ARN`).
- **Output**: Packaged CloudFormation template and exported configuration files.

### `get_baselines_and_configs.py`
- **Purpose**: Retrieves baselines from the SageMaker Model Registry and prepares monitoring configurations.
- **How It Works**: Fetches model baselines, processes bias and explainability configurations, and extends staging/production configs with parameters like S3 URIs and image URIs.
- **Input**: Command-line arguments (e.g., `--model-monitor-role`, `--sagemaker-project-name`).
- **Output**: Updated JSON configuration files (`staging-output.json`, `prod-output.json`).

### `model-monitor-template.yml`
- **Purpose**: CloudFormation template defining SageMaker monitoring resources.
- **How It Works**: Creates job definitions and schedules for data quality, model quality, bias, and explainability monitoring, with configurable parameters (e.g., instance type, schedule expressions).
- **Input**: Parameters from configuration files (e.g., `EndpointName`, `MonitoringScheduleName`).
- **Output**: Deployed monitoring schedules in SageMaker.

### `prod-monitoring-schedule-config.json`
- **Purpose**: Configures monitoring for the production environment.
- **How It Works**: Specifies parameters like hourly schedules (`cron(0 * ? * * *)`), instance type (`ml.m5.large`), and monitoring types (all enabled).
- **Input**: None (static configuration).
- **Output**: JSON parameters for CloudFormation.

### `staging-monitoring-schedule-config.json`
- **Purpose**: Configures monitoring for the staging environment.
- **How It Works**: Similar to the production config but includes a `GroundTruthInput` S3 URI for model quality and bias monitoring.
- **Input**: None (static configuration).
- **Output**: JSON parameters for CloudFormation.

### `utils.py`
- **Purpose**: Provides utility functions for configuration management and S3 operations.
- **How It Works**: Includes functions for fetching baselines, processing bias/explainability configs, handling S3 files, and managing tags, with robust error handling.
- **Input**: Varies by function (e.g., S3 URIs, boto3 clients).
- **Output**: Processed data or configurations (e.g., updated JSON files, S3 uploads).

## Lambda Trigger Integration

The pipeline includes a Lambda function (`lambda_function.py`) that triggers real-time inference on streaming data, complementing the monitoring schedules. While not part of this repository, it is a critical component of the fraud detection pipeline.

### `lambda_function.py` (External)
- **Purpose**: Processes streaming data from S3, invokes the SageMaker endpoint for predictions, and stores results in DynamoDB.
- **How It Works**:
  - Triggered by new data in an S3 bucket (e.g., `akshronix-frauddata/Chandan's Playground MLOPS/stream_data/stream_100.csv`).
  - Reads CSV data, preprocesses numerical fields (e.g., `Transaction Amount`, `Quantity`), and converts rows to JSON.
  - Invokes the SageMaker endpoint (`Fraud-Detection-Test-MLOPS-10-prod`) for each row.
  - Stores predictions with `TransactionID` and timestamp in DynamoDB (`StorageDB-prod`).
- **Input**: S3 event notification for new CSV files.
- **Output**: DynamoDB records with transaction IDs, predictions, and timestamps; HTTP response (status code 200).
- **Key Components**:
  - **S3 Client**: Fetches CSV data.
  - **SageMaker Runtime Client**: Invokes the endpoint.
  - **DynamoDB Client**: Writes predictions.
  - **Error Handling**: Skips invalid rows and logs errors.

## Getting Started

### Prerequisites

- **AWS Account**: With permissions for SageMaker, S3, IAM, Lambda, and DynamoDB.
- **Python**: Version 3.11 or later.
- **Dependencies**:
  - `sagemaker==2.243.1`
  - `boto3`, `botocore`, `awscli`
- **AWS CLI**: Configured with valid credentials.
- **Git**: For cloning the repository.
- **Deployed Endpoint**: From the `Fraud-detection-Mlops-1-ModelDeploy` repository.
- **Lambda Function**: Deployed with appropriate IAM roles and S3 trigger.

### Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/chandan-akshronix/Fraud-detection-Mlops-1-ModelMoniter.git
   cd Fraud-detection-Mlops-1-ModelMoniter
   ```

2. **Install Dependencies** (for local testing):
   ```bash
   pip install sagemaker==2.243.1 boto3 botocore awscli
   ```

### Lambda Function Setup

1. **Create the Lambda Function**:
   - In the AWS Lambda console, create a function with Python 3.11 runtime.
   - Upload `lambda_function.py` or paste its code.
   - Set the environment variables:
     - `DYNAMODB_TABLE_NAME`: `StorageDB-prod`
   - Configure an S3 trigger for the bucket (e.g., `akshronix-frauddata`) and prefix (e.g., `Chandan's Playground MLOPS/stream_data/`).

2. **IAM Role**:
   - Attach policies for S3 read, SageMaker endpoint invocation, DynamoDB write, and CloudWatch logs.

3. **Test the Function**:
   - Upload a sample CSV to the S3 bucket and verify DynamoDB entries.

## Usage

### Deploy Monitoring Schedules

1. **Prepare Configurations**:
   - Run the configuration script:
     ```bash
     python get_baselines_and_configs.py \
         --model-monitor-role "arn:aws:iam::123456789012:role/ModelMonitorRole" \
         --sagemaker-project-id "p-12345" \
         --sagemaker-project-name "FraudDetection" \
         --monitor-outputs-bucket "sagemaker-monitor-output" \
         --export-staging-config "staging-output.json" \
         --export-prod-config "prod-output.json"
     ```

2. **Deploy the CloudFormation Stack**:
   - Deploy using the AWS CLI:
     ```bash
     aws cloudformation deploy \
         --template-file <EXPORT_TEMPLATE_NAME> \
         --stack-name FraudDetectionMonitoring \
         --capabilities CAPABILITY_NAMED_IAM \
         --parameter-overrides file://staging-output.json  # or prod-output.json
     ```

3. **Verify Monitoring**:
   - Check the SageMaker console for active monitoring schedules.
   - Review S3 outputs in `sagemaker-monitor-output/monitor-output/` for each monitoring type (data-quality, model-quality, model-bias, model-explainability).

### Monitor Streaming Predictions

- **Trigger**: New CSV files in the S3 bucket (e.g., `akshronix-frauddata/Chandan's Playground MLOPS/stream_data/`) activate the Lambda function.
- **Output**: Predictions are stored in DynamoDB (`StorageDB-prod`) with `TransactionID`, `Prediction`, and `Timestamp`.
- **Verification**: Query DynamoDB or use AWS Athena to analyze predictions.

## Monitoring Workflow

1. **Baseline Retrieval**: Fetches data quality, model quality, bias, and explainability baselines from the Model Registry.
2. **Configuration Preparation**: Processes baselines (e.g., combines bias constraints) and updates configs.
3. **Monitoring Deployment**: Deploys hourly schedules via CloudFormation.
4. **Execution**: Jobs run, storing results in S3 for analysis.
5. **Streaming Inference**: The Lambda function processes real-time data, invoking the endpoint and logging predictions.

## Integration with the MLOps Pipeline

- **ModelBuild**: Trains and registers the model with baselines.
- **ModelDeploy**: Deploys the SageMaker endpoint.
- **ModelMoniter**: Monitors the endpoint, complemented by the Lambda function for real-time inference.
- **Lambda Function**: Triggers inference on streaming data, feeding predictions to DynamoDB for monitoring integration.

## License

This project is licensed under the terms specified in the `LICENSE` file (if available).