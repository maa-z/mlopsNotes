WHAT WE WANT TO ACHIEVE
=======================================================================
GitHub
   ↓
GitHub Actions
   ↓
S3 Dataset
   ↓
SageMaker Training Job
   ↓
Model Artifact (.tar.gz)
   ↓
Model Registry
   ↓
Approval
   ↓
SageMaker Endpoint
   ↓
Prediction API
----------------------------------------------------------------------
No EC2
No Docker Pull
No SSH
=======================================================================




=======================================================================

A. SageMaker Studio

Think:

Jupyter Notebook
+
ML IDE
+
AWS Integration

B. SageMaker Training Job

Think:

docker run training-image

but AWS runs it.

C. SageMaker Model

Think:

best_model.pth
+
inference code

registered in AWS.

D. Model Registry

Think:

Git for Models

Example:

Model v1
Model v2
Model v3

with approval workflows.

E. Endpoint

Think:

Managed FastAPI

except AWS hosts it.


=======================================================================

=======================================================================
FLOW

Now The Big SageMaker Idea

AWS asks:

Why are you creating EC2?

Why are you SSHing?

Why are you pulling Docker images manually?

Why are you managing disk space?

Why are you deleting containers?

So SageMaker introduces:

SageMaker Training Job

Instead of:

Create EC2
↓
SSH
↓
Pull Image
↓
Run Container

you simply say:

start_training_job()

AWS internally does:

Create EC2
↓
Pull Container
↓
Run Training
↓
Store Artifact
↓
Terminate EC2

without you touching anything.

=======================================================================


=======================================================================
Let's Map Our  V3    Components
V3	                  SageMaker
EC2	               Managed Compute
Docker Image	      Training Image
train.py	            Training Script
S3 Dataset	         Input Channel
best_model.pth	      Model Artifact
GitHub Action SSH	   Training Job API
=======================================================================


=======================================================================
SageMaker Resources

When you open SageMaker for the first time, you'll see many things:

Studio
Domain
User Profile
Notebook
Training Job
Model
Endpoint
Pipeline
Registry

Looks scary.

But actually:

Everything starts with:

Domain
-------

Think of Domain Like This

Imagine AWS creates a company campus.

Company Campus

Inside the campus:
Employees
Laptops
Projects
Files

exist.

In SageMaker:

Domain

is the campus.

Inside the Domain:

User Profiles

exist.

User Profile
------------

Think:

Maaz

inside SageMaker.

Example:

Domain
└── Maaz

The User Profile gets:


Notebook
Storage
Permissions
Studio Access


Studio
---------
Think:

JupyterLab
+
VS Code
+
AWS Integration

running in the cloud.

Mental Model
Domain
   ↓
User Profile
   ↓
Studio
   ↓
Training Jobs
   ↓
Models
   ↓
Endpoints

=======================================================================


=======================================================================
estimator.fit()

AWS internally performs:

Create Training Job
        ↓
Provision Compute
        ↓
Pull Docker Image
        ↓
Download Dataset
        ↓
Run Training Script
        ↓
Save Model Artifact
        ↓
Upload Artifact To S3
        ↓
Upload Logs To CloudWatch
        ↓
Terminate Compute
=======================================================================


# ###############################################################################################
1. create this resources 
-------------------------
a. SageMaker Domain
b. SageMaker User Profile
c. SageMaker Studio
d. SageMaker Execution Role
----------------------------
'''

i. go to amazon sagemaker AI and clik <create sagemaker domain>

(aws adk for quick setup or organization setup or other setup option)

ii. click 
# other domain setup option

iii. it will create these above things for you

'''

DOMAIN CREATED
--------------

# ###############################################################################################
2. go to domain  user click
# open studio

# ###############################################################################################
3. click on 
# jupyterlab

''' 
you will find no space yet 
# space ?
# something like project 

-------------------------------
SageMaker Domain
     ↓
User Profile
     ↓
JupyterLab Space
     ↓
Compute
     ↓
Kernel
     ↓
Notebook
'''

# ###############################################################################################
4. click on quick start
Quick start

