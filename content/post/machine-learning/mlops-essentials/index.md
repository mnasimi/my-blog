---
title: "MLOps Essentials: From Notebook to Production"
description: "Learn the practices and tools for deploying and maintaining ML models in production."
slug: mlops-essentials
date: 2024-04-28
categories:
  - Machine Learning
tags:
  - mlops
  - deployment
  - ml-engineering
  - production
---

Training a model is just the beginning. MLOps bridges the gap between data science experiments and production systems.

## What Is MLOps?

MLOps applies DevOps principles to machine learning:

```
┌─────────────────────────────────────────────────────────┐
│                       MLOps                              │
├──────────────┬──────────────┬──────────────┬────────────┤
│    Data      │    Model     │   Deploy     │   Monitor  │
│  Management  │   Training   │              │            │
└──────────────┴──────────────┴──────────────┴────────────┘
```

## The ML Lifecycle

```
┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐
│ Data │───▶│Train │───▶│Eval  │───▶│Deploy│───▶│Monitor│
└──────┘    └──────┘    └──────┘    └──────┘    └──────┘
    ▲                                               │
    └───────────────── Feedback Loop ──────────────┘
```

## Version Control Everything

### Code
Standard Git for training scripts, pipelines, etc.

### Data
Track data versions with tools like DVC:

```bash
# Initialize DVC
dvc init

# Track a dataset
dvc add data/training_set.csv

# Push to remote storage
dvc push
```

### Models
Track model versions with metadata:

```python
import mlflow

with mlflow.start_run():
    mlflow.log_param("learning_rate", 0.01)
    mlflow.log_param("epochs", 100)
    mlflow.log_metric("accuracy", 0.95)
    mlflow.sklearn.log_model(model, "model")
```

## Reproducibility

### Environment Management

```yaml
# environment.yml
name: ml-project
dependencies:
  - python=3.9
  - scikit-learn=1.0
  - pandas=1.4
  - tensorflow=2.8
```

### Containerization

```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY model/ ./model/
COPY app.py .

CMD ["python", "app.py"]
```

## Model Serving

### REST API with FastAPI

```python
from fastapi import FastAPI
import joblib

app = FastAPI()
model = joblib.load("model.pkl")

@app.post("/predict")
async def predict(features: dict):
    prediction = model.predict([features["data"]])
    return {"prediction": prediction.tolist()}
```

### Serving Options

| Option | Best For |
|--------|----------|
| REST API | General web applications |
| gRPC | High-performance, internal services |
| Batch inference | Large-scale offline predictions |
| Edge deployment | Mobile/IoT devices |

## Monitoring in Production

### What to Monitor

```
┌─────────────────────────────────────────────────┐
│               ML Monitoring                      │
├────────────────┬────────────────┬───────────────┤
│   Data Drift   │  Model Perf    │   System      │
│                │                │   Health      │
│ - Input stats  │ - Accuracy     │ - Latency     │
│ - Distributions│ - Precision    │ - Throughput  │
│ - Missing vals │ - Recall       │ - Errors      │
└────────────────┴────────────────┴───────────────┘
```

### Data Drift Detection

```python
from evidently.metrics import DataDriftPreset
from evidently.report import Report

report = Report(metrics=[DataDriftPreset()])
report.run(
    reference_data=training_data,
    current_data=production_data
)
```

### Model Performance Tracking

```python
# Log predictions and actual outcomes
logger.info({
    "prediction_id": uuid4(),
    "features": features,
    "prediction": prediction,
    "timestamp": datetime.now()
})

# Compare with actual outcomes later
# Alert if accuracy drops below threshold
```

## CI/CD for ML

```yaml
# .github/workflows/ml-pipeline.yml
name: ML Pipeline

on:
  push:
    branches: [main]

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Train model
        run: python train.py
      - name: Run tests
        run: pytest tests/
      - name: Evaluate model
        run: python evaluate.py
      - name: Deploy if improved
        if: ${{ steps.evaluate.outputs.improved == 'true' }}
        run: ./deploy.sh
```

## Key MLOps Tools

| Category | Tools |
|----------|-------|
| Experiment Tracking | MLflow, Weights & Biases, Neptune |
| Data Versioning | DVC, Delta Lake, LakeFS |
| Model Serving | TensorFlow Serving, TorchServe, Seldon |
| Orchestration | Kubeflow, Airflow, Prefect |
| Monitoring | Evidently, Whylabs, Arize |

## Conclusion

MLOps is essential for sustainable ML in production. Start with experiment tracking and version control, then gradually add deployment automation and monitoring as your needs grow.
