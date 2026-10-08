----------------------------------------------

some important coommands:

docker build -t hello-docker . 

docker images 

docker run hello-docker  

docker ps -a  ( see all container)

docker container ls -a

docker rm e23f7e46d1a6

docker run --rm -v D:\mlops\notes\v2\docker-volume-demo:/data python:3.13 cat /data/message.txt

-----------------------------
check docker container 

docker rm <container_id>

docker rmi <image_id> 

aws ecr describe-repositories
# to see the list of all the ecr repositories in aws 
(make sure you have aws CLI configuration )




# DOCKER: 
Question: Can I take your project folder and run it on my laptop?

Answer: Maybe or Maybe not.

Why?

Because my machine may have:

Python 3.10
instead of
Python 3.13

Or:

Torch 2.7
instead of
Torch 2.9

or 

CUDA installed vs No CUDA

or 

ubuntu vs windows


This famous problem is called: 
#### "It Works On My Machine" ####

# Visual

Maaz Laptop

Python 3.13
Torch 2.9
↓
Works

Engineer B Laptop

Python 3.10
Torch 2.6
↓
Fails


# what is docker?

Docker Packages these things!
    |
    V

Application
+
Dependencies
+
Runtime
+
Configuration


Project + Python + Libraries + OS Requirements
                    =
                Docker Image


# Analog

Inside the box:

Python
Torch
Torchvision
Code
Dependencies


Then you give that box to anyone.

Result: Same Environment Everywhere



### Why ML Engineers Love Docker

# Without Docker:

Install Python
Install Torch
Install CUDA
Install Dependencies

Hope It Works

# With Docker:

docker run image


# You must understand:

Dockerfile (Recipe or Instruction) 
↓
Image      (Cake or Built Artifact of DockerFile)
↓
Container  (Running Image or actual object of image)
↓
Registry   (where image lives , docker-hub or ECR)


# Future flow:
train.py
↓
Docker Image
↓
ECR
↓
EC2
↓
Training
↓
Artifacts
↓
S3



########################################################################


# first docker 

. mkdir abc

. cd abc

. touch hello.py

# code
print("hello")

. touch Dockerfile

# code
FROM python:3.13
WORKDIR /app
COPY hello.py .
CMD ["python", "hello.py"] 'or' ENTRYPOINT["python","hello.py"]

. open docker desktop

. navigate to abc

.   docker build -t hello-docker .   
    'or' 
    docker build -t hello-docker:v1 .   (to give a versiom)
(create docker image) 

. docker images 
(it will show all images)

. docker run hello-docker  
or 
docker run hello-docker <command_line_arguments>
(<cmd_args> works only if you use ENTRYPOINT in place of CMD in Dockerfile)

(run container)

. docker ps -a 
or 
.docker container ls -a
(give detail of your conatainer that build when you execute run)

.docker rm e23f7e46d1a6
remove any existing container




# some commands

docker history <image_name>:<tag>
ex
docker history animal-classifier:v1

(it will show the layer how your image created)


# Another Huge Realization

Run:

docker run hello-docker

again.

Docker will create:

ANOTHER CONTAINER

Not reuse the old one.

Result:

docker ps -a

will show:

Container A
Container B

both created from:

hello-docker image






# Volume 

to separate large data from image we use volume

lets for example assume our code use /data 
so we have to put /data into our image 
that will make image size very large 

solution
we create volumen (-v) and bridge our data to our image
so now image does not have our data but still can use 
/data 

another benefits

let assume you create image of training file 
then run the file and create artifacts(contain model <actually not model you know just trained weight>)
and upload the artifacts in s3

another container:
now you create predict container , you have to use 
/artifacts to run predict container will you add the artifacts in image ?
what if you model changed (artifacts change)
so you will not include /artifacts to your image just 
you will use volume to connect a bridge between predict container and s3 artifacts so next time model changed you just need to replace volume location not to change the container Docerfile 

ex:
docker run --rm -v D:\mlops\notes\v2\docker-volume-demo:/data python:3.13 cat /data/message.txt

