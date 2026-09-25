# Vehicle Insurance | End-to-End MLOps Pipeline

## 📌 Project Overview

An end-to-end **Machine Learning and MLOps pipeline** for predicting whether an existing vehicle-insurance customer is likely to be interested in purchasing vehicle insurance.

The project combines a **Random Forest classification model** with a complete MLOps workflow covering data storage, ingestion, validation, transformation, model evaluation, artifact management, containerization, CI/CD, cloud infrastructure, and model serving.

The ML model is developed using **Scikit-learn**, while the deployment and infrastructure layer is built using **MongoDB Atlas, Docker, AWS, GitHub Actions, and Flask**.

---

## 🛠️ Tech Stack

| Category                     | Technology                        | Purpose                                    |
| ---------------------------- | --------------------------------- | ------------------------------------------ |
| 🐍 **Programming**           | Python                            | Core application and ML development        |
| 🤖 **Machine Learning**      | Scikit-learn                      | Model development and preprocessing        |
| 🌲 **ML Model**              | Random Forest                     | Binary classification                      |
| 🎯 **Hyperparameter Tuning** | RandomizedSearchCV                | Model parameter optimization               |
| 🗄️ **Database**             | MongoDB Atlas                     | Cloud-based dataset storage                |
| 🐳 **Containerization**      | Docker                            | Package application and dependencies       |
| ☁️ **Cloud Storage**         | Amazon S3                         | Store production model artifacts           |
| 📦 **Container Registry**    | Amazon ECR                        | Store Docker images                        |
| 🖥️ **Cloud Compute**        | Amazon EC2                        | Host application and CI/CD runner          |
| 🔐 **Cloud Security**        | AWS IAM                           | Manage AWS access and permissions          |
| 🔄 **CI/CD**                 | GitHub Actions                    | Automate build and deployment              |
| 🖥️ **CI/CD Runner**         | Self-Hosted GitHub Actions Runner | Execute deployment workflow on EC2         |
| 🌐 **Web Framework**         | Flask                             | Model serving and prediction application   |
| 📊 **Data Processing**       | Pandas, NumPy                     | Data manipulation and numerical processing |

---

# 🏗️ Project Architecture

The project is divided into two connected layers:

* **Machine Learning Pipeline** — handles the movement of data from ingestion through model training and evaluation.
* **MLOps & Deployment Pipeline** — handles model artifact storage, containerization, CI/CD, cloud infrastructure, deployment, and serving.

```mermaid
flowchart TD
    %% Styling and Palettes
    classDef storage fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B5E20;
    classDef pipeline fill:#E3F2FD,stroke:#1565C0,stroke-width:2px,color:#0D47A1;
    classDef eval fill:#FFF8E1,stroke:#F57F17,stroke-width:2px,color:#E65100;
    classDef cicd fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px,color:#4A148C;
    classDef app fill:#FBE9E7,stroke:#D84315,stroke-width:2px,color:#BF360C;

    %% Data Source & DB
    subgraph S1[" 🗄️ Data Ingestion & Storage "]
        A[("Raw Dataset<br/>(Notebook)")] -->|Push Data| B[("MongoDB Atlas<br/>(Cloud Database)")]
        B -->|Fetch Records to DF| C["Data Ingestion Component"]
    end
    class A,B storage;

    %% Training Pipeline Components
    subgraph S2[" ⚙️ Training Pipeline "]
        C --> D["Data Validation<br/>(schema.yaml)"]
        D --> E["Data Transformation<br/>(Preprocessors)"]
        E --> F["Model Trainer<br/>(Estimator)"]
    end
    class C,D,E,F pipeline;

    %% Evaluation, Registry & S3
    subgraph S3[" 🎯 Evaluation & Registry "]
        F --> G{"Model Evaluation"}
        H[("AWS S3 Bucket<br/>(Production Model)")] -.->|Pull Existing Model| G
        G -->|Score > Threshold| I["Model Pusher"]
        I -->|Upload Best Model| H
    end
    class G,I eval;
    class H storage;

    %% CI / CD & Deployment
    subgraph S4[" 🚀 CI/CD & Cloud Infrastructure "]
        J["GitHub Repository<br/>(Git Push / Actions)"] -->|Trigger Workflow| K["GitHub Self-Hosted Runner<br/>(EC2 Instance)"]
        K -->|Build & Push Image| L["AWS ECR<br/>(Docker Registry)"]
        L -->|Pull Docker Container| M["AWS EC2 Host<br/>(Ubuntu 24.04 / Port 5080)"]
    end
    class J,K,L,M cicd;

    %% Serving & Inference
    subgraph S5[" 🌐 Prediction & Serving "]
        H -.->|Fetch Production Artifact| N["Flask App (app.py)"]
        M --- N
        O["User / Client"] -->|Inference Form / UI| N
        O -->|Trigger Pipeline /training| N
    end
    class N,O app;
```

### 🤖 Machine Learning Flow

The ML pipeline handles the movement of data from the database to the trained model:

**MongoDB Atlas → Data Ingestion → Data Validation → Data Transformation → Model Training → Model Evaluation**

The ML workflow uses:

* **Data validation** using a defined schema
* **Categorical encoding**
* **Feature scaling**
* **Random Forest Classifier**
* **RandomizedSearchCV** for hyperparameter tuning
* **Model evaluation** against the defined evaluation criteria

The trained model is then passed to the MLOps layer for artifact management and deployment.

### ⚙️ MLOps & Deployment Flow

The MLOps layer manages the lifecycle of the trained model and application:

**Model Evaluation → Model Pusher → Amazon S3**

The production model is stored in **Amazon S3**, providing a persistent model artifact that can be accessed by the prediction application.

For application deployment:

**GitHub → GitHub Actions → Self-Hosted Runner → Docker → Amazon ECR → Amazon EC2**

GitHub Actions automates the deployment workflow. The application is packaged into a Docker image, pushed to Amazon ECR, and deployed on an EC2 instance.

The deployed **Flask application** retrieves the production model artifact from S3 and provides the prediction interface.

### ☁️ Cloud Architecture

AWS is used across different parts of the system based on their respective responsibilities:

* **S3** — production model artifact storage
* **ECR** — Docker image registry
* **EC2** — application hosting and self-hosted CI/CD runner
* **IAM** — AWS authentication and permissions

MongoDB Atlas is used separately as the **cloud data storage layer**, while AWS provides the infrastructure for **model management and application deployment**.
