# WHAT WE WANT TO DO

1. IAM User

2. MFA

3. S3 Bucket

4. AWS CLI

5. EC2 GPU

6. Understand EC2

7. Connect Using PEM

8. Create IAM Role

9. Upload Dataset To S3

10. EC2 Downloads Dataset

11. Train Model

12. Save Model To S3

'''choti choti baate

###########################################
EC2 Wants To Read S3

EC2
↓
Download Dataset
↓
Read S3

Do we create access keys?

Professional answer: NO

We attach: IAM Role ( we create IAM Role then attach)

###########################################
to see bucket list in s3
aws CLI

# aws s3 ls (ls of bucket)

# aws s3 ls <bucket> (ls of that bucket)

# aws s3 ls <bucket>/<prefix> (ls of that bucket's that object with prefix)

###########################################

.pem file owner could not be the user "maaz"
so that you will not be able to use .pem file to
connect that is why if error come
add user maaz and use .pem file

###########################################

To activate pre-built pytorch environment:

# source /opt/pytorch/bin/activate

##########################################

from ec2 controller if you run

# aws sts get-caller-identity

it will give unable to credentials becasue
no IAM role assign with the EC2

############################################
aws s3 cp
vs
aws s3 sync

#############################################

some commands
ls
cd
pwd
mkdir
cp
mv
cat
nano
find dataset_split -maxdepth 2 -type d

###############################################

we did
. download data from s3 keep in /data folder
. split the data and keep the split data into /split folder

|-----split
|------|------Train/
|------|------Validate/
|------|------Test/
folders
we use shutile to copy from one location to other

# shutil.copy2(src, dst)

as data is very minimal we can keep copy in split
but in real industry we just keep the path location

# What Companies Usually Do

Keep:

images/
├── cat_1.jpg
├── cat_2.jpg
├── dog_1.jpg
└── ...

only once.

Then create:

train.csv
valid.csv
test.csv

Example:

train.csv
image_path,label
images/cat_1.jpg,Cat
images/dog_1.jpg,Dog

valid.csv
image_path,label
images/cat_999.jpg,Cat
images/dog_999.jpg,Dog

test.csv
image_path,label
images/cat_1200.jpg,Cat
images/dog_1200.jpg,Dog
Why This Is Better

Storage:
Images = 500 GB
CSV Files = few MB
instead of:
1000 GB+

##############################################
Verify python syntax

# python -m py_compile train.py

'''

# ############################################################## create IAM user

1. login to root user
2. go to IAM
3. click Users --> create User
4. select region
5. Enter User_name
6. # Enable (provide user access to AWS Managemet Console)
   (without this IAM user cannot login to AWS console)
7. Choose Custom Password
8. attach policies
   # AdministratorAccesss
   (for learning purpose)
9. Create User
10. click to your IAM user

# creating MFA and getting access_key and security_key

11. Go to security Credentials
    you will see :
    Console Sign-in
    Multi-factor Authentication (MFA)
    Access Keys
    Signing Certificates

12. create a MFA
13. create access key
    Go to:

IAM
↓
Users
↓
maaz-DL
↓
Security Credentials
↓
Access Keys
↓
Create Access Key

AWS will ask: Use Case
You may see options like:

CLI
Local Code
Application Running Outside AWS

Choose:

# Command Line Interface (CLI)

14. save access_key and security_key

#######################

Very Important Question
Suppose tomorrow you write:

import boto3
s3 = boto3.client("s3")

How does AWS know:
This code belongs to Maaz?
AWS must authenticate the request.

when code run on: Laptop

AWS cannot type a password.

So AWS provides:

Access Key ID
Secret Access Key

Example:

import boto3
s3 = boto3.client(
"s3",
aws_access_key_id="AKIA...",
aws_secret_access_key="abcd..."
)

#######################

# ###########################################################creating s3 bucket

1.  Login with IAM user
2.  select region (keep similar to IAM)
3.  navigate to s3
4.  click --> create Bucket
5.  Enter Bucket_name
6.  Leave ACLs Disabled (old method)
7.  You'll see "Block All Public Access" checked
    keep it checked  
    (
    Imagine:
    You accidentally upload:

         dataset
         models
         credentials

    and bucket becomes public.
    Anyone on internet could access it.

    For ML projects:
    Private Bucket
    is the standard.

    Therefore:

    Block All Public Access
    ✅ Enabled
    Leave all checkboxes selected.
    )

8.  Bucket Versioning --> Keep: Disabled for now.
    (
    What does it do?
    Suppose:  
     You upload:
    best_model.pth
    today.

         Tomorrow:
         You upload:
         best_model.pth
         again.

         Without versioning:
         Old file lost forever.

         With versioning:
         Version 1
         Version 2
         Version 3

         all retained.

         Industry?
             Production:
             Usually Enabled

    )