( docer use python pre-build image and run the container and suddently after execution stop and kill the container -rm , and this container use -v volume even though the /docer-volume-demo was not in python image it was in your local storage , docker crete a bridge between image and your local /docker-volume-demo folder and put everything from your folder to its own storage with name :data and as  your docker-volume-demo has a file name message.txt , now for conatiner it seems like it is in /data folder so it can easily use the file , 
see it use the file cat /data/message.txt)















BASIC DONE
--------------------------------------------------------------------------------------------------------------

------------------------------------------------
                    ┌─────────────┐
                    │   Laptop    │
                    └──────┬──────┘
                           │
                    docker build
                           │
                           ▼
                 ┌──────────────────┐
                 │       ECR        │
                 │ Docker Images    │
                 └────────┬─────────┘
                          │
                    docker pull
                          │
                          ▼
                 ┌──────────────────┐
                 │       EC2        │
                 │ Training Compute │
                 └────────┬─────────┘
                          │
                Read Dataset From
                          │
                          ▼
                 ┌──────────────────┐
                 │        S3        │
                 │    Dataset       │
                 └──────────────────┘

                          │
                    Train Model
                          │
                          ▼

                 ┌──────────────────┐
                 │    Artifacts     │
                 │ best_model.pth   │
                 │ config.json      │
                 └────────┬─────────┘
                          │
                     Upload
                          │
                          ▼
                 ┌──────────────────┐
                 │        S3        │
                 │    Artifacts     │
                 └──────────────────┘


TRAINING:

1. create s3
maaz-animal-classifier-v2

2. create ECR
AWS Console
↓
Elastic Container Registry (ECR)
↓
Repositories
↓
Create Repository

'''
Settings:

Repository Name:
animal-classifier-train

Repository Type:
Private

Everything else:
Default

Create it.
'''

4. craate ECR
(Enter name and keep everything as default)

Registry
│
└── Repository 
      │
      ├── v1
      ├── v2
      └── v3

(
    Repository : classifier_train , classifier_interface etc 
)

5. Create EC2 

6. Create a IAM role and attached 
    a. s3 access
    b. amazonElasticContainerfullAccess

7. connect your CLI with EC2

# now from EC2

8. verify 
a. IAM role : aws sts get-caller-identity     ---> IAM role
b. s3       : aws s3 ls                       ---> list of S3 bucket
c ECR       : aws ecr describe-repositories   ---> list of ECR container


9. connect docker with aws ECR
# from CLI
--------
a. check your caller (it should be IAM user not IAM role as you are using CLI)
# aws sts get-caller-identity
b. connect with docker to aws ECR
# aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 956076205820.dkr.ecr.ap-south-1.amazonaws.com

'''
Step 1

AWS CLI says:

Who are you?
Answer:
IAM User
maaz-DL

because:
aws sts get-caller-identity

returned:
user/maaz-D

Step 2

AWS verifies:
Does maaz-DL have ECR permissions?

Answer:
YES
otherwise login would fail.

Step 3

AWS creates:
Temporary ECR Login Token

Step 4

Docker receives:
Temporary ECR Login Token


Step 5

Docker stores:
Authenticated Session

for:
956076205820.dkr.ecr.ap-south-1.amazonaws.com

Visual:

IAM User
   ↓
AWS CLI
   ↓
ECR Token
   ↓
Docker Login
   ↓
ECR
'''


10. give your image in local docker a tag 
# docker tag animal-classifier:v2 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1
<docker tag <your local image name : with Tag> <ECR:registary>/<ECR:repository>:<cloud Tag>>

'''
Why Tagging Exists
Because Docker needs to know:
Where Should I Push?

Current local name:
animal-classifier:v2

tells Docker:
Local Image

ECR name:
956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1

tells Docker:

Push To This Registry
Push To This Repository
Store As This Tag
 
-----------------------------------------------------
Let's Parse The Full Name

956076205820.dkr.ecr.ap-south-1.amazonaws.com
↓
Registry

animal-classifier-train
↓
Repository

v1
↓
Tag

'''
----------------------------------------------------------------------------------------------------------

