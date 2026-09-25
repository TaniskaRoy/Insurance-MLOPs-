#Vehicle Insurance | End-to-End MLOps Pipeline

An end-to-end machine learning and MLOps project for predicting whether an existing vehicle-insurance customer is likely to be interested in purchasing vehicle insurance.

The machine learning component uses a **Random Forest classifier** with preprocessing and hyperparameter tuning. The primary focus of the project is the **engineering and deployment lifecycle around the model** — from data ingestion and validation to model storage, containerization, CI/CD and deployment on AWS.

---

## What This Project Does

The project takes vehicle-insurance customer data, processes it through a modular ML pipeline, trains a classification model, evaluates the trained model, stores the model artifact in AWS S3, and exposes the prediction pipeline through a deployed application.

### End-to-end workflow

```text
                    DATA LAYER
                        │
                        ▼
              MongoDB Atlas
                        │
                        ▼
               Data Ingestion
                        │
                        ▼
               Data Validation
                        │
                        ▼
             Data Transformation
                        │
                        ▼
                  ML TRAINING
                        │
                        ▼
             Random Forest Model
                        │
                        ▼
              Model Evaluation
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
         Model Artifact      Prediction Pipeline
              │                   │
              ▼                   ▼
          AWS S3             Application
              │
              ▼
        Docker Container
              │
              ▼
         AWS ECR
              │
              ▼
          AWS EC2
              │
              ▼
        Deployed Service

        GitHub Actions
              │
              └──── CI/CD ──────► ECR / EC2
```

---

# Key Skills & Technologies

### Machine Learning

* Python
* Pandas
* NumPy
* Scikit-learn
* Exploratory Data Analysis
* Feature preprocessing
* Categorical encoding
* Feature scaling
* Train/Test splitting
* Random Forest Classification
* Hyperparameter tuning with `RandomizedSearchCV`
* Classification metrics

### MLOps / Software Engineering

* Modular ML pipeline architecture
* Data ingestion
* Data validation
* Data transformation
* Model training
* Model evaluation
* Model artifact management
* Prediction pipeline
* Logging
* Exception handling
* Configuration management
* Custom Python package structure
* `setup.py` / `pyproject.toml`

### Data & Cloud

* MongoDB Atlas
* AWS S3
* AWS IAM
* AWS ECR
* AWS EC2

### DevOps

* Docker
* GitHub Actions
* CI/CD
* Self-hosted GitHub Actions runner
* Environment variables and GitHub Secrets
* Container-based deployment

---

# 1. Machine Learning

## Problem Statement

The objective is to predict whether a customer will be interested in purchasing vehicle insurance.

The target variable is:

```text
Response

0 → Customer is not interested
1 → Customer is interested
```

The dataset contains customer, vehicle and policy-related information such as:

* Age
* Gender
* Driving license status
* Region
* Previous insurance status
* Vehicle age
* Vehicle damage history
* Annual premium
* Policy sales channel
* Customer vintage

The dataset contains **381,109 records and 12 columns** before preprocessing.

---

## Exploratory Data Analysis

The ML workflow starts with exploratory analysis of the dataset.

The notebook covers:

* Dataset dimensions and schema
* Missing-value checks
* Numerical feature statistics
* Target distribution
* Categorical feature analysis
* Vehicle damage vs. response analysis
* Annual premium distribution
* Feature relationships

The target is imbalanced:

```text
Response = 0 → 334,399
Response = 1 → 46,710
```

This makes class distribution an important consideration during model development.

---

## Data Preprocessing

The preprocessing pipeline converts the raw customer data into a numerical representation suitable for model training.

### Categorical preprocessing

Categorical variables are encoded using:

* Binary mapping for `Gender`
* One-hot encoding using `pandas.get_dummies()`
* Conversion of engineered categorical columns to integer representations

Examples include:

```text
Gender
Vehicle_Age
Vehicle_Damage
Driving_License
Previously_Insured
Region_Code
Policy_Sales_Channel
```

### Numerical preprocessing

Numerical features are scaled using:

* `StandardScaler` for selected numerical features
* `MinMaxScaler` for `Annual_Premium`

The customer `id` column is removed before model training because it does not represent a predictive feature.

---

# 2. Model Development

## Random Forest Classifier

The primary model used in the project is a:

**Random Forest Classifier**

Rather than using a single manually configured model, hyperparameter search is performed using:

```python
RandomizedSearchCV
```

The search explores parameters including:

```text
n_estimators
max_depth
min_samples_split
min_samples_leaf
criterion
```

The model is trained using cross-validation during hyperparameter search.

The selected model is then serialized as:

```text
rf_model.pkl
```

This trained artifact becomes the input to the downstream MLOps workflow.

---

## Model Evaluation

The trained classifier is evaluated against the test set using a classification report.

Evaluation includes standard classification metrics such as:

* Precision
* Recall
* F1-score
* Support

The evaluation stage is separated from model training so that the trained artifact can be evaluated before it is promoted to the model-storage/deployment stage.

---

# 3. MLOps Architecture

The main objective of this project is not to demonstrate an advanced ML algorithm.

The objective is to demonstrate how a machine learning model can be taken from a local experiment and converted into a **repeatable, modular and deployable ML system**.

The MLOps pipeline is organized into independent components:

```text
Data Ingestion
      ↓
Data Validation
      ↓
Data Transformation
      ↓
Model Training
      ↓
Model Evaluation
      ↓
Model Pushing
      ↓
Prediction Pipeline
      ↓
Containerization
      ↓
CI/CD
      ↓
AWS Deployment
```

---

# 4. Data Layer — MongoDB Atlas

Instead of keeping the training data only as a local CSV, the project integrates **MongoDB Atlas** as the data source.

The ingestion layer:

1. Connects to MongoDB using a connection string stored as an environment variable.
2. Retrieves the stored customer records.
3. Converts the retrieved data into a Pandas DataFrame.
4. Passes the data to the downstream ML pipeline.

The MongoDB connection is isolated inside the configuration/data-access layer rather than being embedded directly inside the training code.

```text
MongoDB Atlas
      │
      ▼
MongoDB Connection
      │
      ▼
Data Access Layer
      │
      ▼
Pandas DataFrame
      │
      ▼
ML Pipeline
```

---

# 5. Data Validation

The pipeline contains a dedicated data-validation stage.

Dataset expectations are defined through configuration/schema files, while validation utilities are separated from the training logic.

This provides a boundary between:

```text
Raw Data
   ↓
Validation
   ↓
Trusted Data
   ↓
Transformation
```

The goal is to prevent downstream components from blindly consuming an unexpected dataset structure.

---

# 6. Data Transformation

The transformation stage is responsible for converting validated input data into the representation required by the trained model.

Transformation logic is kept separate from model-training logic so that the same architectural boundary can be used when moving from training to prediction.

The project also uses an estimator abstraction to keep model-related operations separate from the pipeline orchestration.

---

# 7. Model Evaluation & Model Storage

After training, the model is evaluated before being pushed to cloud storage.

The project uses **AWS S3** for model artifact storage.

```text
Trained Model
     │
     ▼
Model Evaluation
     │
     ▼
Evaluation Check
     │
     ▼
AWS S3
(Model Registry / Storage)
```

The project defines configuration for:

```text
MODEL_BUCKET_NAME
MODEL_PUSHER_S3_KEY
MODEL_EVALUATION_CHANGED_THRESHOLD_SCORE
```

The model storage layer provides functionality to push and retrieve trained model artifacts from S3.

This separates the model artifact from the application/container itself.

---

# 8. Prediction Pipeline

A separate prediction pipeline is implemented for inference.

The deployed application does not need to retrain the model for every prediction.

Instead:

```text
User Input
    ↓
Prediction Pipeline
    ↓
Preprocessing
    ↓
Trained Model
    ↓
Prediction
```

The application layer is implemented using `app.py`, with supporting static/template directories for the web interface.

The project also exposes a training route for triggering model training through the application.

---

# 9. Docker

The application is containerized using Docker.

The repository contains:

```text
Dockerfile
.dockerignore
```

The Docker image packages the application and its runtime dependencies into a reproducible deployment unit.

Instead of configuring the Python environment manually on the deployment server, the application can be run as a container.

```text
Source Code
     ↓
Docker Build
     ↓
Docker Image
     ↓
AWS ECR
     ↓
AWS EC2
     ↓
Running Container
```

---

# 10. AWS Infrastructure

The project uses multiple AWS services for different parts of the ML lifecycle.

| AWS Service | Purpose                            |
| ----------- | ---------------------------------- |
| **S3**      | Model artifact storage             |
| **ECR**     | Docker image registry              |
| **EC2**     | Application hosting                |
| **IAM**     | Authentication and AWS permissions |

The project therefore separates:

* **Model storage** → S3
* **Container storage** → ECR
* **Application compute** → EC2

---

# 11. CI/CD Pipeline

The deployment process is automated using **GitHub Actions**.

The workflow is defined under:

```text
.github/workflows/
```

A code change followed by a Git push triggers the CI/CD workflow.

The pipeline integrates GitHub Actions with AWS and Docker.

### Deployment flow

```text
Developer
    │
    │ git push
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ├── Build application
    │
    ├── Build Docker image
    │
    ├── Authenticate with AWS
    │
    └── Push image
            │
            ▼
          AWS ECR
            │
            ▼
          AWS EC2
            │
            ▼
     Running Docker Container
```

AWS credentials and deployment configuration are provided through **GitHub Secrets** rather than being hard-coded into the repository.