9.  Encyption
    you'll see "Server-Side Encryption"
    Usually
    SSE-S3
    is selected
    (Meaning:AWS automatically encrypts files at rest.)
    '''
    Bucket Name:
    maaz-dl-animal-classifier-2026
    (or similar unique name)

Region:
ap-south-1

Object Ownership:
ACLs Disabled

Block Public Access:
Enabled

Versioning:
Disabled

Encryption:
Default (SSE-S3)
''' 10. click --> Create Bucket

# ################################################# AWS CLI + Connect with IAM user

(command Line Interface)

'''
Instead of clicking:

AWS Console
↓
S3
↓
Upload

you can do:

# aws s3 cp file.txt s3://bucket/

(copy from to)
from your local terminal.

'''

1. browse AWS CLI installer for <windows>
2. install
3. open powerShell
   RUN

# aws --version

Excepted : aws-cli/2.x.x 4. aws config
(
aws asked:
Access Key : <paste access key>
Secret Key : <paste secret key>
Region : <paste your IAM user region>
Output Format: <json>
)
'''
Before configuration
if you run

# aws s3 ls

aws cli will say something like

> > Unable to locate credentials

because AWS CLI does not know <Who are you>

after configuration
RUN

# aws s3 ls

2026-06-08 16:39:14 maaz-animal-classifier-01
(your buckets name)

'''

# ################ SOME S3 COMMANDS

see list of all buckets:
aws s3 ls

uploaded to s3:
aws s3 cp dataset/ s3://maaz-animal-classifier-01/dataset/ --recursive

Download From S3:
aws s3 cp s3://maaz-animal-classifier-01/dataset/ dataset/ --recursive

# ############################################################## CREATE EC2 INSTANCE

'''
EC2:

EC2 components

1. CPU
2. RAM
3. EBS (storage just like hardDisk)
4. Network
5. OS

# What Is A Public IP?

Suppose we create: Ubuntu EC2

AWS assigns: 13.235.xxx.xxx

This becomes the machine's address.

Your laptop can then say:
Connect to 13.235.xxx.xxx

# while creating EC2 always allow port 22 at least

# why?

What Is A Security Group?
This is one of the most important AWS concepts.

Think:
Firewall

Imagine your house.
You have:
Front Door

Question: who can enter?

Everyone?
Nobody?
Only invited guests?

Security Group answers that.

Example:
Allow SSH
Port 22

means:

Let me login remotely
Without this:
SSH won't work
even if EC2 exists.

---------------- <Knowledge Check> ------------

Suppose tomorrow:
EC2 Exists

but Security Group does NOT allow: Port 22 (SSH)

Question:

Will this command work?

ssh -i my-key.pem ubuntu@13.xxx.xxx.xxx

A) Yes
B) NO

Answer is B) No
port 22 allow SSH login using .pem

<some importent ports>

Port 22 → SSH
Port 80 → HTTP Website
Port 443 → HTTPS Website
Port 8000 → FastAPI
Port 8888 → Jupyter

# What is A Key Pair

When EC2 is created:

AWS generates:
Public Key
Private Key

You download: Private Key (.pem)

Later:

ssh -i my-key.pem ubuntu@IP

AWS verifies:
Correct Key?

If yes:
Login Allowed

'''

# creating EC2

AWS asks:

1. Name
   ''''
   name of your instance
   '''

2. Operating System (AMI) (Amazon Machine Image)
   '''

# Preinstalled Operating System + Optional Software

like:
windows ISO 'or'
ubunti ISO 'or'
linex Image 'or' +
Optional Software

'''

3. Instance Type (processor)
   '''

AMI different families:

# T Family : tiny , cheap , general purpose

# C family : compute optimized more CPU power

# R family : RAM opotimized

# G family : graphic / GPU

# P family : heavy GPU Training

'''

4. Key Pair
   '''

Suppose:

Ubuntu EC2 --> gets created.

Question:

How will AWS know: You are the owner?

AWS could use:

Username + Password

but that's less secure.

Instead AWS uses:

Public Key Cryptography

# AWS creates two keys

Public Key
Private Key

--- AWS stores public key inside EC2
--- you donwlod private key (.pem) to your laptop
--- leter use this .pem file to connect (remote control)
EC2 instance from your local

# ssh -v -i D:\mlops\maaz-dl-key.pem ubuntu@3.109.183.73

'''

5. Network / Security Group
   '''
   Allow the port you will access from your EC2

'''

6. Storage (EBS)
   '''
   select the size of hard disk
   '''

# create EC2 instance

1. go to EC2
2. give a name
3. for AMI click on search more and search
   <Deep Learning AMi> # AMI (Amazon machine Image) == OS + software require (additional software preinstalled)
   see all the option you get and select any one
   ( we select ubuntu server 24.04 LTS)