ml.t3.medium • 5 GB • 4 GiB RAM

>> launch now

(it will create a studio compute not training compute)
-----------------------------------------------------
When you clicked:

Quick Start
ml.t3.medium

SageMaker started creating:

JupyterLab Space
       ↓
Studio Compute (ml.t3.medium)
       ↓
Kernel Environment
------------------------------------------------------

'''
Let's connect this to V3

In V3:

Your Laptop
    ↓
VS Code
    ↓
Python Environment

In SageMaker:

Browser
    ↓
JupyterLab
    ↓
Kernel
    ↓
ml.t3.medium Studio Compute

So this machine is basically your cloud laptop.

Extremely Important

This machine:

ml.t3.medium

is NOT the training machine.

Many engineers get confused here.

Example:

estimator.fit()

The notebook runs on:

ml.t3.medium

But SageMaker may create:

ml.m5.large

or

ml.g5.xlarge

for training.

So:

Studio Compute
        ≠
Training Compute
'''



IMPORTENT THINGS TO REMEMBER
===============================================================================================
>>. Domain

We create a domain just like creating a company.

Good analogy.

Think:

AWS Account
    ↓
SageMaker Domain
    ↓
Users
    ↓
Spaces
    ↓
Applications

The Domain is the top-level SageMaker environment.

>>. Quick Setup

You said:

it will create me a execution role and user

Correct.

Quick Setup automatically creates:

Domain
User Profile
Execution Role
Network Configuration
Storage Configuration

instead of making you create them manually.

>>. Studio Space

You said:

I created a Jupyter notebook space

Correct.

A Space is your development environment.

Think:

VS Code Workspace
PyCharm Project
Jupyter Environment

all rolled into one.

>>. Kernel

You said:

like a virtual environment

Excellent.

Kernel ≈ Python Environment

Example:

Kernel A
 ├── torch 2.7
 ├── pandas
 └── boto3

Kernel B
 ├── tensorflow
 └── scikit-learn

Different projects can use different kernels.



>>. Training Job Compute

You said:

it will itself make a compute as per the project requirement

Exactly.

When you launch training:

estimator.fit()

SageMaker creates:

Temporary Training Compute

such as:

ml.m5.large
ml.g4dn.xlarge
ml.p5.48xlarge

depending on your configuration.

Important Corrections
❌ "Space creates a training job"

Not exactly.

A Space only creates Studio Compute.

Space
    ↓
Studio Compute
    ↓
Notebook
    ↓
Kernel

No training job exists yet.

Only when you execute:

estimator.fit()

does SageMaker create:

Training Job
❌ "Studio Container"

Not exactly.

Internally SageMaker uses containers, but for learning purposes think:

Space
      ↓
Studio Compute
      ↓
Notebook
      ↓
Kernel

Do not think:

Space → Training Container

Those are different things.

The Biggest Concept

When you click:

Launch JupyterLab Space

what is actually created?

The answer is:

A dedicated EC2-backed Studio Compute

not:

Training Job

not:

Training Container

not:

Kernel

The kernel runs on that compute.

Full Flow (Correct Version)
Create SageMaker Domain
        ↓
Create User Profile
        ↓
Open Studio
        ↓
Create JupyterLab Space
        ↓
Studio Compute Starts
        ↓
Notebook Opens
        ↓
Kernel Starts
        ↓
Write Training Code
        ↓
estimator.fit()
        ↓
SageMaker Service Receives Request
        ↓
Creates Temporary Training Compute
        ↓
Pulls Training Container
        ↓
Downloads Data From S3
        ↓
Training Runs
        ↓
model.tar.gz Saved To S3
        ↓
Training Compute Deleted
Cost Understanding

You said:

while this training job executes actual AWS credits will be cost

Correct.

But there are actually two separate billable computes:

Studio Compute
JupyterLab Space

You pay while it is running.

Example:

ml.t3.medium

running for 5 hours

↓

charged for 5 hours

Training Compute
Training Job

You pay while training runs.

Example:

ml.g4dn.xlarge

running for 2 hours

↓