'''
not :v1 or :v2 tag you give while creating docker image it is another tag 
something like cloud address to your image 
-----------------------------------------------------

We Need To Give It A Cloud Address

Current:
animal-classifier:v2

Target:
956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1

Notice the hierarchy:

Registry
│
├── 956076205820.dkr.ecr.ap-south-1.amazonaws.com
│
Repository
│
├── animal-classifier-train
│
Tag
│
└── v1
Docker Tag

Question:
Do we rebuild image?
NO

Question:
Do we duplicate 9.5GB?
NO

Docker tags are basically:

Another Name
For The Same Image

Visual:

Image ID

6de1f1ced7dc

├── animal-classifier:v2
└── 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1

Same image.
Two names.

'''

----------------------------------------------------------------------------------------------------------
'''
PS C:\Users\Maaz> docker images
IMAGE                                                                      ID             DISK USAGE   CONTENT SIZE   EXTRA
956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1   6de1f1ced7dc       9.54GB         3.26GB
animal-classifier:v2                                                       6de1f1ced7dc       9.54GB         3.26GB


it seems like there is two image but both pointing two same Image ID one is local name and anther is name for the cloud (Tag)
'''


11. push image to ECR
# docker push 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1


'''
-------------------------------------------------------------------------------------------------------
IMPORTENT PUSHING CONCEPT

ECR stores ALL layers

including:

Debian Layer
Python Layer
Torch Layer
Your Code Layer
Why?

Imagine:

EC2 is completely empty.

Suppose AWS only stored:

predict.py
artifacts/

and not:

Python
Torch
Debian

Question:

How would EC2 know which:

Python Version?
Torch Version?
Linux Packages?

to install?

It can't.

Therefore ECR stores:

Complete Image

which means:

All Layers
Then Why Layers?

This is where Docker becomes clever.

Imagine:

You push:

animal-classifier:v1

Layers:

Layer A → Debian
Layer B → Python
Layer C → Torch
Layer D → Your Code

Later you create:

animal-classifier:v2

and only change:

print("Hello")

to

print("Hello World")

New layers:

Layer A → Debian      (same)
Layer B → Python      (same)
Layer C → Torch       (same)
Layer D → New Code    (changed)

Question:

Should Docker upload:

9.54 GB again?

No.

Docker notices:

Layer A already exists
Layer B already exists
Layer C already exists

and only uploads:

Layer D

This is called:

Layer Deduplication

and

Content Addressable Storage
Real Industry Example

Imagine Netflix has:

100 services

all using:

python:3.13

Would ECR store:

Python Layer

100 times?

No.

It stores the layer once and reuses it.

This Is Why Docker History Matters

Remember:

docker history animal-classifier:v1

showed:

5 GB  → pip install torch
16 MB → COPY . .

When you push:

Docker is essentially pushing those layers.
'''


'''
----------------------------------------------------------------------------------------------------------
PS C:\Users\Maaz> docker push 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1
The push refers to repository [956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train]
c983fd580956: Pushed
c590385ab393: Pushed
ef1b08ddd59d: Pushed
8b365b6ff7b7: Pushed
d4df1fc17fbc: Pushed
7a52e325352a: Pushed
5a3aebbd3f9b: Pushed
2ca884014342: Pushed
442cca122a8b: Pushed
067a6e3736a9: Pushed
f092b553abff: Pushed
v1: digest: sha256:6de1f1ced7dcbf81bd63818a3d30c1a4f64d5b610124ec129137ad9efb46e878 size: 856

digest is finger_print of your image 

if multiple user pull the image if digest is same then it means both has same image
'''

12. Switch to EC2
'''
EC2
↓
Install Docker
↓
Login To ECR
↓
docker pull
↓
docker run
'''

13. update 
# sudo apt update

14. install docker
# sudo apt install -y docker.io

15. verify docker 
# docker ps 'or' sudo docker ps
# docker images 'or' sudo docker images

