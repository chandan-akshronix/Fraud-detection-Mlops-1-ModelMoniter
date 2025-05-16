# Fraud-detection-Mlops-1-ModelMoniter

This repository is the monitoring component of the fraud detection MLOps pipeline. It configures and deploys Amazon SageMaker monitoring schedules to track the performance, data quality, bias, and explainability of the deployed fraud detection model. The monitoring schedules are set up for both staging and production environments, ensuring continuous oversight of the model's behavior in real-time.

## Features

- **Data Quality Monitoring**: Detects data drift and anomalies in incoming data.
- **Model Quality Monitoring**: Tracks the model's prediction performance (requires ground truth labels).
- **Bias Monitoring**: Identifies potential bias in the model's predictions (requires ground truth labels).
- **Explainability Monitoring**: Analyzes feature importance and model interpretability.
- **Environment-Specific Configurations**: Supports separate monitoring setups for staging and production.
- **Automated Deployment**: Uses AWS CodeBuild and CloudFormation for infrastructure as code.
- **Integration with SageMaker Model Registry**: Retrieves baselines for monitoring from the registered model.

## File Structure

The repository is structured as follows:

```
Fraud-detection-Mlops-1-ModelMoniter/
├── pipelines/
│   └── __version__.py          # Version information for the pipeline module
├── README.md                   # Project documentation (this file)
├── __init__.py                 # Python package initialization
├── buildspec.yml               # AWS CodeBuild specification for building and deploying monitoring
├── get_baselines_and_configs.py # Script to retrieve baselines and prepare monitoring configs
├── model-monitor-template.yml  # CloudFormation template for monitoring schedules
├── prod-monitoring-schedule-config.json  # Production monitoring configuration
├── staging-monitoring-schedule-config.json # Staging monitoring configuration
└── utils.py                    # Utility functions for configuration and S3 operations
```

## File Descriptions

### `pipelines/__version__.py`
- **Purpose**: Stores version information for the pipeline module.
- **How It Works**: Provides a version string (e.g., "1.0.0") for tracking changes.

### `README.md`
- **Purpose**: Documents the repository, providing an overview, setup instructions, and file descriptions.

### `__init__.py`
- **Purpose**: Initializes the Python package, allowing imports of modules within the repository.

### `buildspec.yml`
- **Purpose**: Defines the AWS CodeBuild process for building and deploying the monitoring schedules.
- **How It Works**: Specifies the environment, installs dependencies, runs scripts, and packages the CloudFormation template.

### `get_baselines_and_configs.py`
- **Purpose**: Retrieves baselines from the SageMaker Model Registry and prepares monitoring configurations.
- **How It Works**: Fetches baselines, processes them, and updates configuration files for staging and production.

### `model-monitor-template.yml`
- **Purpose**: CloudFormation template that defines SageMaker monitoring resources.
- **How It Works**: Creates job definitions and schedules for various monitoring types.

### `prod-monitoring-schedule-config.json`
- **Purpose**: Configuration file for production monitoring with production-specific settings.

### `staging-monitoring-schedule-config.json`
- **Purpose**: Configuration file for staging monitoring with staging-specific settings.

### `utils.py`
- **Purpose**: Contains utility functions for handling configurations, S3 operations, and error handling.

## Getting Started

### Prerequisites

- **AWS Account**: With access to SageMaker, S3, and IAM.
- **Python**: Version 3.11 or later.
- **Dependencies**:
  - `sagemaker==2.243.1`
  - `boto3`, `botocore`, `awscli`
- **AWS CLI**: Configured with appropriate credentials.
- **Git**: For cloning the repository.

### Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/chandan-akshronix/Fraud-detection-Mlops-1-ModelMoniter.git
   cd Fraud-detection-Mlops-1-ModelMoniter
   ```

2. **Install Dependencies** (for local testing):
   ```bash
   pip install sagemaker==2.93.0 boto3 botocore awscli
   ```

## Usage

The monitoring setup is automated via AWS CodeBuild and CloudFormation. Follow these steps:

1. **Prepare Configurations**:
   - Run the script with required arguments:
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
   - Check the SageMaker console for active schedules.
   - Review S3 outputs in the specified bucket.

## Monitoring Workflow

1. **Baseline Retrieval**: Fetches baselines from the Model Registry.
2. **Configuration Preparation**: Processes baselines and updates configurations.
3. **Monitoring Deployment**: Deploys schedules via CloudFormation.
4. **Execution**: Runs monitoring jobs hourly, storing results in S3.

## Integration with the MLOps Pipeline

- **ModelBuild**: Provides the trained model and baselines.
- **ModelDeploy**: Deploys the endpoint monitored here.
- **ModelMoniter**: Configures and deploys monitoring schedules.

## License

This project is licensed under the terms specified in the `LICENSE` file (if available).