4. select an instance (processor)
   (we selected m7i0flex.large 2 vCPU 8gb RAM)
   '''
   Some time aws does not allow to use large instance (GPU)
   by defalut you have to request for that

a. aws search Service Quotas
b. Open it
c. choose Amazon Elastic compute Cloud (Amazon EC2)
d. search for
Running on-Demand G and VT instance
'or'
G

# you will be able to see your quotas of GPU

![alt text](image.png)

if
Applied account-level quota value = 0
AWS default quota value = 0

then it means
AWS is saying:
Maximum G-family vCPUs allowed = 0

e. click --> Request increase at account level

For your learning project:
1 g4dn.xlarge

is enough.

g4dn.xlarge uses: 4 vCPUs

Therefore we only need quota: 4

'''

5. Key pair (login) creation

   a. click create new key pair
   b. give a name of key-pair
   c. choose RSA (work everywhere)
   d. private key format choose .pem
   e. click create
   f. downlaod .pem in your local

6. Network settings
   '''
   Auto-assign public IP : enable
   (without this your laptop will not be able to connect
   with your ec2)

for our project:
allow access from any where : allow
(other can login to our ec2 but need .pem that only we have)

HTTP : disable
HTTPS : disable

'''

7. configure storage
   '''
   ![alt text](image-1.png)

30 gb is enouth and

''' 8. file System
None

---

Pre-Launch Checklist

Let's verify:

Name
maaz-dl-classification
✅

AMI
Deep Learning OSS Nvidia Driver AMI
✅

Instance Type
m7i-flex.large
✅

Key Pair
maaz-dl-key
✅

Security Group
Access from everywhere(how can use) and port access(which port can use)
SSH Enabled
✅

Storage
30 GB gp3
✅

Summary
Number of Instance 1

9. launch Instance
10. go to instance  
     wait for
    a. Running
    b. 3/3 status check passed
    ![alt text](image-2.png)

'''
public ip : we use to connect EC2 with local
private ip : other Ec2 use to connect with this EC2
'''

# ############################################################################# CONNECT WITH EC2

# ssh -i C:\Users\Maaz\.ssh\maaz-dl-key.pem ubuntu@3.109.183.73

# Why username == ubuntu

Remember the AMI we selected : ubuntu 24.04

---

Ubuntu -> ubuntu
Amazon Linux -> ec2-user
CentOS -> centos
Debian -> admin

---

'''

# ssh -i C:\Users\Maaz\.ssh\maaz-dl-key.pem ubuntu@3.109.183.73

The authenticity of host '3.109.183.73'
can't be established.
Are you sure you want to continue connecting (yes/no)?

write : yes + press Enter

You should eventually see something like:
<ubuntu@ip-172-31-44-172:~$>

'or'

# GIVE ERROR

Bad permissions. Try removing permissions for user: NT AUTHORITY\\Authenticated Users (S-1-5-11) on file D:/mlops/maaz-dl-key.pem.

# $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

-------------------------------------- VERY IMPORTENT --------------------------------------

SSH is saying:
WARNING: UNPROTECTED PRIVATE KEY FILE!

Why?
Because Windows sees:
D:\mlops\maaz-dl-key.pem

and thinks:
Many users on this computer can read this file.
SSH refuses to use such a key because:

# Private Key

Secret

# ------------- SOLUTION ------------

Method:
Method 1: Using File Properties (Recommended)
Right-click D:\mlops\maaz-classifier-train-v2.pem
Click Properties
Go to Security tab
Click Advanced
Click Disable inheritance
Choose Remove all inherited permissions
Click Add → Select a principal
Enter your Windows username and click Check Names
Grant Full Control
Remove entries such as:
Authenticated Users
Users
Everyone
Click Apply → OK

![alt text](image-3.png)

. write click .pem file
. properties
. open security
. click advance
. click disable inheritance
. Convert inherited permissions into explicit permissions ------> choose: Convert

# ############################################################## then write these commands

1. icacls "D:\mlops\maaz-dl-key.pem" /remove "NT AUTHORITY\Authenticated Users"

2. icacls "D:\mlops\maaz-dl-key.pem" /remove "BUILTIN\Users"

3. icacls "D:\mlops\maaz-dl-key.pem" /grant:r "$env:USERNAME:(F)"

. in advance you will see <Owner> in top
. in permission add ---> select a peinciple --> enter --> Maaz (DESKTOP-TMSIV79\Maaz) --> ok ---> grant (full control) --> ok --> apply

---

now try:

# ssh -i C:\Users\Maaz\.ssh\maaz-dl-key.pem ubuntu@3.109.183.73

Congratulation you connected now!!!

# write these commands

. whoami
. pwd
. ls
. ls -la
. python --version
. python -c "import torch; print(torch.**version**)"
. python -c "import torch; print(torch.cuda.is_available())"

'''