16. connect EC2 with ECR 
'''
<BUT EC2 HAS PERMISSION OF ECR>
-----------------------------------------------------

EC2 has:

IAM Role
↓
AmazonEC2ContainerRegistryFullAccess

Does that mean Docker is logged in?
NO

Think of it this way:

IAM says:
You ARE ALLOWED

Docker login says:
I AM AUTHENTICATED

Analogy:
You have permission
to enter the office.

does not mean
You already swiped
your badge.

So even on EC2 we will still run:

# ---------------------------------------------------

aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin \
956076205820.dkr.ecr.ap-south-1.amazonaws.com

# ---------------------------------------------------

The difference is:

Laptop:
IAM User credentials

EC2:
IAM Role credentials

Same login process.
Different credential source.


----------------------------------------------------------------------------------------------------------
Earlier on your laptop:

aws ecr get-login-password
 |
 V
used:
IAM User
(maaz-DL)

to generate the token.

Now on EC2:

aws ecr get-login-password
 |
 V
will use:
IAM Role
(maaz-ec2-s3-role)

to generate the token.
Same command.

Different identity source.

This is one of the coolest AWS concepts.

The command is identical:

aws ecr get-login-password

but AWS decides who you are based on: aws sts get-caller-identity
'''

17. first try to pull without login to ECR
# docker pull 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1
'''
it will give error
'''

18. Now login
# aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin \
956076205820.dkr.ecr.ap-south-1.amazonaws.com

19. now try to pull
# docker pull 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1

20. docker images 
(will show the images)

21. then run the image 
'''
as per our code our image need command line arguments image path with image should be in same machine (local or EC2) to run how would we 
run our this image in EC2

# ------------------------------------------------------------------

we will use volume -v



# =============================================================================
READ CLEARFULLY

# Case 1 — File Inside Container

Suppose your Docker image contains:
/app/11.jpg

Then:
docker run image-name /app/11.jpg

works.

Because:

Container
├── predict.py
├── artifacts/
└── 11.jpg

all exist inside the container filesystem.

# Case 2 — File On EC2

Suppose:

EC2
└── /home/ubuntu/11.jpg

and inside container:
/app/
does NOT contain 11.jpg.

Then:
docker run image-name /home/ubuntu/11.jpg

will fail.

Why?
Because the container sees:
Container Filesystem

not:
Host Filesystem
by default.

This Is Why Volumes Exist

Remember:
-v host_path:container_path

# Example:

docker run \
-v /home/ubuntu:/data \
image-name \
/data/11.jpg
(note -v <directory not path>:/data it means you want to docker to user your directory as /data that is why /data/11.jpg)

Now:
EC2
/home/ubuntu/11.jpg
        │
        ▼
Container
/data/11.jpg

The file becomes visible inside the container.


# Mentor Test

Let's use our current image.

Look at the flow:

Laptop
↓
Build Image
↓
Push ECR
↓
Pull EC2

The image contains:

predict.py
artifacts/

but does NOT contain:
11.jpg



Therefore if we run:

docker run \
956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1 \
11.jpg

what happens?

The argument is passed correctly to:

sys.argv[1]

✅

But then Python tries:

Image.open("11.jpg")

Question:
Where will Python look for 11.jpg?

Inside:

Container Filesystem
or
EC2 Filesystem


Answer : inside Container FIleSystem so will give error
'''




22. download cat image from internet to EC2 ---> EBS

a. create a folder test-image
mkdir test-image

b. move to test-image
cd test-iamge

# c. download the image 
wget https://img.magnific.com/free-photo/little-cat-sitting-grass_1150-17019.jpg -O cat.jpg


23. # run the file using downloaded file directory as volume 
docker run \
-v /home/ubuntu/test-images:/data \
956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1 \
/data/cat.jpg


'''
# ----------------------------------------------------------------------------------------
IMPORTENT DOCKER CONCEPT

What Most People Think

Suppose image contains:

/app/artifacts
├── best_model.pth
├── config.json
└── metadata.json

Then they mount:
-v /home/ubuntu/test-images:/app/artifacts

and think:
/app/artifacts
├── best_model.pth
├── config.json
├── metadata.json
└── cat.jpg

This would be:
Merge

But Docker does NOT do that.

What Actually Happens

Docker says:
You mounted something
on top of /app/artifacts

So the original folder becomes hidden.

Visual:

Before:

Container
/app/artifacts
├── best_model.pth
├── config.json
└── metadata.json

After:
-v /home/ubuntu/test-images:/app/artifacts

Container sees:
/app/artifacts
└── cat.jpg

ONLY.
The original files still exist in the image.
But they are invisible because the volume is mounted over them.

Real World Example

Suppose your image contains:

/app/artifacts/best_model.pth

and your prediction code does:
torch.load("/app/artifacts/best_model.pth")

Then you mount:
-v /home/ubuntu/test-images:/app/artifacts

Now inside container:

/app/artifacts
└── cat.jpg

Question:
Where is:

best_model.pth
?

Gone (hidden).

Result:
FileNotFoundError
'''





