charged for 2 hours
================================================================================================




CONCEPT CLEAR
================================================================================================

Scenario

Suppose your JupyterLab Space is:

ml.g4dn.xlarge

This means:

SageMaker Studio
      ↓
JupyterLab Space
      ↓
Studio Compute
      ↓
ml.g4dn.xlarge

This machine is only for:

Writing code
Running notebooks
Testing small snippets
Exploring data
Launching training jobs

Think:

Your Laptop

but inside AWS.

Question

You asked:

If my Space is ml.g4dn.xlarge and training requires a bigger machine, what happens?

Answer:

Training Job does NOT care what Studio Compute is.

Example:

Studio:

ml.g4dn.xlarge

Training Job:

estimator = PyTorch(
    ...
    instance_type="ml.p5.48xlarge"
)

When you run:

estimator.fit()

SageMaker creates:

NEW Compute
ml.p5.48xlarge
Actual Architecture
Studio Space
--------------------------------
ml.g4dn.xlarge
--------------------------------

        |
        |
        | submit job
        ↓

Training Job
--------------------------------
ml.p5.48xlarge
--------------------------------

        |
        ↓

S3

These are TWO different machines.

Think Like This

Your Studio Compute is like:

Manager

Your Training Compute is like:

Worker

Manager says:

Train model using ml.p5.48xlarge

Worker starts.

Manager can go home.

Worker keeps working.

Another Example

Imagine:

Studio Space

ml.t3.medium

Very small machine.

Only:

2 vCPU
4 GB RAM

Yet you can launch:

instance_type="ml.g5.48xlarge"

Training job.

Why?

Because training happens elsewhere.

What Actually Happens Internally

When notebook executes:

estimator.fit()

Notebook sends API request:

CreateTrainingJob

to SageMaker Service.

Something like:

{
  "instance_type": "ml.g5.12xlarge",
  "training_image": "...",
  "input_data": "s3://..."
}

SageMaker Service receives this request.

Then:

Creates new EC2
Pulls container
Downloads data
Runs training
Uploads model
Terminates EC2

Your notebook machine is not doing the training.

Most Important Visualization
                     ┌─────────────┐
                     │ Studio      │
                     │ ml.g4dn.xlarge
                     └──────┬──────┘
                            │
                            │ fit()
                            ▼

                 ┌─────────────────────┐
                 │ SageMaker Service   │
                 └─────────┬───────────┘
                           │
                           ▼

                 ┌─────────────────────┐
                 │ Training Compute    │
                 │ ml.p5.48xlarge      │
                 └─────────┬───────────┘
                           │
                           ▼

                        S3
Your Second Question

When we create space you said EC2 compute created. Is it the same compute where training job executes?

Answer:

No. Completely different.

Studio Compute:

Created when Space starts

Training Compute:

Created when estimator.fit() starts

Different machines.
Different billing.
Different lifecycle.

Real Industry Example

Many ML Engineers use:

Studio:

ml.t3.medium

(~cheap)

because they only write code.

But training uses:

ml.p5.48xlarge

(very expensive GPU)

only during training.

This saves huge cost.

Knowledge check:

Suppose your Studio Space is:

ml.t3.medium

and inside notebook you run:

estimator = PyTorch(
    ...
    instance_type="ml.g5.2xlarge"
)

Which machine actually trains the model?

A)

ml.t3.medium

B)

ml.g5.2xlarge

And why?

B) ml.g5.2xlarge
================================================================================================










# ###############################################################################################
5. now your stuido has been created 
# open it 
'''
to to user and see open studio
'''

# ###############################################################################################
6. now create a space something like project  ( with a simple processor)
(you knwo what it is as we discuss above about this)
'''
wait till it get ready and status == running 
'''

# ###############################################################################################
7. open it 


# ###############################################################################################
8. go create a notebook 
# code

import sagemaker
print(sagemaker)
--------------------------------------------------------------------------------------------

import sagemaker
print(dir(sagemaker))
--------------------------------------------------------------------------------------------


import boto3
sts = boto3.client("sts")
print(sts.get_caller_identity())

