# 🎓 Senior MLOps Mentor Guide — Animal Classifier on SageMaker

> A complete, no-fluff reference written by a senior MLOps engineer. Every answer is grounded in **your actual code**.

---

## 📋 Table of Contents

1. [Is My Code Good?](#1-is-my-code-good)
2. [Handling Large GB-scale Datasets](#2-handling-large-gb-scale-datasets)
3. [CI/CD Pipeline — Automating SageMaker Notebook Runs](#3-cicd-pipeline)
4. [Multiple Training Runs — How Models Are Stored & Tracked](#4-multiple-training-runs)
5. [Using the Model in a Full-Stack Backend](#5-using-the-model-in-full-stack-backend)
6. [Data Versioning — What to Use](#6-data-versioning)

---

## 1. Is My Code Good?

### ✅ What You're Doing Right

| Area | What you did | Why it's good |
|---|---|---|
| **SageMaker contract** | `ENTRYPOINT ["python", "train.py"]` | SageMaker calls the container this way. Correct. |
| **Paths** | `/opt/ml/input/data/training`, `/opt/ml/model` | These are SageMaker's standard paths. ✅ |
| **Best model tracking** | `copy.deepcopy(model)` when `accuracy > best_accuracy` | You save the best checkpoint, not just the last. Professional habit. |
| **Config-driven** | `config.json` drives all hyperparameters | Makes it easy to change without touching code. |
| **Separation of concerns** | `dataset.py`, `trainer.py`, `evaluator.py`, `model.py` | Clean module separation. Scales well. |
| **Docker ignore** | `.dockerignore` exists | Keeps the Docker image lean. |

---

### ⚠️ Real Issues to Fix (with exact locations)

#### Issue 1 — Duplicate `Estimator` instantiation in your SageMaker notebook

**In your SageMaker notebook:**
```python
# ❌ You create the Estimator TWICE. The first one is immediately overwritten.
estimator = Estimator(image_uri=image_uri, role=role)          # ← DELETE THIS BLOCK

estimator = Estimator(                                          # ← KEEP only this one
    image_uri=image_uri,
    role=role,
    instance_type="ml.g5.xlarge",
    instance_count=1,
    output_path="s3://maaz-animal-classifier-v3/model-artifacts/",
    sagemaker_session=session,
    base_job_name="animal-classifier"
)
```

**Fix:** Remove lines 11–13 (the first `estimator = Estimator(...)`). Only keep the full one.

---

#### Issue 2 — `dataset_dir` hardcoded inside `dataset.py` instead of reading from `config.json`

**In [dataset.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/src/dataset.py) line 23–24:**
```python
# ❌ The commented line is the RIGHT approach. The active line is redundant.
# dataset_dir = config["data"]["DATASET_DIR"]    ← UNCOMMENT THIS
dataset_dir = "/opt/ml/input/data/training"      # ← DELETE THIS LINE
```

Your `config.json` already has `"DATASET_DIR": "/opt/ml/input/data/training"`. Use it. This way when you want to test locally, you change just the config — not the source code.

---

#### Issue 3 — Double `optimizer.zero_grad()` in trainer

**In [trainer.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/src/trainer.py) lines 41 and 50:**
```python
optimizer.zero_grad()    # line 41 ← KEEP THIS (before forward pass)
outputs = model(images)
loss = criterion(outputs, labels)
loss.backward()
optimizer.step()
optimizer.zero_grad()    # line 50 ← DELETE THIS (redundant, runs again next iteration anyway)
```

Calling `zero_grad()` after `step()` is harmless but wasteful noise. The standard PyTorch loop calls it once, at the TOP of each batch iteration (line 41). Remove the second one.

---

#### Issue 4 — No `class_names` saved with model

**In [train.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/train.py) line 30:**
```python
# ❌ Only weights saved. When you load the model later, you won't know which index = which animal.
torch.save(best_model.state_dict(), f"{MODEL_DIR}/best_model.pth")
```

**What a senior engineer adds (without changing logic):**
```python
import json
# Save class names alongside weights so inference knows the label mapping
with open(f"{MODEL_DIR}/class_names.json", "w") as f:
    json.dump(class_names, f)

torch.save(best_model.state_dict(), f"{MODEL_DIR}/best_model.pth")
```

Without `class_names.json`, when you deploy your model and it predicts index `2`, you won't know if that's "cat", "dog" or "elephant".

---

#### Issue 5 — No logging (only `print`)

**In [trainer.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/src/trainer.py) line 67:**
```python
# ❌ print statements don't integrate with SageMaker Experiments or any monitoring
print(f"Epoch [{epoch+1}/{epochs}], Loss: {epoch_loss:.4f}, Accuracy: {accuracy:.4f}")
```

SageMaker captures `stdout`, so `print` works but gives you zero observability in CloudWatch or SageMaker Experiments dashboards.

**What to do:**
```python
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Then replace print with:
logger.info(f"Epoch [{epoch+1}/{epochs}], Loss: {epoch_loss:.4f}, Accuracy: {accuracy:.4f}")
```

---

#### Issue 6 — `inference.py` is empty

**File:** [inference.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/inference.py) is completely empty.

This file is critical for SageMaker **real-time endpoints**. Without it, you cannot deploy the trained model as an API. We'll cover what goes in it in **Topic 5**.

---

#### Issue 7 — `FROM python:3.13-slim` in Dockerfile

**In [Dockerfile](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/Dockerfile) line 1:**
```dockerfile
FROM python:3.13-slim   # ← Python 3.13 is very new, PyTorch support is uneven
```

For ML workloads, use a **PyTorch base image** from AWS Deep Learning Containers. This gives you pre-installed CUDA, cuDNN, PyTorch — saving 10–15 min build time:
```dockerfile
FROM 763104351884.dkr.ecr.ap-south-1.amazonaws.com/pytorch-training:2.1.0-gpu-py310-cu118-ubuntu20.04-sagemaker
```
Or if you want to stay lean for CPU:
```dockerfile
FROM python:3.10-slim    # 3.10 is the sweet spot for PyTorch ecosystem
```

---

### 📊 Code Health Summary

| | Score | Note |
|---|---|---|
| SageMaker integration | ⭐⭐⭐⭐⭐ | Perfect paths, ENTRYPOINT |
| Code structure | ⭐⭐⭐⭐ | Clean modules, one redundant hardcode |
| Training logic | ⭐⭐⭐⭐ | Good but double zero_grad |
| Production readiness | ⭐⭐ | No logging, no class_names save, empty inference.py |
| Notebook code | ⭐⭐⭐ | Duplicate Estimator instantiation |

**Overall: 7/10. Very solid foundation. Fix the 7 issues above and it becomes a 9/10.**

---

## 2. Handling Large GB-scale Datasets

When your dataset grows from MBs to **tens of GBs**, two things break:
1. **S3 → Instance transfer time** (downloading all data before training starts)
2. **RAM/VRAM pressure** (trying to load everything into memory)

### The SageMaker Input Mode Solution

SageMaker gives you **3 input modes**. You currently use the default (`File` mode):

| Mode | How it works | When to use |
|---|---|---|
| `File` (default) | Downloads ALL data to `/opt/ml/input/data/` before training | < 10 GB |
| `FastFile` | Streams files from S3 on-demand, no download wait | 10 GB – 100 GB |
| `Pipe` | Streams data as a Unix pipe (requires TFRecord/RecordIO format) | 100+ GB, legacy |

---

### Changes in Your SageMaker Notebook Code

**Current code:**
```python
train_input = TrainingInput(
    s3_data="s3://maaz-animal-classifier-v3/data/raw/"
)
```

**For large datasets — change to FastFile mode:**
```python
train_input = TrainingInput(
    s3_data="s3://maaz-animal-classifier-v3/data/raw/",
    input_mode="FastFile"          # ← Add this one line
)
```

That's the ONLY change needed in the notebook. SageMaker handles the rest. Your `/opt/ml/input/data/training/` path in your code stays exactly the same — files appear there via streaming.

---

### Changes in Your Local Files

**In [dataset.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/src/dataset.py) — optimize the DataLoader:**

```python
# Current:
train_loader = DataLoader(
    train_dataset,  
    batch_size=batch_size,
    shuffle=True,
)

# For large datasets, add:
train_loader = DataLoader(
    train_dataset,  
    batch_size=batch_size,
    shuffle=True,
    num_workers=4,        # ← Parallel data loading workers (use 2–4 for ml.g5)
    pin_memory=True,      # ← Faster GPU transfer (only when using CUDA)
    prefetch_factor=2,    # ← Pre-load 2 batches ahead
    persistent_workers=True,  # ← Keep workers alive between epochs
)
```

**In [dataset.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/src/dataset.py) — add stronger augmentation for large diverse datasets:**

```python
# Current transform has no augmentation. For large real-world datasets, add:
transform = transforms.Compose([
    transforms.Resize((image_size, image_size)),
    transforms.RandomHorizontalFlip(),      # ← Add augmentation
    transforms.RandomRotation(10),          # ← Add augmentation
    transforms.ColorJitter(brightness=0.2), # ← Add augmentation
    transforms.ToTensor(),
    transforms.Normalize(
        mean=[0.485, 0.456, 0.406], 
        std=[0.229, 0.224, 0.225])
])
```

**In [config.json](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/config.json) — tune batch size for large datasets:**

```json
{
  "training": {
    "epochs": 20,           // ← Increase for more data (was 5)
    "batch_size": 64,       // ← Increase (ml.g5 has 24 GB VRAM, use it)
    "learning_rate": 0.0001,
    "image_size": 224,      // ← Standard ImageNet size (was 128)
    "train_ratio": 0.8,
    "random_seed": 42,
    "model_name": "resnet50" // ← Upgrade to resnet50 for complex data (was resnet18)
  },
  "data": {
    "DATASET_DIR": "/opt/ml/input/data/training",
    "MODEL_DIR": "/opt/ml/model"
  }
}
```

**In your SageMaker notebook — add checkpoint to survive spot interruptions (cost saving for large jobs):**

```python
estimator = Estimator(
    image_uri=image_uri,
    role=role,
    instance_type="ml.g5.xlarge",      
    instance_count=1,
    output_path="s3://maaz-animal-classifier-v3/model-artifacts/",
    sagemaker_session=session,
    base_job_name="animal-classifier",
    # Add these for large dataset jobs:
    use_spot_instances=True,                      # ← Up to 70% cost savings
    max_run=7200,                                  # ← 2 hour max (in seconds)
    max_wait=10800,                                # ← 3 hour wait budget for spot
    checkpoint_s3_uri="s3://maaz-animal-classifier-v3/checkpoints/",
)
```

---

### S3 Data Organization for Large Datasets

Structure your S3 bucket **before** uploading large datasets:

```
s3://maaz-animal-classifier-v3/
├── data/
│   ├── raw/
│   │   ├── cat/
│   │   ├── dog/
│   │   └── elephant/
│   └── processed/       ← Pre-resized images go here (saves compute at runtime)
├── model-artifacts/
└── checkpoints/
```

**Pro tip:** Pre-resize images to your target `image_size` (e.g., 224×224) before uploading to S3. This saves the `transforms.Resize()` happening on every single batch during training — a significant speedup at scale.

---

## 3. CI/CD Pipeline

### The Goal

Every time you push code to GitHub → Docker image gets built → pushed to ECR → SageMaker training job starts automatically. **Zero manual steps.**

### Architecture

```
Developer (local)
    │
    │  git push
    ▼
GitHub Repository
    │
    │  Triggers
    ▼
GitHub Actions Workflow
    ├── Step 1: Checkout code
    ├── Step 2: Configure AWS credentials
    ├── Step 3: Build Docker image (with git SHA tag)
    ├── Step 4: Push to ECR
    └── Step 5: Run SageMaker training job (via boto3)
```

---

### Step-by-Step Setup

#### Step 1 — Create AWS IAM User for GitHub Actions

In AWS Console → IAM → Create a new user `github-actions-ci` with these policies:
- `AmazonEC2ContainerRegistryFullAccess`
- `AmazonSageMakerFullAccess`
- `AmazonS3FullAccess`

Generate **Access Key ID** and **Secret Access Key**. You'll need these next.

---

#### Step 2 — Add Secrets to GitHub Repository

Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these 3 secrets:

| Secret Name | Value |
|---|---|
| `AWS_ACCESS_KEY_ID` | Your IAM user's access key |
| `AWS_SECRET_ACCESS_KEY` | Your IAM user's secret key |
| `AWS_REGION` | `ap-south-1` |

---

#### Step 3 — Create the GitHub Actions Workflow

**Create this file in your repo:**
`.github/workflows/train.yml`

```yaml
name: Train Animal Classifier

on:
  push:
    branches: [main]
    paths:
      - 'training/**'     # Only trigger when training code changes
      - 'config.json'

env:
  ECR_REPO: animal-classifier-train
  AWS_ACCOUNT_ID: 956076205820
  SAGEMAKER_ROLE: arn:aws:iam::956076205820:role/SageMakerExecutionRole

jobs:
  build-and-train:
    runs-on: ubuntu-latest
    
    steps:
      # 1. Get the code
      - name: Checkout Repository
        uses: actions/checkout@v4

      # 2. Set up AWS credentials
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      # 3. Login to ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # 4. Build and push Docker image
      - name: Build, Tag and Push Docker Image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}   # Uses git commit SHA as tag
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG ./training
          docker push $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
          echo "IMAGE_URI=$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG" >> $GITHUB_ENV

      # 5. Run SageMaker Training Job
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install boto3
        run: pip install boto3 sagemaker

      - name: Launch SageMaker Training Job
        env:
          IMAGE_URI: ${{ env.IMAGE_URI }}
          SAGEMAKER_ROLE: ${{ env.SAGEMAKER_ROLE }}
        run: |
          python - <<'EOF'
          import boto3
          import os
          import time

          sm = boto3.client('sagemaker', region_name='ap-south-1')

          job_name = f"animal-classifier-{int(time.time())}"
          image_uri = os.environ['IMAGE_URI']
          role = os.environ['SAGEMAKER_ROLE']

          response = sm.create_training_job(
              TrainingJobName=job_name,
              AlgorithmSpecification={
                  'TrainingImage': image_uri,
                  'TrainingInputMode': 'File'
              },
              RoleArn=role,
              InputDataConfig=[{
                  'ChannelName': 'training',
                  'DataSource': {
                      'S3DataSource': {
                          'S3DataType': 'S3Prefix',
                          'S3Uri': 's3://maaz-animal-classifier-v3/data/raw/',
                          'S3DataDistributionType': 'FullyReplicated'
                      }
                  }
              }],
              OutputDataConfig={
                  'S3OutputPath': 's3://maaz-animal-classifier-v3/model-artifacts/'
              },
              ResourceConfig={
                  'InstanceType': 'ml.g5.xlarge',
                  'InstanceCount': 1,
                  'VolumeSizeInGB': 30
              },
              StoppingCondition={'MaxRuntimeInSeconds': 3600},
          )
          print(f"Training job started: {job_name}")
          EOF
```

---

#### Step 4 — Trigger the Pipeline

```bash
git add .
git commit -m "feat: update model architecture"
git push origin main
```

This triggers the entire pipeline automatically. Go to **GitHub → Actions tab** to watch it run.

---

#### What Happens on Each Push

```
git push
   ↓ (30 sec)
Docker build starts on GitHub's ubuntu runner
   ↓ (5-10 min)
Image tagged with git SHA (e.g., abc1234) pushed to ECR
   ↓ (1 min)
SageMaker training job created via boto3 API
   ↓ (SageMaker provisions GPU instance — 5-10 min)
Your container runs on ml.g5.xlarge
   ↓ (your training loop runs)
best_model.pth saved to s3://maaz-animal-classifier-v3/model-artifacts/<job_name>/output/model.tar.gz
```

---

### Optional: Get Slack/Email Notification When Training Completes

Add this to `train.yml` after the training job step:
```yaml
      - name: Notify Slack on Success
        if: success()
        run: |
          curl -X POST -H 'Content-type: application/json' \
          --data '{"text":"✅ Animal Classifier training job launched successfully!"}' \
          ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## 4. Multiple Training Runs — Model Storage & Tracking

### What SageMaker Actually Creates in S3

Every time you call `estimator.fit()` or the CI/CD pipeline launches a job, SageMaker creates a **unique training job** with a timestamped name like:

```
animal-classifier-2024-01-15-09-23-45-123
```

The output in S3 looks like this:

```
s3://maaz-animal-classifier-v3/
└── model-artifacts/
    ├── animal-classifier-2024-01-15-09-23-45-123/
    │   └── output/
    │       └── model.tar.gz        ← Your best_model.pth + class_names.json zipped
    │
    ├── animal-classifier-2024-01-16-11-00-12-456/
    │   └── output/
    │       └── model.tar.gz        ← Different data, different model
    │
    └── animal-classifier-2024-01-17-14-55-33-789/
        └── output/
            └── model.tar.gz        ← Different hyperparams, different model
```

**Inside each `model.tar.gz`:**
```
best_model.pth
class_names.json    ← (after you add the fix from Topic 1)
```

---

### How to View All Your Training Runs

**Option A — AWS Console:**
1. Go to **SageMaker** → **Training** → **Training jobs**
2. You'll see every job with: status, instance type, duration, start time, end time
3. Click any job → see full logs in CloudWatch, metrics, hyperparameters

**Option B — Programmatically (add this to a notebook cell):**
```python
import boto3
import pandas as pd

sm = boto3.client('sagemaker', region_name='ap-south-1')

# List all your training jobs
response = sm.list_training_jobs(
    NameContains='animal-classifier',
    SortBy='CreationTime',
    SortOrder='Descending',
    MaxResults=20
)

jobs = response['TrainingJobSummaries']
df = pd.DataFrame([{
    'Job Name': j['TrainingJobName'],
    'Status': j['TrainingJobStatus'],
    'Created': j['CreationTime'].strftime('%Y-%m-%d %H:%M'),
    'Duration (min)': round((j.get('TrainingEndTime', j['CreationTime']) - j['CreationTime']).seconds / 60, 1)
} for j in jobs])

print(df.to_string())
```

**Output looks like:**
```
Job Name                                    Status     Created           Duration (min)
animal-classifier-2024-01-17-14-55-33-789  Completed  2024-01-17 14:55  42.3
animal-classifier-2024-01-16-11-00-12-456  Completed  2024-01-16 11:00  38.1
animal-classifier-2024-01-15-09-23-45-123  Failed     2024-01-15 09:23  2.0
```

---

### How to Compare Models Across Runs — SageMaker Experiments

SageMaker Experiments is a built-in **MLflow-like tracker**. Add to your notebook:

```python
from sagemaker.experiments.run import Run

# Wrap your estimator with an experiment run
with Run(
    experiment_name="animal-classifier-experiments",
    run_name=f"run-resnet50-lr001",
    sagemaker_session=session
) as run:
    estimator.fit({"training": train_input})
```

Then in SageMaker Console → **Experiments** → you get a comparison table of all runs with metrics side by side.

---

### How to Know Which Model Was Trained on What

**The key insight:** The training job name IS your version identifier. Each job in S3 has:

1. **Which code** → the Docker image URI (contains git SHA e.g., `7a2e5a9`)
2. **Which data** → the S3 input path logged in the job metadata
3. **Which hyperparameters** → stored in the job description

**Retrieve full details of any job:**
```python
sm = boto3.client('sagemaker', region_name='ap-south-1')

job = sm.describe_training_job(
    TrainingJobName='animal-classifier-2024-01-17-14-55-33-789'
)

print("Image URI:", job['AlgorithmSpecification']['TrainingImage'])
print("Data path:", job['InputDataConfig'][0]['DataSource']['S3DataSource']['S3Uri'])
print("Model path:", job['ModelArtifacts']['S3ModelArtifacts'])
```

---

### How to Download and Compare a Specific Model

```python
import boto3
import tarfile

s3 = boto3.client('s3')

# Download a specific model
s3.download_file(
    'maaz-animal-classifier-v3',
    'model-artifacts/animal-classifier-2024-01-17-14-55-33-789/output/model.tar.gz',
    'model_v3.tar.gz'
)

# Extract it
with tarfile.open('model_v3.tar.gz', 'r:gz') as tar:
    tar.extractall('model_v3/')

# Now you have model_v3/best_model.pth and model_v3/class_names.json
```

---

## 5. Using the Model in a Full-Stack Backend

### The Two Deployment Approaches

| Approach | What it is | Cost | Latency | When to use |
|---|---|---|---|---|
| **SageMaker Real-Time Endpoint** | AWS-managed API that runs your model 24/7 | $$ (per hour) | ~100ms | Production, high traffic |
| **Self-Hosted (Flask/FastAPI)** | You download model, run it on your server | $ (your server cost) | ~50ms | Personal project, full-stack |

For a full-stack project, I recommend **starting with self-hosted** and upgrading to SageMaker endpoint when traffic demands it.

---

### Option A — Self-Hosted FastAPI Backend (Recommended for You)

**Step 1: Download model from S3 in your backend server**

```python
# backend/model_loader.py
import boto3
import tarfile
import os
import json
import torch
from torchvision import transforms
from PIL import Image
import io

# ---- Download model from S3 on server startup ----
def download_model(s3_bucket, s3_key, local_dir="./model"):
    os.makedirs(local_dir, exist_ok=True)
    s3 = boto3.client('s3')
    s3.download_file(s3_bucket, s3_key, f"{local_dir}/model.tar.gz")
    
    with tarfile.open(f"{local_dir}/model.tar.gz", 'r:gz') as tar:
        tar.extractall(local_dir)
    
    print(f"Model extracted to {local_dir}/")
```

**Step 2: Load and run inference**

```python
# backend/inference.py
import torch
import torchvision.models as models
from torchvision import transforms
from PIL import Image
import json

class AnimalClassifier:
    def __init__(self, model_dir="./model"):
        # Load class names
        with open(f"{model_dir}/class_names.json") as f:
            self.class_names = json.load(f)
        
        num_classes = len(self.class_names)
        
        # Recreate the same model architecture as training
        self.model = models.resnet18(pretrained=False)
        self.model.fc = torch.nn.Linear(self.model.fc.in_features, num_classes)
        
        # Load weights
        self.model.load_state_dict(
            torch.load(f"{model_dir}/best_model.pth", map_location="cpu")
        )
        self.model.eval()
        
        # Same preprocessing as training
        self.transform = transforms.Compose([
            transforms.Resize((128, 128)),
            transforms.ToTensor(),
            transforms.Normalize(
                mean=[0.485, 0.456, 0.406],
                std=[0.229, 0.224, 0.225]
            )
        ])
    
    def predict(self, image_bytes: bytes) -> dict:
        """
        Takes raw image bytes, returns predicted class and confidence scores.
        """
        img = Image.open(io.BytesIO(image_bytes)).convert("RGB")
        tensor = self.transform(img).unsqueeze(0)   # Add batch dimension
        
        with torch.no_grad():
            outputs = self.model(tensor)
            probs = torch.softmax(outputs, dim=1)[0]
        
        predicted_idx = probs.argmax().item()
        
        return {
            "predicted_class": self.class_names[predicted_idx],
            "confidence": round(probs[predicted_idx].item(), 4),
            "all_scores": {
                self.class_names[i]: round(probs[i].item(), 4)
                for i in range(len(self.class_names))
            }
        }
```

**Step 3: Expose as FastAPI endpoint**

```python
# backend/main.py
from fastapi import FastAPI, UploadFile, File
from fastapi.middleware.cors import CORSMiddleware
from inference import AnimalClassifier
from model_loader import download_model

app = FastAPI(title="Animal Classifier API")

# Allow your frontend to call this
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # Your React/Next.js app
    allow_methods=["*"],
    allow_headers=["*"],
)

# Load model once at startup
download_model(
    s3_bucket="maaz-animal-classifier-v3",
    s3_key="model-artifacts/animal-classifier-2024-01-17-14-55-33-789/output/model.tar.gz"
)
classifier = AnimalClassifier(model_dir="./model")


@app.post("/predict")
async def predict(file: UploadFile = File(...)):
    """
    Upload an image, get back the predicted animal class.
    
    Example curl:
      curl -X POST "http://localhost:8000/predict" \
           -F "file=@cat.jpg"
    """
    image_bytes = await file.read()
    result = classifier.predict(image_bytes)
    return result


@app.get("/health")
def health():
    return {"status": "ok", "classes": classifier.class_names}
```

**Response from `/predict`:**
```json
{
  "predicted_class": "cat",
  "confidence": 0.9823,
  "all_scores": {
    "cat": 0.9823,
    "dog": 0.0134,
    "elephant": 0.0043
  }
}
```

---

### Option B — SageMaker Real-Time Endpoint (Production)

For this, you first need to fill in your [inference.py](file:///d:/mlops/notes/v4/animal-classifier-mlops/training/inference.py) (currently empty):

```python
# inference.py — SageMaker's standard interface (4 required functions)
import json
import torch
import torchvision.models as models
from torchvision import transforms
from PIL import Image
import io

def model_fn(model_dir):
    """SageMaker calls this to load your model."""
    with open(f"{model_dir}/class_names.json") as f:
        class_names = json.load(f)
    
    model = models.resnet18(pretrained=False)
    model.fc = torch.nn.Linear(model.fc.in_features, len(class_names))
    model.load_state_dict(torch.load(f"{model_dir}/best_model.pth", map_location="cpu"))
    model.eval()
    model.class_names = class_names
    return model

def input_fn(request_body, content_type):
    """SageMaker calls this to deserialize incoming request."""
    if content_type == "application/octet-stream":
        return Image.open(io.BytesIO(request_body)).convert("RGB")
    raise ValueError(f"Unsupported content type: {content_type}")

def predict_fn(image, model):
    """SageMaker calls this to run inference."""
    transform = transforms.Compose([
        transforms.Resize((128, 128)),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
    ])
    tensor = transform(image).unsqueeze(0)
    with torch.no_grad():
        outputs = model(tensor)
        probs = torch.softmax(outputs, dim=1)[0]
    return probs, model.class_names

def output_fn(prediction_output, accept):
    """SageMaker calls this to serialize the response."""
    probs, class_names = prediction_output
    predicted_idx = probs.argmax().item()
    result = {
        "predicted_class": class_names[predicted_idx],
        "confidence": round(probs[predicted_idx].item(), 4)
    }
    return json.dumps(result), "application/json"
```

**Then deploy the endpoint from your notebook:**
```python
from sagemaker.pytorch import PyTorchModel

model = PyTorchModel(
    model_data="s3://maaz-animal-classifier-v3/model-artifacts/.../output/model.tar.gz",
    role=role,
    framework_version="2.1",
    py_version="py310",
    entry_point="inference.py"
)

predictor = model.deploy(
    initial_instance_count=1,
    instance_type="ml.t2.medium"    # CPU is fine for inference
)
```

**Call it from your backend:**
```python
import boto3

runtime = boto3.client('sagemaker-runtime', region_name='ap-south-1')

with open("test_image.jpg", "rb") as f:
    response = runtime.invoke_endpoint(
        EndpointName="your-endpoint-name",
        ContentType="application/octet-stream",
        Body=f.read()
    )

result = json.loads(response['Body'].read())
print(result)  # {"predicted_class": "cat", "confidence": 0.98}
```

---

## 6. Data Versioning

### Why You Need It

Without data versioning:
- Model trained on dataset v1 performs differently than v2, but you don't know why
- You can't reproduce an old experiment because the data has changed
- You can't audit what data a production model was trained on

### The Options Compared

| Tool | What it is | Best for |
|---|---|---|
| **DVC** | Git for datasets. Tracks data in S3/GCS, versions with git | ✅ Your setup (S3 + Git) |
| **AWS Data Versioning (S3 Versioning)** | S3 built-in feature. Every S3 upload keeps old version. | Simple. No extra tool. |
| **Weights & Biases Artifacts** | Part of W&B MLOps platform. Tracks data + models | When you also use W&B for experiment tracking |
| **LakeFormation** | AWS enterprise data catalog | Enterprise, overkill for your project |

**My recommendation for your project: DVC + S3 (industry standard, free, works with your existing stack)**

---

### Setting Up DVC with S3 (Step-by-Step)

#### Step 1 — Install DVC

```bash
pip install "dvc[s3]"
```

#### Step 2 — Initialize DVC in your project

```bash
# In your training/ directory
git init    # if not already done
dvc init
```

This creates a `.dvc/` folder. Commit it:
```bash
git add .dvc/ .dvcignore
git commit -m "chore: initialize DVC"
```

#### Step 3 — Configure S3 as DVC remote storage

```bash
dvc remote add -d myremote s3://maaz-animal-classifier-v3/dvc-store
```

#### Step 4 — Track your dataset

```bash
# Instead of having data locally and manually uploading to S3,
# let DVC manage it:
dvc add data/raw/

# This creates data/raw.dvc — a tiny text file with a hash:
# outs:
# - md5: d41d8cd98f00b204e9800998ecf8427e
#   size: 2150000000
#   path: raw
```

Commit the `.dvc` file (not the actual data):
```bash
git add data/raw.dvc data/.gitignore
git commit -m "data: add animal dataset v1"
git tag "data-v1"   # ← Tag = version name!
```

#### Step 5 — Push data to S3

```bash
dvc push     # Uploads data to s3://maaz-animal-classifier-v3/dvc-store
```

#### Step 6 — When data changes (new animals added)

```bash
# Add new images to data/raw/
dvc add data/raw/
git add data/raw.dvc
git commit -m "data: add 500 new dog images — dataset v2"
git tag "data-v2"
dvc push
```

#### Step 7 — Reproduce any experiment exactly

```bash
# Go back to data used in an old experiment:
git checkout data-v1
dvc pull     # Downloads the exact dataset from that tag

# Now train — you'll get the same model as v1
```

---

### Connect DVC to CI/CD

In your `train.yml` GitHub Actions workflow, add before the SageMaker step:

```yaml
      - name: Pull dataset from DVC
        run: |
          pip install "dvc[s3]"
          dvc pull                    # Downloads current data version from S3
```

This ensures each training run uses the **exact dataset version** that was tracked at the time of the commit.

---

### DVC + S3 Versioning Together (Belt + Suspenders)

Enable **S3 bucket versioning** as a safety net:

1. AWS Console → S3 → `maaz-animal-classifier-v3` → Properties → **Bucket Versioning** → **Enable**

This means every overwrite of a file in S3 keeps the old version. You can restore any file to any point in time. Think of it as your backup plan if DVC metadata gets corrupted.

---

### The Full Picture: What to Version and Where

| Artifact | Tool | Storage |
|---|---|---|
| **Code** | Git | GitHub |
| **Data** | DVC | S3 (`dvc-store/`) |
| **Docker Images** | ECR tags (git SHA) | ECR |
| **Model weights** | SageMaker job name | S3 (`model-artifacts/`) |
| **Experiments/metrics** | SageMaker Experiments | SageMaker |
| **Config/hyperparams** | Git (config.json) | GitHub |

Everything is versioned. Everything is reproducible. This is what "production MLOps" looks like.

---

## 🏁 Final Checklist — What to Do This Week

| Priority | Task |
|---|---|
| 🔴 High | Fix the 7 code issues from Topic 1 |
| 🔴 High | Save `class_names.json` alongside model weights |
| 🔴 High | Fill in `inference.py` (currently empty) |
| 🟡 Medium | Add `num_workers=4, pin_memory=True` to DataLoaders |
| 🟡 Medium | Set up GitHub Actions CI/CD workflow (`.github/workflows/train.yml`) |
| 🟡 Medium | Build the FastAPI backend for your full-stack app |
| 🟢 Low | Set up DVC for data versioning |
| 🟢 Low | Enable S3 bucket versioning |
| 🟢 Low | Explore SageMaker Experiments for metric tracking |