DOCKER TRAIN
--------------


practice over now like we do in industry 
-----------------------------------------
'''
Training Container Responsibilities

Our training container should only know:
1. Download Dataset From S3
2. Store On EBS
3. Train Locally
4. Save Artifacts
5. Upload Artifacts To S3
6. Destroy EC2

'''

'''
-----------------------------------------------------------
IMPORTENT THINGS TO READ


Why download to EC2? Just read directly from S3.
Many engineers think this at first.

Let's analyze it.


Architecture:

S3
↓
Image 1
↓
Train

S3
↓
Image 2
↓
Train

S3
↓
Image 3
↓
Train

No local storage.
No downloads.

Looks efficient.

But Let's Think Like PyTorch

Suppose:

Dataset = 10,000 images
Epochs = 30

Epoch 1:

Read image 1 from S3
Read image 2 from S3
...
Read image 10000 from S3

Epoch 2:

Read image 1 from S3
Read image 2 from S3
...
Read image 10000 from S3

Epoch 30:

Read image 1 from S3
Read image 2 from S3
...
Read image 10000 from S3

Question:

How many image reads?

10,000 × 30 = 300,000 reads
Now Think About Network

Every read means:

S3
↓
Network
↓
EC2
↓
PyTorch

Training loop:

for epoch in range(30):
    for image in dataloader:
        ...

would constantly wait for:

Network

instead of:

CPU
GPU
Industry Reality

Most training systems do:

S3
↓
Download Once
↓
Local SSD / EBS
↓
ImageFolder
↓
DataLoader
↓
Train

Why?

Because:

Local Disk

is MUCH faster than:

Network + S3
'''

----------------------------------------------------------------------------------------------------------------------------------------------

DOCKER TRAIN
--------------

1. upload data to s3 
(as data is in your local use aws s3 cp <local_data_location> <s3_name_folder_if_exist_or_it_will_create> )
# aws s3 cp "D:\mlops\notes\v2\docker-train\data_to_upload_on_s3_awsS3CpThisLocatonSpaceS3Location" "s3://maaz-animal-classifier-v2/data/raw/" --recursive


'''
# Note
-------
if 
s3 sync
then no ---recursive 

you know the difference right 
==============================
sync will not upload the already uploaded file 

if s3 has 500 images 
now upload 510 images 
cp : will upload all 510 images
sync : will upload only 10 images

'''

2. verify 
aws s3 ls s3://maaz-animal-classifier-v2/data/raw/ --recursive



3. now write then entire training code 


4. now run the train.py file 
# python train.py

no need to configure IAM to the s3 client to do s3 operation as aws CLI already being configured with s3
see from git-bash or powershell or cmd or terminal 


'''
-------------------------------------------------------------------------------------------------------------------
Maaz@DESKTOP-TMSIV79 MINGW64 
$ aws sts get-caller-identity
{
    "UserId": "AIDA55GUVE36D5SDRD6U7",
    "Account": "956076205820",
    "Arn": "arn:aws:iam::956076205820:user/maaz-DL"
}


if aws was not configurated in then 

# high level api easy to work and control basic objects
s3_resource = boto3.resource('s3',
                                aws_access_key_id=__access_key_id,
                                aws_secret_access_key=__secret_access_key,
                                region_name=region_name
                                )

# low level api control full features of s3 . used when we want more features and flexibility
s3_client = boto3.client('s3',
                            aws_access_key_id=__access_key_id,
                            aws_secret_access_key=__secret_access_key,
                            region_name=region_name
                            )
                            
use these two reference to CRUD in s3 bucket
s3_resource
s3_client
'''


5. write Dockerfile

6. build .dockerignore
'''
virt/
venv/
__pycache__/
*.pyc
*.pyo
tmp/
data/
artifacts/
.git/
.gitignore
'''