>> output 

{'UserId': 'AROA55GUVE36APSJVHJS5:SageMaker', 'Account': '956076205820', 'Arn': 'arn:aws:sts::956076205820:assumed-role/AmazonSageMaker-ExecutionRole-20260624T113932/SageMaker', 'ResponseMetadata': {'RequestId': 'd2ba1e67-77ea-4ad5-b958-8ad135d7dbb3', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': 'd2ba1e67-77ea-4ad5-b958-8ad135d7dbb3', 'x-amz-sts-extended-request-id': 'MTphcC1zb3V0aC0xOlM6MTc4MjI5NjgwMzUyOTpSOlBKMWhUUE1K', 'content-type': 'text/xml', 'content-length': '470', 'date': 'Wed, 24 Jun 2026 10:26:43 GMT'}, 'RetryAttempts': 0}}

see the 
# AmazonSageMaker-ExecutionRole-20260624T113932

so it is the execution role sagem-maker quick start created for your this space 


# ###############################################################################################
9. go to IAM role and find 
# AmazonSageMaker-ExecutionRole-20260624T113932


# ###############################################################################################
10. open this role
>> now add permission 

# AmazonEC2ContainerRegistryFullAccess
# s3fullaccess

'''
Industry Best Practice

We normally DO NOT use:

AmazonS3FullAccess
AmazonEC2ContainerRegistryFullAccess

because they are too broad.

Instead:

Custom IAM Policy

Example:

{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject",
    "s3:ListBucket"
  ],
  "Resource": [
    "arn:aws:s3:::maaz-animal-classifier-v3",
    "arn:aws:s3:::maaz-animal-classifier-v3/*"
  ]
}

Only your bucket.

For our learning project

I would attach:

AmazonS3FullAccess
AmazonEC2ContainerRegistryFullAccess
AmazonSageMakerFullAccess

because we are learning.

Later we will replace them with least-privilege policies.
'''



# ###############################################################################################
11.  you train the /train cicd from github just to push image in ECR 

-------------------------------------------------------------------------------------------------
When you created the SageMaker Domain:

Domain
   ↓
User
   ↓
Execution Role

The Execution Role is not owned by the Space.

The Space merely uses it.

Think:

Execution Role
      ↑
      |
Notebook Kernel
      |
Training Job
      |
Processing Job
      |
Endpoint

Multiple SageMaker resources can use the same role.

What Actually Happens

When you run:

estimator = Estimator(
    image_uri=...,
    role=role
)

and then:

estimator.fit()

SageMaker receives:

CreateTrainingJob
    Image URI
    S3 Dataset
    Execution Role ARN

Example:

Image:
animal-classifier-train

Role:
AmazonSageMaker-ExecutionRole-20260624T113932

Then SageMaker does:

Training Compute Created
         ↓
Assume Execution Role
         ↓
Temporary Credentials
         ↓
Pull Image from ECR
Download Data from S3
Upload Artifacts to S3

Notice something:

Training Compute
      ↓
Execution Role

NOT

Training Compute
      ↓
Studio Space
      ↓
Execution Role

The Studio Space is completely out of the picture now.

Real Industry Mental Model

Think of the role as an ID card.

Execution Role
    = Company ID Card

Notebook:

Notebook gets ID card

Training Job:

Training Job gets same ID card

Endpoint:

Endpoint gets same ID card

They don't borrow the ID card from each other.

They all independently use the same IAM Role.
-------------------------------------------------------------------------------------------------











# ###############################################################################################
# ###############################################################################################
# ###############################################################################################

IMPORTENT READ IN CLEARLY 
-------------------------------------------------------------------------------------------------
What did this code actually show?
import boto3

sts = boto3.client("sts")

print(sts.get_caller_identity())

Output:

arn:aws:sts::956076205820:assumed-role/
AmazonSageMaker-ExecutionRole-20260624T113932/...

This means:

Current Notebook Kernel
           ↓
Currently Using
           ↓
AmazonSageMaker-ExecutionRole-20260624T113932

It only tells us:

"Which role is currently attached to the notebook environment?"

It does NOT mean this is the only role SageMaker can use.

Can Estimator use another IAM role?
YES ✅

You can create:

IAM
  ↓
Roles
  ↓
Create Role

animal-classifier-training-role

Attach:

AmazonS3FullAccess
AmazonEC2ContainerRegistryFullAccess
AmazonSageMakerFullAccess

(or custom policies)

Then use:

from sagemaker.estimator import Estimator

estimator = Estimator(
    image_uri=image_uri,
    role="arn:aws:iam::956076205820:role/animal-classifier-training-role",
    instance_count=1,
    instance_type="ml.g5.2xlarge"
)

Perfectly valid.

Then why do people use get_execution_role()?

Convenience.

Instead of writing:

role = "arn:aws:iam::956076205820:role/animal-classifier-training-role"

they do:

role = get_execution_role()

which automatically fetches the role attached to the notebook environment.

Real Industry Setup

Often you will see:

Notebook Role
sagemaker-studio-role

Permissions:

CreateTrainingJob
CreateEndpoint
Read S3
Training Role
animal-classifier-training-role

Permissions:

Read Dataset S3
Pull ECR Image
Write Model Artifacts S3
Endpoint Role
animal-classifier-endpoint-role

Permissions:

Read Model Artifact
Pull Inference Image
Write CloudWatch Logs

Different jobs, different roles.

What Happens Internally?

Suppose notebook uses:

Role A

and estimator specifies:

Role B

Flow:

Notebook (Role A)
        ↓
CreateTrainingJob
        ↓
Pass Role B ARN
        ↓
Training Compute
        ↓
Assume Role B

Training uses Role B.

Not Role A.

For Our Learning Project

To keep things simple:

Notebook Role
         =
Training Role
         =
AmazonSageMaker-ExecutionRole-20260624T113932

We'll use the same role everywhere initially.

Later I'll show you the production-grade approach with separate roles.

Mini Quiz

Suppose:

Notebook Role:
    Has SageMakerFullAccess

Training Role:
    Has ECR Access
    Has S3 Access

Notebook creates:

estimator.fit()

Question:

Which role needs permission to:

Download dataset from S3?

A)

Notebook Role

B)