# connect with virual environemt

-- check virtual environemnt name
check

# ls /opt

then check each folder

# ls /opt/pytorch

whichever has bin lib pyenv.cfg
like in our case pytorch
that will be our venv env

-- To activate pre-built pytorch environment:

# source /opt/pytorch/bin/activate

# ############################################################## ATTACH IAM ROLE TO EC2

IAM user : authentication
IAM role : authorization

from EC2

# aws sts get-caller-identity

it will give unable to credentials becasue
no IAM role assign with the EC2
( we have to attach IAM role of our IAM user to EC2)
'''
Currently:

Maaz (IAM User)
│
├── S3 Bucket
│
└── EC2 Instance

But EC2 itself has:
IAM Role = None

Therefore:
aws sts get-caller-identity

returns:
Unable to locate credentials

because EC2 has no identity.

---

What We Are Going To Build
EC2
│
▼
IAM Role
│
▼
S3 Permission
│
▼
S3 Bucket

Now EC2 gets its own AWS identity.

'''

# step to create and attach

# create

0. login to IAM user
1. go to IAM user
2. go to ROLE
   ![alt text](image-4.png)
3. click create role
4. Trusted entity type (Who will assume (use) this role)
   select : EC2
   '''
   AWS Service : because EC2 is an AWS service.
   '''
5. use case
   Select EC2
6. next
7. permission
   select : AmazonS3FullAccess
8. role name and description
   '''
   name : maaz-ec2-s3-role
   Description: Allow EC2 instances to access S3 for ML training
   '''
   ![alt text](image-5.png)
9. click on create

---

# Attaching

'''
EC2
↓
maaz-DL-01
↓
Actions
↓
Security
↓
Modify IAM Role
'''

1. go to EC2
2. select intance
3. click action
4. click security
5. modify IAM role
6. select role that you created (maaz-ec3-s3-role)
7. click update IAM role
8. wait 80 to 90 sec to update role

---

# checking

# 1. aws sts get-caller-identity

output :
{
"UserId": "...",
"Account": "956076205820",
"Arn": "arn:aws:sts::956076205820:assumed-role/maaz-ec2-s3-role/..."
}
AWS automatically generates temporary credentials
and EC2 can authenticate itself

'''
Need Credentials?
↓
Check ~/.aws/credentials
↓
Not Found
↓
Check EC2 Metadata Service
↓
Found IAM Role
↓
Request Temporary Credentials
↓
Call STS
↓
Success
'''

# 2. aws s3 ls

output : list of all s3 bucket

# ############################################################## UPLOAD DATASET TO S3

'''

# cp vs sync

There are two commands people confuse:

aws s3 cp

Think:
Copy

Example:
aws s3 cp cat.jpg s3://bucket/
One file copied.

aws s3 sync

Think:
Mirror folders

Example:
Local Folder
↓
S3 Folder

If:
Cat/
Dog/

already exists in S3,

sync uploads only new/changed files.

This is why ML engineers use it constantly.

---

Example:
Real Industry Example

Imagine:

dataset/
├── 50,000 images

Tomorrow you add:

100 new images

With:
aws s3 cp

you may upload everything again.

With:
aws s3 sync

AWS uploads only:
100 new images

Much faster.

'''

## <from cli>

# aws s3 ls

output : list of buckets
ex : maaz-animal-classfifier-01

# upload: data from your local to s3

1. create an object string ( like a empty folder)

# aws s3 cp NUL s3://maaz-animal-classifier-01/data/.keep

2. write to upload
   aws s3 sync "YOUR_DATASET_PATH" s3://maaz-animal-classifier-01/data/raw/
   ex:

# aws s3 sync "D:\mlops\classification_deep-learning\dataset" s3://maaz-animal-classifier-01/data/raw/

3. after upload check

# aws s3 ls s3://maaz-animal-classifier-01/data/raw/ --recursive

# ############################################################## LETS TRAIN OUR MODEL

# <EC2> (obviously)

## Downlaod Data

1. create directory

# mkdir -p ~/animal-classifier

2. change directory

# cd ~/animal-classifier

3. copy data from s3 to EC2 (EBS)

# aws s3 sync s3://maaz-animal-classifier-01/data/raw ./dataset

(it will create a /dataset folder inside EBS)

4. verify the download

# find dataset | head -20

5. verify dataset size

# du -sh dataset

6. verify class count

# find dataset/Cat -type f | wc -l

# find dataset/Dog -type f | wc -l

## Split Data

1. # nano split_dataset.py

   (it will create file and open so you can write the code)

2. # code
   import os
   import random
   import shutil
   from pathlib import Path

random.seed(42)

SOURCE_DIR = "dataset"
OUTPUT_DIR = "dataset_split"

TRAIN_RATIO = 0.70
VALID_RATIO = 0.20
TEST_RATIO = 0.10

