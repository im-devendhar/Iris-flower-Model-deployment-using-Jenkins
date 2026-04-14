# mlops-poc1

# 🌸 Iris Flower ML Model Deployment (CI/CD + Monitoring)

This project demonstrates an **end-to-end ML deployment pipeline** using:

* CI/CD with Jenkins
* Model storage using Amazon S3
* Monitoring with Datadog
* Flask API for serving predictions

---

# 🚀 Architecture

```
GitHub → Jenkins → Train Model → Upload to S3 → Deploy Flask API
                                          ↓
                                   Datadog Monitoring
```

---

# 📁 Project Structure

```
iris-ml-app/
│
├── app.py
├── model.py
├── requirements.txt
├── Jenkinsfile
└── README.md
```

---

# ⚙️ Prerequisites

* Python 3.x
* Jenkins installed and running
* AWS account (S3 bucket created)
* Datadog account (API key)

---

# ☁️ Step 1: Setup Amazon S3

1. Create an S3 bucket (example: `iris-model-bucket`)
2. Note:

   * Bucket Name
   * AWS Access Key & Secret Key

---

# 🧠 Step 2: Model Training Script

## `model.py`

```python
import boto3
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
import pickle

BUCKET_NAME = "your-bucket-name"
MODEL_FILE = "iris_model.pkl"

# Load dataset
iris = load_iris()
X, y = iris.data, iris.target

# Train model
model = RandomForestClassifier()
model.fit(X, y)

# Save model
with open(MODEL_FILE, "wb") as f:
    pickle.dump(model, f)

# Upload to S3
s3 = boto3.client('s3')
s3.upload_file(MODEL_FILE, BUCKET_NAME, MODEL_FILE)

print("Model trained and uploaded to S3!")
```

---

# 🌐 Step 3: Flask API

## `app.py`

```python
from ddtrace import patch_all
patch_all()

import boto3
import pickle
from flask import Flask, request, jsonify

app = Flask(__name__)

BUCKET_NAME = "your-bucket-name"
MODEL_FILE = "iris_model.pkl"

# Download model from S3
s3 = boto3.client('s3')
s3.download_file(BUCKET_NAME, MODEL_FILE, MODEL_FILE)

model = pickle.load(open(MODEL_FILE, "rb"))

@app.route('/')
def home():
    return "Iris Model Running with S3 + Datadog!"

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json['features']
    prediction = model.predict([data])
    return jsonify({'prediction': int(prediction[0])})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

# 📦 Step 4: Requirements

## `requirements.txt`

```
flask
scikit-learn
boto3
ddtrace
```

---

# ⚙️ Step 5: Jenkins Pipeline

## `Jenkinsfile`

```groovy
pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = 'your-access-key'
        AWS_SECRET_ACCESS_KEY = 'your-secret-key'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git 'https://github.com/your-repo/iris-ml-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Train & Upload Model to S3') {
            steps {
                sh 'python model.py'
            }
        }

        stage('Run Flask App with Datadog') {
            steps {
                sh 'nohup ddtrace-run python app.py &'
            }
        }
    }
}
```

---

# 📊 Step 6: Setup Datadog

## Install Datadog Agent

```bash
DD_API_KEY=your_api_key bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"
```

---

## Run Flask with APM

```bash
ddtrace-run python app.py
```

---

# 🧪 Step 7: Test API

```bash
curl -X POST http://<server-ip>:5000/predict \
-H "Content-Type: application/json" \
-d '{"features": [5.1, 3.5, 1.4, 0.2]}'
```

---

# ✅ Sample Output

```
{"prediction": 0}
```

* 0 → Setosa
* 1 → Versicolor
* 2 → Virginica

---

# 📊 Monitoring (Datadog)

In Datadog dashboard, you can monitor:

* API request rate
* Response time
* Error rates
* Endpoint tracing (`/predict`)

---

# 🔐 Best Practices

* Use IAM Roles instead of access keys
* Enable S3 versioning for model rollback
* Store secrets in Jenkins credentials
* Add logging & alerts in Datadog

---

# 🚀 Future Enhancements

* Dockerize the application
* Deploy to Kubernetes (EKS)
* Add CI triggers using GitHub Webhooks
* Implement Blue-Green Deployment

---

---

# 👨‍💻 Author

DevOps + ML Deployment POC