Training Role

And which role needs permission to:

Call CreateTrainingJob API?

A)

Notebook Role

B)

Training Role

This distinction is exactly how AWS separates control-plane permissions from data-plane permissions.

training role need permission to download dataset from s3 
and note book role need permission to call createTrainingjob api

🎯 100% Correct.

This is one of the most important IAM concepts in SageMaker.

Control Plane vs Data Plane

AWS internally separates permissions into two categories.

Control Plane

Things that create/manage resources.

Examples:

CreateTrainingJob
StopTrainingJob
CreateEndpoint
DeleteEndpoint
CreateModel

These permissions are needed by:

Notebook Role

because the notebook is making AWS API calls.

Data Plane

Things that actually access data.

Examples:

Download Dataset from S3
Upload Artifacts to S3
Pull Image from ECR
Push Logs to CloudWatch

These permissions are needed by:

Training Role

because the training container is performing those actions.

Complete Flow
Notebook Kernel
      │
      │ CreateTrainingJob API
      ▼
Notebook Role
      │
      ▼
SageMaker Service
      │
      │ Uses Role ARN supplied in estimator
      ▼
Training Compute
      │
      ▼
Training Role
      │
      ├── Pull Image From ECR
      ├── Download Dataset From S3
      ├── Upload Artifacts To S3
      └── Write Logs
Let's Relate This To Your V3

In V3:

EC2 Role

had to do everything:

Create nothing
Pull ECR
Read S3
Write S3
Run Container

because EC2 was both:

Control Plane
+
Data Plane

In SageMaker:

AWS separates responsibilities:

Notebook Role
       ↓
CreateTrainingJob

Training Role
       ↓
Access Data

Much cleaner.
-------------------------------------------------------------------------------------------------








# ###############################################################################################
12. 































START AGAIN
------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------

The notebook is not a trainer.

It is a manager.