def create_dir(path):
os.makedirs(path, exist_ok=True)

classes = os.listdir(SOURCE_DIR)

for class_name in classes:

    source_class_dir = os.path.join(SOURCE_DIR, class_name)

    if not os.path.isdir(source_class_dir):
        continue

    images = os.listdir(source_class_dir)

    random.shuffle(images)

    total = len(images)

    train_end = int(total * TRAIN_RATIO)
    valid_end = train_end + int(total * VALID_RATIO)

    train_files = images[:train_end]
    valid_files = images[train_end:valid_end]
    test_files = images[valid_end:]

    for split_name, split_files in [
        ("train", train_files),
        ("valid", valid_files),
        ("test", test_files),
    ]:

        target_dir = os.path.join(
            OUTPUT_DIR,
            split_name,
            class_name
        )

        create_dir(target_dir)

        for file_name in split_files:

            src = os.path.join(
                source_class_dir,
                file_name
            )

            dst = os.path.join(
                target_dir,
                file_name
            )

            shutil.copy2(src, dst)

print("Dataset split completed.")

3. CTRL + O
4. Enter
5. CTRL + X
6. RUN

# python split_dataset.py

7. verify

# find dataset_split/train/Cat -type f | wc -l

# find dataset_split/valid/Cat -type f | wc -l

# find dataset_split/test/Cat -type f | wc -l

## train model

'''
Importent Codes to understand train.py code
===============================

1.  from torchvision import datasets

train_dataset = datasets.ImageFolder(
"dataset_split/train"
)

print(train_dataset.classes)

print(train_dataset.class_to_idx)

> > output :
> > 2
> > {

    "cat" : 0,
    "Dog" : 1

}

'''

---

## train_code

1. import
2. config
3. create tranformer
4. create data_set
   with :
   ImageFolder (image_path + transformer)
5. create dataLoader
   with :
   DataLoader (data_set + batch_size)

6. DATALOADER TEST

images, labels = next(
iter(train_loader)
)

print("\nImage Batch Shape:")
print(images.shape)

print("\nLabel Batch Shape:")
print(labels.shape)

print("\nFirst 10 Labels:")
print(labels[:10])

7. download/load model
8. optimize sequantial layer as per your data

# ----------------------------------------------

<very Importent>
'''
Sequential(
  (0): Dropout(p=0.2, inplace=True)
  (1): Linear(
        in_features=1280,
        out_features=1000,
        bias=True
      )
)
'''

# -----------------------------------------------

9. create object of loss as per your data
10. create optimizer with your config learning_rate
11. RUN training epoch loop
    i. model.train()
    A. Loop on images, labels ---> train*loader:
    a. put image and labels on device
    b. make optimizer ZERO
    c. outputs = model(images)
    d. find loss (outputs,labels)
    e. back_propagation --> loss.backwar()
    f. optimize weight --> optimizer.step()
    g. predict -->
    *, preds = torch.max(
    outputs,
    dim=1
    )
    ii. model.eval()
    A. with torch.no*grad():
    A. Loop on images,labels ---> valid_loader:
    a. put image and labels on device
    b. make optimizer ZERO
    c. outputs = model(images)
    d. find loss (outputs,labels)
    e. back_propagation --> loss.backwar()
    f. optimize weight --> optimizer.step()
    g. predict -->
    *, preds = torch.max(
    outputs,
    dim=1
    )

    iii. save model
    if valid_loss < best_valid_loss:
    best_valid_loss = valid_loss
    torch.save(
    model.state_dict(),
    MODEL_SAVE_PATH
    )

    iV. test model
    A. load model and put your saved*state_dict
    model.load_state_dict(
    torch.load(
    MODEL_SAVE_PATH,
    map_location=device
    )
    )
    B. model.eval()
    C. with torch.no_grad():
    a. Loop images,labels ---> test_loader:
    ai. put image and labels on device
    aii. ouputs = models(images)
    aiii. predict
    *, preds = torch.max(
    outputs,
    dim=1
    )
    aiv. find correct outputs
    test_correct += (
    preds == labels
    ).sum().item()
    b. find accuracy
    bi. test_total += labels.size(0)
    bii. test_acc = test_correct / test_total

---

0. check packages

# python -c "import torchvision; print(torchvision.**version**)"

# python -c "import torch; print(torch.**version**)"

1. nano train.py
   CODE

---

import json
import os
import time
import copy
import random
import numpy as np

import torch
import torch.nn as nn
import torch.optim as optim

from torchvision import datasets
from torchvision import transforms
from torchvision import models

from torch.utils.data import DataLoader

# =====================================

# CONFIG

# =====================================

BASE_DIR = "/home/ubuntu/animal-classifier"

ARTIFACT_DIR = (
f"{BASE_DIR}/artifacts"
)