6. run 
(sometime you have to open docker desktop to run this)
# docker build -t animal-classifier-train:v1 .

7. login to ECR
# aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 956076205820.dkr.ecr.ap-south-1.amazonaws.com

8. Tag image
(just creating another name of image to push in ECR)
docker tag animal-classifier-train:v1 `
956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1

9. push image 
docker push 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1



10. connect with EC2
ssh -v -i D:\mlops\notes\maaz-classifier-train-v2.pem ubuntu@3.7.70.54

11. login to ECR
# aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 956076205820.dkr.ecr.ap-south-1.amazonaws.com

12. pull the image from ECR
# docker pull 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1


# ###################################################################
above 12 to 21 follow 
----------------------


13. run the image from EC2 you downloaded 
# docker run 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-train:v1



'''
# ============================================================================================================================================
READ THIS 
---------
First: Small Correction

You said:

/tmp/data and /tmp/artifacts are in EC2 -> EBS and will remain there

Actually:
NO
Not in our current setup.

Remember:

docker run image

creates:
Container Filesystem

Inside container:

/tmp/data
/tmp/artifacts

exist in: Container Layer

not directly on EBS.

Visual:

EC2
│
├── EBS
│
└── Docker
     │
     └── Container
          │
          ├── /tmp/data
          └── /tmp/artifacts

When container exits:

docker run image
↓
Container Stops
↓
Container Deleted

those files disappear.

Why are we okay?
Because:

Dataset
came from: S3

and can be downloaded again.

And:

"Artifacts"

were uploaded to:
S3

before container exited.

So nothing valuable is lost.

Now Your Main Question

You asked:

when container runs pip install requirements.txt, will packages be installed globally or in container?

Answer:

Inside Container Only

🔥 This is one of Docker's superpowers.

Think About Build Phase

During:

docker build

Docker executes:

RUN pip install -r requirements.txt

But WHERE?

Not on:

Your Laptop

Not on:

EC2 Global Python

Inside a temporary build container.

Result:

Docker Image
├── Python
├── Torch
├── TorchVision
├── boto3
└── Your Code

Everything becomes part of the image.

When You Run
docker run image

Docker creates:

Container

from that image.

Container already contains:

Python
Torch
TorchVision
boto3

because they were installed during:

docker build
Virtual Environment?

You said:

I guess I should activate virtual environment

Inside Docker:

Usually NO

Because container itself is already isolated.

Think:

Laptop
├── Python
├── Java
├── Node
├── Project A
├── Project B

Need:

venv

to avoid conflicts.

But Docker container is already:

Mini Machine

Visual:

Container A
├── Python 3.13
├── Torch

Container B
├── Python 3.10
├── TensorFlow

No conflict.

Industry Practice

Outside Docker:

Use venv

Inside Docker:

Usually No venv

because Docker itself provides isolation.

What Happened In Your Run

This line:

Downloading:
https://download.pytorch.org/models/resnet18-f37072fd.pth

shows something interesting.

Your image contains:

Torch
TorchVision

But NOT:

ResNet18 Weights

Therefore first run:

TorchVision
↓
Downloads Pretrained Weights
↓
Stores In
/root/.cache/torch/

inside container.

Then training starts.

Mentor Gold Nugget

The fact that:

docker run image

worked on EC2 WITHOUT:

python -m venv
source venv/bin/activate
pip install ...

proves that you have successfully created a self-contained training image.

That's one of the biggest milestones in MLOps.

You are no longer deploying code.

You are deploying an environment + code + dependencies bundle.
'''


































INTERFERENCE 
######################################################################################################################################



1. CREATE folder structure 

animal-classifier-inference/
├── app.py
├── config.json
├── requirements.txt
│
└── src/
    ├── artifact_downloader.py
    ├── model_loader.py
    ├── predictor.py
    └── image_transform.py




2. write entire code 

    a. write download_artifacts code (just downlaod " best_model.pth , config.json , class_mapping ")

    b. write model_loader code

    Step 1 : Recreate architecture
    model = models.resnet18(...)

    Step 2 : Replace final layer
    model.fc = nn.Linear(...)

    Step 3 : Load weights
    model.load_state_dict(...)

    '''
        Download Artifacts
        ↓
        Load Config
        ↓
        Load Class Mapping
        ↓
        Create ResNet18
        ↓
        Replace Final Layer
        ↓
        Load best_model.pth
        ↓
        Model Ready
    '''


3. similarly all code all the files

4. make virtual env then activate the virt

5. run the app.py (fast api )
# uvicorn app:app --reload

6. go to  : http://127.0.0.1:8000/docs

7. create docker image 
# docker build -t animal-classifier-inference:v1 .

8. verify dockerignore 
(by verifying the layaer of copy . . size it shoud be in few kb)
# docker history animal-classifier-inference:v1

'''
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
afc947e4c321   22 minutes ago   CMD ["uvicorn" "app:app" "--host" "0.0.0.0" …   0B        buildkit.dockerfile.v0
<missing>      22 minutes ago   EXPOSE [8000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      22 minutes ago   COPY . . # buildkit                             65.5kB    buildkit.dockerfile.v0
<missing>      22 minutes ago   RUN /bin/sh -c pip install --no-cache-dir -r…   5.12GB    buildkit.dockerfile.v0
<missing>      39 minutes ago   COPY requirements.txt . # buildkit              12.3kB    buildkit.dockerfile.v0
<missing>      27 hours ago     WORKDIR /app                                    8.19kB    buildkit.dockerfile.v0
<missing>      9 days ago       CMD ["python3"]                                 0B        buildkit.dockerfile.v0
<missing>      9 days ago       RUN /bin/sh -c set -eux;  for src in idle3 p…   16.4kB    buildkit.dockerfile.v0
<missing>      9 days ago       RUN /bin/sh -c set -eux;   savedAptMark="$(a…   40.2MB    buildkit.dockerfile.v0
<missing>      9 days ago       ENV PYTHON_SHA256=639e43243c620a308f968213df…   0B        buildkit.dockerfile.v0
<missing>      9 days ago       ENV PYTHON_VERSION=3.13.14                      0B        buildkit.dockerfile.v0
<missing>      9 days ago       ENV GPG_KEY=7169605F62C751356D054A26A821E680…   0B        buildkit.dockerfile.v0
<missing>      9 days ago       RUN /bin/sh -c set -eux;  apt-get update;  a…   4.94MB    buildkit.dockerfile.v0
<missing>      9 days ago       ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
<missing>      10 days ago      # debian.sh --arch 'amd64' out/ 'trixie' '@1…   87.4MB    debuerreotype 0.17
'''


==========================================================================================================================================


9. run docker container 
# docker run -p 8000:8000 animal-classifier-inference:v1

'''
it will give error as in this code you are trying to connect with s3 but in our code we haven't connected with docker ,
think about a separeate isolated container does it has anything to connect with s3 (IAM)

but how it works in local ?
becase local cli has iam configuration and when you write caller it will give response of your caller 

# ------------------------------------------------------------------------------------------------------------------------------------------------
Why It Worked Locally Without Docker

When you ran:

uvicorn app:app --reload

boto3 found:

C:\Users\Maaz\.aws\credentials

that came from:

aws configure
Why It Fails In Docker

Inside container:

Container
│
├── Python
├── boto3
├── FastAPI
└── NO AWS CREDENTIALS

There is no:

C:\Users\Maaz\.aws

inside container.

Therefore:

boto3.client("s3")

throws:

NoCredentialsError
Unable to locate credentials
Did We Make A Coding Mistake?
NO

Your code is fine.

This is an infrastructure/authentication issue.

How To Fix Locally?

For local Docker testing only:

Mount AWS credentials.

docker run `
-p 8000:8000 `
-v C:\Users\Maaz\.aws:/root/.aws:ro `
animal-classifier-inference:v1