Think of the notebook as a project manager.

It says:
---------
Dear SageMaker,

Here is my code.
Here is my Docker image.

Here is my dataset.

Please train the model.

Call me when you're done.

SageMaker replies:
------------------
Okay.

I'll create a separate machine.

I'll train it.

I'll store the model.

Then I'll destroy the machine.

That sentence is the heart of SageMaker.



 
====================================================================================
sage-maker download the dataset so you dont need to downoad from your tarining code

'''
Where should it put it?

Should SageMaker invent a random path every time?

Imagine one training job puts it here:

/tmp/data

Another puts it here:

/home/ubuntu/data

Another here:

/app/dataset

That would be chaos.

So SageMaker says:

"Every training container in the world will use the same folder."

This is why /opt/ml exists

AWS chose one standard location.

/opt/ml/

Inside it:

/opt/ml/

├── input/
│      └── data/
│             └── training/
│                    ├── Cat
│                    └── Dog
│
├── model/
│
└── output/

This is one of the most important directories in SageMaker.

# DATASET_DIR = "/opt/ml/input/data/training"
'''





=============================================================================================================================================
sagemaker estimator parameter
-----------------------------
'''
Parameter 1 — image_uri

Suppose AWS asks you:

"Which code should I run?"

You answer:

956076205820.dkr.ecr.ap-south-1.amazonaws.com/
animal-classifier-train:7a2e5a9

That is

image_uri

Notice.

You're not giving Python files.

You're giving

an entire operating environment.

Inside that image are

train.py

dataset.py

trainer.py

requirements

PyTorch

TorchVision

Everything

AWS doesn't know Python.

It only knows Docker.

Parameter 2 — role

This confused you a few days ago.

You asked:

"Why pass the role again? My notebook already has one."

Excellent question.

Let's revisit it.

Remember this architecture?

Notebook

↓

estimator.fit()

↓

Training Instance

Are these the same machine?

No.

Completely different.

Your notebook dies after sending the request.

The training machine is created later.

How will that future machine access

S3

ECR

CloudWatch

It needs an IAM role.

Therefore

role=

is not for the notebook.

It is for

Future Training Instance
Parameter 3 — instance_type

AWS asks

"What computer should I rent for you?"

You answer

instance_type="ml.g5.xlarge"

or

instance_type="ml.m5.large"

or

instance_type="ml.c5.xlarge"

Notice.

This is not where training happens.

This is only a specification.

Parameter 4 — instance_count

This one is often ignored.

You usually write

instance_count=1

Question.

Can SageMaker train on

instance_count=8

Yes.

Now AWS creates

8 EC2 instances

instead of one.

This is called

Distributed Training

We're not using it today.

But now you know why the parameter exists.

Parameter 5 — output_path

Question.

Where should SageMaker upload

model.tar.gz

after training?

You answer

output_path=
"s3://maaz-animal-classifier-v3/models"

Notice.

Not dataset.

Not image.

Only output.

Parameter 6 — sagemaker_session

This is the most misunderstood parameter.

Many people just write

Session()

without understanding.

Think about this.

Who actually calls AWS APIs?

Not the Estimator.

The Estimator is just a Python object.

It needs someone to communicate with AWS.

That someone is

sagemaker.Session()

Think of it as

AWS Communication Manager

Whenever the SDK wants to

Create Training Job
Upload data
Create Model
Deploy Endpoint

it uses

Session

to talk to SageMaker.

Parameter 7 — base_job_name

Suppose you don't specify it.

AWS creates

pytorch-training-20260628-...

Ugly.

Instead

base_job_name="animal-classifier"

Then every training job becomes

animal-classifier-001

animal-classifier-002

animal-classifier-003

Much cleaner.

Putting it Together

Look at this.

Estimator(

    image_uri=...

    role=...

    instance_type=...

    instance_count=...

    output_path=...

    sagemaker_session=...

    base_job_name=...

)

Read it like English.

"I want SageMaker to use this Docker image, on this compute, with this IAM role, saving results here."

Nothing more.

No infrastructure has been created.
'''
