Configured secrets include:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_DEFAULT_REGION
ECR_REPO
```

---

# 12. Self-Hosted GitHub Actions Runner

The EC2 instance is also configured as a **self-hosted GitHub Actions runner**.

This allows the GitHub Actions workflow to execute deployment-related operations on the configured EC2 infrastructure.

```text
GitHub Actions
      │
      ▼
Self-Hosted Runner
      │
      ▼
AWS EC2
      │
      ▼
Docker Deployment
```

This demonstrates an additional layer of infrastructure integration beyond simply deploying an application manually to EC2.

---

# 13. Logging & Exception Handling

The project includes dedicated logging and exception-handling utilities.

Instead of relying exclusively on print statements, pipeline components can use centralized logging and exception handling.

This is particularly useful for debugging failures across stages such as:

```text
Data Ingestion
Data Validation
Data Transformation
Model Training
Model Evaluation
Prediction
```

---

# 14. Project Structure

The project follows a modular structure separating configuration, entities, pipeline components, utilities and application code.

A simplified view:

```text
project/
│
├── components/
│   ├── data_ingestion.py
│   ├── data_validation.py
│   ├── data_transformation.py
│   ├── model_trainer.py
│   ├── model_evaluation.py
│   └── model_pusher.py
│
├── configuration/
│   ├── mongo_db_connections.py
│   └── aws_connection.py
│
├── data_access/
│
├── entity/
│   ├── config_entity.py
│   ├── artifact_entity.py
│   ├── estimator.py
│   └── s3_estimator.py
│
├── pipeline/
│   ├── training_pipeline.py
│   └── prediction_pipeline.py
│
├── utils/
│   └── main_utils.py
│
├── static/
├── template/
│
├── notebook/
│
├── app.py
├── Dockerfile
├── .dockerignore
├── requirements.txt
├── setup.py
├── pyproject.toml
└── .github/
    └── workflows/
        └── aws.yaml
```

---

# 15. Tech Stack

### Languages & ML

`Python` · `Pandas` · `NumPy` · `Scikit-learn`

### Database

`MongoDB Atlas`

### Cloud

`AWS S3` · `AWS ECR` · `AWS EC2` · `AWS IAM`

### DevOps

`Docker` · `GitHub Actions` · `CI/CD` · `Self-hosted Runner`

### Application

`Python` · `Flask` · `HTML/CSS`

### Engineering

`Logging` · `Exception Handling` · `Configuration Management` · `Modular Architecture`

---

# 16. Local Setup

### Clone the repository

```bash
git clone <repository-url>
cd <repository-name>
```

### Create environment

```bash
conda create -n vehicle python=3.10 -y
conda activate vehicle
```

### Install dependencies

```bash
pip install -r requirements.txt
```

### Configure environment variables

MongoDB:

```bash
export MONGODB_URL="<mongodb-connection-string>"
```

AWS:

```bash
export AWS_ACCESS_KEY_ID="<your-access-key>"
export AWS_SECRET_ACCESS_KEY="<your-secret-key>"
export AWS_DEFAULT_REGION="us-east-1"
```

Do not commit credentials or connection strings to the repository.

---

# 17. Running the Application

After configuring the required environment variables:

```bash
python app.py
```

The deployed application is configured to run on port:

```text
5080
```

For the AWS deployment, the EC2 security group must allow inbound traffic on the application port.

---

# 18. What This Project Demonstrates

This project demonstrates the complete lifecycle of a machine learning application:

```text
        MACHINE LEARNING
              │
              ▼
      Dataset & EDA
              │
              ▼
       Preprocessing
              │
              ▼
       Model Training
              │
              ▼
       Model Evaluation
              │
              │
              ▼
           MLOps
              │
              ▼
       Data Pipeline
              │
              ▼
      Artifact Management
              │
              ▼
        Cloud Storage
              │
              ▼
       Docker Container
              │
              ▼
        CI/CD Pipeline
              │
              ▼
       AWS Deployment
```

The machine learning model itself is intentionally straightforward. The engineering focus of the project is building the infrastructure required to **reliably move a machine learning model from experimentation to a deployable application**.

---

## Engineering Highlights

* End-to-end modular ML pipeline
* MongoDB-backed data ingestion
* Configurable data validation
* Dedicated transformation and training components
* Random Forest model with hyperparameter search
* Model evaluation before deployment
* Model artifact storage using AWS S3
* Dockerized application
* AWS ECR container registry
* AWS EC2 deployment
* GitHub Actions CI/CD
* Self-hosted GitHub Actions runner
* Environment-based configuration
* Logging and exception handling
* Separate training and prediction workflows

---

## Project Objective

The objective of this project is to demonstrate practical experience with the **machine learning lifecycle and MLOps engineering**, rather than focusing solely on model complexity.

The model represents the ML component of the system; the larger implementation demonstrates how data, models, infrastructure, containers and deployment automation can be connected into a reproducible end-to-end workflow.