TRAIN_DIR = f"{BASE_DIR}/dataset_split/train"

VALID_DIR = f"{BASE_DIR}/dataset_split/valid"

TEST_DIR = f"{BASE_DIR}/dataset_split/test"

IMG_SIZE = 224

BATCH_SIZE = 32

LEARNING_RATE = 0.0001

NUM_EPOCHS = 10

# (OLD ) MODEL_SAVE_PATH = f"{BASE_DIR}/best_model.pth"

os.makedirs(
ARTIFACT_DIR,
exist_ok=True
)

MODEL_SAVE_PATH = (
f"{ARTIFACT_DIR}/best_model.pth"
)

# =====================================

# TRANSFORMS

# =====================================

train_transform = transforms.Compose([

    transforms.Resize((256,256)),

    transforms.RandomResizedCrop(
        IMG_SIZE
    ),

    transforms.RandomHorizontalFlip(),

    transforms.RandomRotation(10),

    transforms.ColorJitter(
        brightness=0.2,
        contrast=0.2,
        saturation=0.2
    ),

    transforms.ToTensor(),

    transforms.Normalize(
        mean=[0.485,0.456,0.406],
        std=[0.229,0.224,0.225]
    )

])

valid_transform = transforms.Compose([

    transforms.Resize(
        (IMG_SIZE, IMG_SIZE)
    ),

    transforms.ToTensor(),

    transforms.Normalize(
        mean=[0.485,0.456,0.406],
        std=[0.229,0.224,0.225]
    )

])

# =====================================

# DATASETS

# =====================================

train_dataset = datasets.ImageFolder(
root=TRAIN_DIR,
transform=train_transform
)

valid_dataset = datasets.ImageFolder(
root=VALID_DIR,
transform=valid_transform
)

test_dataset = datasets.ImageFolder(
root=TEST_DIR,
transform=valid_transform
)

# =====================================

# DATALOADERS

# =====================================

train_loader = DataLoader(
train_dataset,
batch_size=BATCH_SIZE,
shuffle=True,
num_workers=2
)

valid_loader = DataLoader(
valid_dataset,
batch_size=BATCH_SIZE,
shuffle=False,
num_workers=2
)

test_loader = DataLoader(
test_dataset,
batch_size=BATCH_SIZE,
shuffle=False,
num_workers=2
)

# =====================================

# DEVICE

# =====================================

device = torch.device(
"cuda" if torch.cuda.is_available()
else "cpu"
)

# =====================================

# MODEL

# =====================================

model = models.efficientnet_b0(
weights=models.EfficientNet_B0_Weights.DEFAULT
)

num_features = model.classifier[1].in_features

model.classifier[1] = nn.Linear(
in_features=num_features,
out_features=len(train_dataset.classes)
)

model = model.to(device)

# =====================================

# LOSS FUNCTION

# =====================================

criterion = nn.CrossEntropyLoss()

# =====================================

# OPTIMIZER

# =====================================

optimizer = optim.AdamW(
model.parameters(),
lr=LEARNING_RATE
)

# =====================================

# BEST MODEL TRACKING

# =====================================

best_valid_loss = float("inf")

# =====================================

# TRAINING LOOP

# =====================================

for epoch in range(NUM_EPOCHS):

    print(f"\nEpoch {epoch + 1}/{NUM_EPOCHS}")

    # -------------------------
    # TRAINING
    # -------------------------

    model.train()

    train_loss = 0.0
    train_correct = 0
    train_total = 0

    for images, labels in train_loader:
        print("Batch Loaded")

        images = images.to(device)
        labels = labels.to(device)

        optimizer.zero_grad()

        outputs = model(images)

        loss = criterion(
            outputs,
            labels
        )

        loss.backward()

        optimizer.step()

        train_loss += (
            loss.item() * images.size(0)
        )

        _, preds = torch.max(
            outputs,
            dim=1
        )

        train_correct += (
            preds == labels
        ).sum().item()

        train_total += labels.size(0)

    train_loss = train_loss / train_total
    train_acc = train_correct / train_total

    # -------------------------
    # VALIDATION
    # -------------------------

    model.eval()

    valid_loss = 0.0
    valid_correct = 0
    valid_total = 0

    with torch.no_grad():

        for images, labels in valid_loader:

            images = images.to(device)
            labels = labels.to(device)

            outputs = model(images)

            loss = criterion(
                outputs,
                labels
            )

            valid_loss += (
                loss.item() * images.size(0)
            )

            _, preds = torch.max(
                outputs,
                dim=1
            )

            valid_correct += (
                preds == labels
            ).sum().item()

            valid_total += labels.size(0)

    valid_loss = valid_loss / valid_total
    valid_acc = valid_correct / valid_total

    print(
        f"Train Loss: {train_loss:.4f} | "
        f"Train Acc: {train_acc:.4f}"
    )

    print(
        f"Valid Loss: {valid_loss:.4f} | "
        f"Valid Acc: {valid_acc:.4f}"
    )

    # -------------------------
    # SAVE BEST MODEL
    # -------------------------

    if valid_loss < best_valid_loss:

        best_valid_loss = valid_loss

        torch.save(
            model.state_dict(),
            MODEL_SAVE_PATH
        )

        print("Best model saved.")

