'''
KNOWDLENGE

ubuntu@ip-172-31-1-199:~$ ls -ld /tmp
drwxrwxrwt 13 root root 45056 Jun 23 12:33 /tmp
ubuntu@ip-172-31-1-199:~$ sudo rm -rf /tmp
ubuntu@ip-172-31-1-199:~$ ls -ld /tmp
ls: cannot access '/tmp': No such file or directory


'''








SOME IMPORTENT CONCEPT
# ###############################################################################################
'''
IDENTITY
---------
Laptop:
aws configure

EC2:
IAM Role

GitHub Runner:
GitHub Secrets
(or later OIDC).



'''




STEPS
# ###############################################################################################

1. create folder and write your code 
a. train b. inference

'''
│
├───inference
│   │   app.py
│   │   config.json
│   │   requirements.txt
│   │
│   └───src
│           artifact_downloader.py
│           image_transform.py
│           model_loader.py
│           predictor.py
│
└───training
    │   config.json
    │   requirements.txt
    │   train.py
    │
    └───src
            artifact_manager.py
            dataset_loader.py
            data_ingestion.py
            evaluator.py
            trainer.py

'''

2. create a github repository and push the project 


================================================================================================
When you do:
git push origin main

Your code goes to: GitHub Repository

Then GitHub sees:

# .github/workflows/

and finds: deploy.yml

GitHub says:
-------------------------------
A push happened on main branch.
I need to run deploy.ymla


Mentor Question 

Can GitHub itself run -

a. docker build
b. docker push

??????????

Not exactly.

GitHub Repository is just storage.

Like: Google Drive for code

It cannot execute commands.

Real Answer

GitHub starts a temporary machine called a:

GitHub Actions Runner

Architecture:

Your Laptop
      ↓
git push
      ↓
GitHub Repository
      ↓
GitHub Actions Triggered
      ↓
Temporary Linux VM (Runner)
      ↓
docker build
      ↓
docker push
      ↓
ECR

#later OIDC ( to connect with aws with github
)
================================================================================================


3. create : .github/workflows/hello.yml

=================================
CODE
----

===name: Hello Workflow

on:
  push:
    branches:
      - main

jobs:
  hello-job:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Print Hello
        run: echo "Hello Maaz, GitHub Actions is working!"

      - name: Show Current Directory
        run: pwd

      - name: List Files
        run: ls -la

      - name: Show Python Version
        run: python --version

==============================



4. then run 
git add .
git commit -m "Added first GitHub Action"
git push origin main


5. now you will see the ci running in github-Action

'''
Your Laptop
      ↓
git push
      ↓
GitHub Repository
      ↓
GitHub Creates Temporary Ubuntu VM
      ↓
Repository Downloaded
      ↓
Workflow Executed
      ↓
VM Destroyed

'''
---------------------------------------------------------------------------

currently- ci/cd ---> hello-world
later    - ci/cd ---> create docker image using Dockerfile 

but where does the image will store ?
      of-course in git-hub runner not in your local machine 
      remember git-hub runner is running ci/cd not your local machine

---------------------------------------------------------------------------



6. configure secrets in github action
'''
Go to your repository:

Settings
↓
Secrets and Variables
↓
Actions

Create:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

AWS_REGION (optional)

using the same IAM user credentials you used previously with:

aws configure
'''

7. create ECR respository : animal-classifier-inference

8. create build-and-push.yml 
'''
it is inference ci/cd we are testing with this one 
letter we will create 
      /training.yml
      /inference.yml 

=====================================================================================
CODE build-and-push.yml
-----------------------
name: Build And Push To ECR

on:
  push:
    branches:
      - main

env:
  AWS_REGION: ap-south-1
  AWS_ACCOUNT_ID: 956076205820
  ECR_REPOSITORY: animal-classifier-inference

jobs:

  build-and-push:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login To ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build Docker Image
        run: |
          docker build \
          -t $ECR_REPOSITORY:v1 \
          ./inference

      - name: Tag Docker Image
        run: |
          docker tag \
          $ECR_REPOSITORY:v1 \
          $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:v1

      - name: Push Docker Image
        run: |
          docker push \
          $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:v1

'''


9. git push 
( it will start pipeline )

'''
HOW EC2 get image
========================================

GitHub Runner
      ↓
SSH
      ↓
EC2
      ↓
docker pull
      ↓
docker stop
      ↓
docker rm
      ↓
docker run


===========================================
FOR TRAINING WE CAN USE

EC2 Public IP
EC2 Username
Private Key (.pem)

to connect git-hub with EC2


===========================================
for learning purpose we are using SHA key for image tag versioning but in industry we use different things like
----------------------------------------------------------------------------------------------------------------
CodeDeploy
ECS
Kubernetes
ArgoCD

'''


10. now time for CD

11. put these secrets in github secrets

