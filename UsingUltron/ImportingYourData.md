---
title: Importing Your Data
description: Copy your datasets and your python scripts
published: true
date: 2026-04-19T17:31:32.228Z
tags: datasets, scripts, copying, scp, sftp
editor: markdown
dateCreated: 2026-04-19T16:19:14.729Z
---

# Moving Files to Ultron
Since computation is to be performed on Ultron, it must have all your relevant files. Ultron is currently targeted for AI and ML so with that objective in mind, we'll train a simple Cat-vs-Dog CNN.

## Copying the Dataset
First we download the dataset from [Kaggle](https://www.kaggle.com/datasets/shaunthesheep/microsoft-catsvsdogs-dataset?resource=download). The structure of the downloaded and then unzipped folder is as follows,
```bash
sysadmin@sysadmin-HP-EliteBook-850-G4:~/Downloads/archive$ ls
'MSR-LA - 3467.docx'   PetImages  'readme[1].txt'
sysadmin@sysadmin-HP-EliteBook-850-G4:~/Downloads/archive$ tree -d
.
└── PetImages
    ├── Cat
    └── Dog

4 directories
```
The requirement of the python script we are using for training is such that we must only have the two folders `Cat` and `Dog`. The additional files are not required. We copy the appropriate dataset. This might take a while.

```bash
scp -r archive/PetImages/ student@192.168.24.100:/data/student/
```

Once this is done, verify if your files are there. SSH into Ultron, then

```bash
ls /data/testuser
```
This should return `PetImages`. Congratulations, your dataset has travelled safe.

## Creating/Copying Python Script
With the dataset copied, we must now create our project directory and copy in our python script. Login to your account,
```bash
ssh student@192.168.24.100
```
Create a new directory,
```bash
mkdir animals
```
Now from your laptop, copy your python script,
```bash
scp train.py student@192.168.24.100:/home/student/animals/
``` 
Or create a file and write your script. `nano` is a text editor in your terminal. 
```bash
nano train.py
```

## Summing Up
This does it for creating a copy of your files on Ultron. You are ready to train the model now.

Next: [Submitting Your First Job](/UsingUltron/SubmittingYourFirstJob)