# =====================================

# TEST EVALUATION

# =====================================

print("\nLoading Best Model...")

model.load_state_dict(
torch.load(
MODEL_SAVE_PATH,
map_location=device
)
)

model.eval()

test_correct = 0
test_total = 0

with torch.no_grad():

    for images, labels in test_loader:

        images = images.to(device)
        labels = labels.to(device)

        outputs = model(images)

        _, preds = torch.max(
            outputs,
            dim=1
        )

        test_correct += (
            preds == labels
        ).sum().item()

        test_total += labels.size(0)

test_acc = test_correct / test_total

print(
f"\nTest Accuracy: "
f"{test_acc:.4f}"
)

## Save modle to s3

1.

# aws s3 cp best_model.pth s3://maaz-animal-classifier-01/models/

2. verify

# aws s3 ls s3://maaz-animal-classifier-01/models/

## prediction

1. download model from s3

# aws s3 cp s3://maaz-animal-classifier-01/models/best_model.pth ./best_model.pth

2. write code
   CODE

---

import sys

import torch
import torch.nn as nn

from PIL import Image

from torchvision import transforms
from torchvision import models

# =====================================

# CONFIG

# =====================================

MODEL_PATH = "best_model.pth"

IMG_SIZE = 224

CLASS_NAMES = [
"Cat",
"Dog"
]

# =====================================

# DEVICE

# =====================================

device = torch.device(
"cuda" if torch.cuda.is_available()
else "cpu"
)

# =====================================

# IMAGE TRANSFORM

# =====================================

transform = transforms.Compose([
transforms.Resize((IMG_SIZE, IMG_SIZE)),
transforms.ToTensor()
])

# =====================================

# LOAD MODEL

# =====================================

model = models.efficientnet_b0(
weights=None
)

model.classifier[1] = nn.Linear(
in_features=1280,
out_features=2
)

model.load_state_dict(
torch.load(
MODEL_PATH,
map_location=device
)
)

model = model.to(device)

model.eval()

# =====================================

# IMAGE PATH

# =====================================

image_path = sys.argv[1]

# =====================================

# LOAD IMAGE

# =====================================

image = Image.open(
image_path
).convert("RGB")

image = transform(image)

image = image.unsqueeze(0)

image = image.to(device)

# =====================================

# INFERENCE

# =====================================

with torch.no_grad():

    outputs = model(image)

    probabilities = torch.softmax(
        outputs,
        dim=1
    )

    confidence, predicted_idx = torch.max(
        probabilities,
        dim=1
    )

# =====================================

# RESULT

# =====================================

predicted_class = CLASS_NAMES[
predicted_idx.item()
]

confidence_percent = (
confidence.item() \* 100
)

print(
f"\nPredicted Class: "
f"{predicted_class}"
)

print(
f"Confidence: "
f"{confidence_percent:.2f}%"
)

# =====================================

# RESULT

# =====================================

## RUN

python predict.py dataset_split/test/Cat/1234.jpg

## Is Saving modle to s3 is enough

or we shoud save entire artifacts

aritfacts/
│
├── best_model.pth
│
├── class_mapping.json
│
├── config.json
│
└── inference.py

#

## CREATE THIS MANUALLYT

class_mapping:
{
"0": "Cat",
"1": "Dog"
}

config.json
{
"architecture": "efficientnet_b0",

    "img_size": 224,

    "num_classes": 2,

    "inference_transform": {

        "resize": [224,224],

        "to_tensor": true,

        "normalize": {
            "mean": [
                0.485,
                0.456,
                0.406
            ],

            "std": [
                0.229,
                0.224,
                0.225
            ]
        }
    }

}

metadata.json
{
"model_name": "animal_classifier",
"version": "1.0.0",
"framework": "pytorch",
"created_by": "Maaz",
"dataset": "Cat_vs_Dog"
}

## TO DO THIS USING CODE

with open(
f"{ARTIFACT_DIR}/config.json",
"w"
) as f:

    json.dump(
        config,
        f,
        indent=4
    )

# UPLOADING ENTIRE ARTIFACTS IN S3

aws s3 cp \
artifacts/ \
s3://maaz-animal-classifier-01/artifacts/ \
--recursive

# verify

aws s3 ls \
s3://maaz-animal-classifier-01/artifacts/

## prediction from artifacts

1. get artifacts from se
   aws s3 cp \
   s3://maaz-animal-classifier-01/artifacts/ \
   artifacts/ \
   --recursive