EC2_HOST                : 13.201.134.246
EC2_USER                : ubuntu
EC2_SSH_PRIVATE_KEY     : open the .pem file in text and copy entire content

======================================================================================
'''
Suppose:

Container A

is currently serving traffic on:

Port 8000

GitHub deploys a new image and executes:

docker stop inference-api

then

docker rm inference-api

then

docker run ...

For a few seconds:

No Container Running

API unavailable.

This is called:

Downtime

For our learning project:

Downtime = Acceptable

But what do you think large companies like Amazon, Netflix, Uber do?
Start new container first, verify healthy, then switch traffic


But Real Companies Do
Current Container (v1)
        ↓
Serving Traffic

New Container (v2)
        ↓
Start
        ↓
Health Check
        ↓
Ready ?
        ↓
Yes
        ↓
Switch Traffic
        ↓
Stop v1

This is called:

Blue-Green Deployment
or
Rolling Deployment

'''

25. add port 8000 to your EC2 
'''
Open Port 8000

Go to:

EC2
↓
Instance
↓
Security
↓
Security Groups
↓
Edit Inbound Rules

Add:

Type: Custom TCP
Port: 8000
Source: My IP

For learning you can temporarily use:

0.0.0.0/0

but understand that means:
Anyone on Internet
can hit your API.
'''

26. connect with EC2
# ssh -v -i D:\mlops\animal-classifier-v3.pem ubuntu@3.7.253.190

27. update packages
# sudo apt update

28. install docker
===============================
CHECK
# docker --version 
'' if already installed dont install again''
================================
# sudo apt install docker.io -y

29. enable docker
# sudo systemctl enable docker
# sudo systemctl start docker

30. add ubuntu user to docker group
# sudo usermod -aG docker ubuntu

31. exit (it will exit you from EC2)

32. again connect to EC2

33. create an IAM role 
give permission of S3 and ECR 

34. attach IAM role to EC2 intance

35. verify iam role get attached or not 
# aws sts get-caller-identity
{
    "UserId": "AROA55GUVE36KTBNB5XOG:i-0d5534a33e9ec4ee8",
    "Account": "956076205820",
    "Arn": "arn:aws:sts::956076205820:assumed-role/maaz-ec2-s3-role/i-0d5534a33e9ec4ee8"
}


35. Verify ECR access:
# aws ecr get-login-password --region ap-south-1



36. login to ECR
# aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 956076205820.dkr.ecr.ap-south-1.amazonaws.com

37. write entire ci/cd code and run

38. verify
'''

==================================================================================================================
Verification Level 1

Check container status:

# docker ps

Expected:
CONTAINER ID   IMAGE      STATUS
xxxxxxxxxx     ...        Up xx seconds

If you see:
Up 10 seconds

that's a good sign.


==================================================================================================================
Verification Level 2

Check logs:

# docker logs inference-api

Expected:

INFO: Started server process
INFO: Waiting for application startup.
INFO: Application startup complete.
INFO: Uvicorn running on http://0.0.0.0:8000

This is actually more reliable than docker ps.

Because sometimes:

Container Started
↓
Application Crashed
↓
Container Exited

==================================================================================================================
Verification Level 3

Test endpoint

Your idea:

http://<public-ip>:8000/docs

✅ Correct.

Example:

http://13.201.134.246:8000/docs

If Swagger UI opens:

FastAPI Running

==================================================================================================================
Verification Level 4 (Production Mindset)

Instead of opening /docs, hit an API endpoint:

curl http://13.201.134.246:8000/

or

curl http://13.201.134.246:8000/health

(if we create a health endpoint later)

This is what load balancers and monitoring systems do.

One More Important Thing

Before we write the deployment step, ensure your EC2 Security Group has:

Inbound Rules

SSH      22      Your IP
HTTP    8000     0.0.0.0/0

because:

Container Running

does NOT mean:

Browser Can Access It

The Security Group must allow port:

8000
'''


# #############################################################################################################################
actually we have to do train.yml first from github action 
then inference 

for learaning purpose we did inference first as it was easy to implement 

in our cicd : github create a temp virtual container where every code execute except EOF wala code 
that execute in EC2 with SSH configuration using github action secrets key 
# ##############################################################################################################################


==========================================================================================================================
NOW CREATE train.yml
'''
workflow_dispatch
        ↓

Build training image
        ↓

Push to
animal-classifier-train
        ↓

SSH EC2
        ↓

docker pull
animal-classifier-train:<SHA>
        ↓

docker run
animal-classifier-train:<SHA>
        ↓

train.py executes
        ↓

downloads dataset
        ↓

trains model
        ↓

uploads artifacts to S3
        ↓

container exits
'''






==========================================================================
in train yml you use 
-----------------------
on:
  workflow_dispatch:

''
this means you run it from github - action 
not from pushing the code
''

==========================================================================

