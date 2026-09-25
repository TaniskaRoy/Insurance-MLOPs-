# Vehicle Insurance | End-to-End MLOps Pipeline

## Project Overview

The project aims to predict whether a **registered vehicle-insurance customer is likely to purchase vehicle insurance** based on their existing customer and policy information. The application takes customer details as input and uses a **Random Forest classification model** to generate the prediction.

The project implements an end-to-end machine learning workflow for processing customer data, training and evaluating the model, with the trained model **integrated into a Flask API for serving predictions**.

The application follows a **modular architecture and MLOps workflow**, using **MongoDB Atlas** for data storage, **AWS S3** for model artifact storage, **AWS ECR** as the Docker container registry, and **AWS EC2** for application deployment. **GitHub Actions** is used to automate the CI/CD pipeline.

---

## Tech Stack

| Technology     | Category             |
| -------------- | -------------------- |
| Python         | Programming Language |
| Pandas         | Data Processing      |
| Scikit-learn   | Machine Learning     |
| Flask          | API / Model Serving  |
| MongoDB Atlas  | Database             |
| AWS S3         | Cloud Storage        |
| Docker         | Containerization     |
| AWS ECR        | Container Registry   |
| AWS EC2        | Cloud Compute        |
| GitHub Actions | CI/CD                |

---

## Project Architecture

The project is divided into two main workflows: a **Model Training & Model Registry Pipeline** for training, evaluating, and managing production model artifacts, and a **CI/CD & Deployment Pipeline** for containerizing and deploying the application.

### 1. Model Training & Model Registry Pipeline

```mermaid
flowchart LR
    A[MongoDB Atlas] --> B[Data Ingestion]
    B --> C[Data Validation]
    C --> D[Data Transformation]
    D --> E[Model Training]
    E --> F[Model Evaluation]

    G[AWS S3<br/>Existing Production Model] --> F

    F -->|Meets Evaluation Threshold| H[Model Pusher]
    H --> I[AWS S3<br/>Production Model Artifact]
```

### Pipeline Flow

* **MongoDB Atlas** — Stores the dataset used by the training pipeline.
* **Data Ingestion** — Retrieves data from MongoDB Atlas for processing.
* **Data Validation** — Validates the incoming data against the defined schema.
* **Data Transformation** — Applies the required preprocessing and feature transformations.
* **Model Training** — Trains the machine learning model using the transformed data.
* **Model Evaluation** — Evaluates the newly trained model against the existing production model.
* **Model Pusher** — Pushes the newly trained model to S3 when it meets the defined evaluation threshold.
* **AWS S3** — Stores the production model artifact used by the deployed application.
* **Logging & Exception Handling** — Provides application logging and structured exception handling across the pipeline for tracking execution and errors.

AWS S3 acts as the bridge between the training and serving workflows. The training pipeline compares the newly trained model with the existing production model. When the new model meets the evaluation criteria, it is stored as the production artifact in S3. The deployed Flask application retrieves this production model artifact for making predictions.

---

## Machine Learning Flow

The ML pipeline processes the data through the following stages:

* **Data Validation** — Validates the input data against the defined schema.
* **Categorical Encoding** — Converts categorical features into numerical representations.
* **Feature Scaling** — Scales numerical features for model training.
* **Random Forest Classifier** — Trains the classification model.
* **RandomizedSearchCV** — Performs hyperparameter tuning to identify suitable model parameters.
* **Model Evaluation** — Evaluates the trained model using classification metrics and the defined evaluation criteria.

---

### 2. CI/CD & Deployment Pipeline

```mermaid
flowchart LR
    A[GitHub Repository] --> B[GitHub Actions]
    B --> C[Self-Hosted Runner<br/>AWS EC2]
    C --> D[Docker Build]
    D --> E[AWS ECR]
    E --> F[AWS EC2<br/>Application Host]
    F --> G[Flask API]
    G --> H[Prediction]

    I[AWS S3<br/>Production Model Artifact] --> G
```

### Pipeline Flow

* **GitHub Repository** — Stores the application source code and CI/CD workflow configuration.
* **GitHub Actions** — Triggers the automated CI/CD workflow when changes are pushed to the repository.
* **Self-Hosted Runner** — Executes the GitHub Actions workflow on the configured EC2 instance.
* **Docker Build** — Packages the application and its dependencies into a Docker image.
* **AWS ECR** — Stores the built Docker image.
* **AWS EC2** — Pulls and runs the Docker image to host the deployed application.
* **Flask API** — Serves the trained model and handles prediction requests.
* **AWS S3** — Provides the production model artifact used by the deployed application.
---