2. verify
   find artifacts -type f

3. nano load_config.py
   code

---

import json

CONFIG_PATH = (
"artifacts/config.json"
)

with open(
CONFIG_PATH,
"r"
) as f:

    config = json.load(f)

print("\nConfig Loaded:\n")

print(config)

print(
"\nArchitecture:",
config["architecture"]
)

print(
"Num Classes:",
config["num_classes"]
)

print(
"Resize:",
config["inference_transform"]["resize"]
)

4. python load_config.py

5. create model_loader.py
   code

---

```python
import json

import torch
import torch.nn as nn

from torchvision import models


# =====================================
# PATHS
# =====================================

CONFIG_PATH = (
    "artifacts/config.json"
)

MODEL_PATH = (
    "artifacts/best_model.pth"
)


# =====================================
# LOAD MODEL
# =====================================

def load_model():

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    # -------------------------
    # LOAD CONFIG
    # -------------------------

    with open(
        CONFIG_PATH,
        "r"
    ) as f:

        config = json.load(f)

    architecture = (
        config["architecture"]
    )

    num_classes = (
        config["num_classes"]
    )

    # -------------------------
    # CREATE MODEL
    # -------------------------

    if architecture == "efficientnet_b0":

        model = models.efficientnet_b0(
            weights=None
        )

        num_features = (
            model.classifier[1].in_features
        )

        model.classifier[1] = nn.Linear(
            in_features=num_features,
            out_features=num_classes
        )

    else:

        raise ValueError(
            f"Unsupported architecture: "
            f"{architecture}"
        )

    # -------------------------
    # LOAD WEIGHTS
    # -------------------------

    model.load_state_dict(
        torch.load(
            MODEL_PATH,
            map_location=device
        )
    )

    model = model.to(device)

    model.eval()

    return model


# =====================================
# TEST ENTRY POINT
# =====================================

if __name__ == "__main__":

    model = load_model()

    print(
        "\nModel Loaded Successfully\n"
    )

    print(model)
```

6. transform_loader.py
   code

---

import json

from torchvision import transforms

# =====================================

# CONFIG PATH

# =====================================

CONFIG_PATH = (
"artifacts/config.json"
)

# =====================================

# LOAD TRANSFORM

# =====================================

def load_transform():

    with open(
        CONFIG_PATH,
        "r"
    ) as f:

        config = json.load(f)

    transform_config = (
        config["inference_transform"]
    )

    transform_steps = []

    # -------------------------
    # Resize
    # -------------------------

    if "resize" in transform_config:

        resize_size = (
            transform_config["resize"]
        )

        transform_steps.append(
            transforms.Resize(
                (
                    resize_size[0],
                    resize_size[1]
                )
            )
        )

    # -------------------------
    # ToTensor
    # -------------------------

    if transform_config.get(
        "to_tensor",
        False
    ):

        transform_steps.append(
            transforms.ToTensor()
        )

    transform = transforms.Compose(
        transform_steps
    )

    return transform

# =====================================

# TEST ENTRY POINT

# =====================================

if **name** == "**main**":

    transform = load_transform()

    print(
        "\nTransform Loaded Successfully\n"
    )

    print(transform)

---

## final predict

import json
import sys

import torch

from PIL import Image

from transform_loader import (
load_transform
)

from model_loader import load_model

# =====================================

# LOAD CONFIG

# =====================================

with open(
"artifacts/config.json",
"r"
) as f:

    config = json.load(f)

# =====================================

# LOAD CLASS MAPPING

# =====================================

with open(
"artifacts/class_mapping.json",
"r"
) as f:

    class_mapping = json.load(f)

# =====================================

# DEVICE

# =====================================

device = torch.device(
"cuda"
if torch.cuda.is_available()
else "cpu"
)

# =====================================

# LOAD MODEL

# =====================================

model = load_model()

# =====================================

# LOAD TRANSFORM

# =====================================

transform = load_transform()

# =====================================

# IMAGE PATH

# =====================================

image_path = sys.argv[1]

# =====================================

# LOAD IMAGE

# =====================================

image = Image.open(
image_path
).convert("RGB")

image = transform(image)

image = image.unsqueeze(0)

image = image.to(device)

# =====================================

# PREDICTION

# =====================================

with torch.no_grad():

    outputs = model(image)

    probabilities = torch.softmax(
        outputs,
        dim=1
    )

    confidence, predicted_idx = torch.max(
        probabilities,
        dim=1
    )

# =====================================

# RESULT

# =====================================

predicted_class = class_mapping[
str(
predicted_idx.item()
)
]

confidence_percent = (
confidence.item() \* 100
)

print(
f"\nPredicted Class: "
f"{predicted_class}"
)

print(
f"Confidence: "
f"{confidence_percent:.2f}%"
)