Now inside container:

/root/.aws/credentials

exists.

boto3 can authenticate.

Mentor Question

Why did the training container not fail like this?

Because we never actually ran:

docker run animal-classifier-train:v1

locally after adding S3 download logic.

We tested training mainly:

Local Python

and then:

EC2

where IAM Role handled authentication.

What Will Happen On EC2?

This is the important part.

On EC2:

EC2
│
├── IAM Role
│      maaz-ec2-s3-role
│
└── Docker Container

When container starts:

boto3.client("s3")

AWS SDK automatically does:

Container
↓
EC2 Metadata Service
↓
IAM Role Credentials
↓
S3

So:

NO access keys
NO aws configure
NO mounted credentials

needed.

Real Industry Practice
Local Development

Either:

aws configure

or

-v ~/.aws:/root/.aws
EC2

Use:

IAM Role
ECS

Use:

Task Role
EKS

Use:

Service Account IAM Role
SageMaker

Use:

Execution Role


'''



11. run with aws authentication of local cli
# docker run -p 8000:8000 -v C:\Users\Maaz\.aws:/root/.aws:ro animal-classifier-inference:v1



------------------------------------------------------------------------
NOW TIME TO PUSH THE ECR 

12. create ECR repository 
(we want separate repository for training and interface that is why we are creating an interferance ECR)

# aws ecr create-repository --repository-name animal-classifier-inference

output : 
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-south-1:956076205820:repository/animal-classifier-inference",
        "registryId": "956076205820",
        "repositoryName": "animal-classifier-inference",
        "repositoryUri": "956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-inference",
        "createdAt": "2026-06-20T23:20:25.380000+05:30",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}

13. verify docker ECR created or not
# aws ecr describe-repositories

14. tag docker image 
# docker tag animal-classifier-inference:v1 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-inference:v1

15. login to ECR (connection of docker with ECR)
# aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 956076205820.dkr.ecr.ap-south-1.amazonaws.com

16. push to aws ECR
# docker push 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-inference:v1

17. after push verify if you image has been uploaded in your repository or not 
# aws ecr describe-images --repository-name animal-classifier-inference





-------------------------------------------------------------------------------------------------------------------------------------
Running interference image from EC2

18. go to ec2
# ssh -v -i D:\mlops\notes\maaz-classifier-train-v2.pem ubuntu@3.7.70.54

19. verify caller
# aws sts get-caller-identity

should show:
assumed-role/maaz-ec2-s3-role

20. login to ECR
# aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 956076205820.dkr.ecr.ap-south-1.amazonaws.com

21. pull the image from ECR
# docker pull 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-inference:v1

22. run the image (you gave a name inference-api to your container)
# docker run -d -p 8000:8000 --name inference-api 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-inference:v1

'''
it will run the container and finish the process that is stop the container , but api has created , you have to start the api
-----------------------------------------------------------------------------

>> docker run -d -p 8000:8000 --name inference-api 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-inference:v1

output : 084f1f1526536ceecabc75cd4a41e88c286fe04e5904412d2b86cf764c954e69

'''

23. start the api 
# docker logs -f inference-api

24. add port 8000 to your EC2 
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

------------------------------------------------------------------------
AFTER ADDING PORT

24. now your public api
# curl ifconfig.me
output : 3.7.70.54

25. open 
# http://3.7.70.54:8000/docs

wllah 

26. now delete everything learning is complete for v2

'''
What To Delete?
1. Stop and Remove Container

On EC2:

docker ps

Find:

inference-api

Stop it:

# docker stop inference-api

Remove it:

# docker rm inference-api

2. Remove Images (Optional)

# docker images

Remove:

# docker rmi 956076205820.dkr.ecr.ap-south-1.amazonaws.com/animal-classifier-inference:v1

3. Terminate EC2

AWS Console:

EC2
↓
Instances
↓
Select Instance
↓
Instance State
↓
Terminate

Not Stop.

Terminate.

Because:

Stopped EC2
=
EBS still attached
=
Can still incur storage cost

Terminate removes the instance.

4. Delete ECR Repositories

AWS Console:

ECR
↓
Repositories
↓
animal-classifier-train
↓
Delete

animal-classifier-inference
↓
Delete

Or CLI:

# aws ecr delete-repository --repository-name animal-classifier-train --force

# aws ecr delete-repository --repository-name animal-classifier-inference --force

5. Delete S3 Bucket

Delete:

maaz-animal-classifier-v2

or empty it first then delete.

Remember:

S3 Objects
=
Storage Cost
'''