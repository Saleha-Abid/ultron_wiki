---
title: Submitting Your First Job
description: Learn how to use our home-made script to create basic slurm batch files.
published: true
date: 2026-04-27T03:21:36.845Z
tags: slurm, batch script, jobs, ai, ml, training, running scripts
editor: markdown
dateCreated: 2026-04-19T16:21:56.174Z
---

# The First Slurm Job
This is the final step to training our Cats-vs-Dogs CNN. Atlas as an HPC uses **Slurm** as its *Job Scheduler*. Atlas's entire use depends on using Slurm to submit jobs which are infact just python scripts in the running. This site will guide you in Slurm's efficient use but you can read more about it [here](https://slurm.schedmd.com/). 

## Generating the Batch Script
Slurm is professional tool used in HPCs across the world for job scheduling. Its use is complicated and detailed. One way to use it is through job scripts. We create simple bash files and execute them using the `sbatch` command. The bash file includes a lot of parameters. Listed below are only some of them.
- The number of nodes being used
- The number of CPUs required
- The amount of memory required
- The expected time the job demands

For ease of our users, we have a custom **script generate** for AI and ML. Run this in your VSCode terminal after you have connected to Remote host.   

```bash
cd animals
gen-script
```
It will ask you a bunch of parameters, in our case,
```bash
student@head01:~/animals$ gen-script
--- Atlas Job Generator [Dir: animals] ---
Job Name [animals]: cats
Python Script Name (e.g., train.py): train.py
Output Model Name (Enter to skip): cats_dogs_resnet18.pth
CPUs [8]: 8
RAM (include unit, e.g., 16G) [32G]: 6G
GPUs [1]: 1
Include NFS dataset? (y/n): y
NFS Source Path: /data/testuser/PetImages

✅ Created submit_cats.sh in /home/student/animals
👉 To run: sbatch submit_cats.sh
```

My python file outputs its model by the file name of `cats_dogs_resnet18.pth`. The script will appear on the *File Explore* panel on the left.

```bash
#!/bin/bash
#SBATCH --job-name=cats
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=6G
#SBATCH --gres=gpu:1
#SBATCH --chdir=/scratch/student/animals
#SBATCH --output=/scratch/student/animals/logs/job_%j.log
#SBATCH --error=/scratch/student/animals/logs/job_%j.err

# Standard Atlas Paths
REMOTE_HOST="head01"
REMOTE_PATH="/home/student/animals"
LOCAL_SCRATCH="/scratch/student/animals"

# a. Initialize Scratch Space
mkdir -p $LOCAL_SCRATCH/logs

# b. Sync code from headnode
echo "Syncing code from $REMOTE_PATH..."
scp -r testuser@$REMOTE_HOST:$REMOTE_PATH/* $LOCAL_SCRATCH/

# c. Sync Dataset from NFS
echo "Syncing dataset..."
rsync -ah --progress /data/student/PetImages/ $LOCAL_SCRATCH/data/

# d. Environment and Run
source /opt/ultron/miniforge3/etc/profile.d/conda.sh
conda activate ai_base

python3 train.py

# e. Sync results back to headnode
scp $LOCAL_SCRATCH/cats_dogs_resnet18.pth student@$REMOTE_HOST:$REMOTE_PATH/
scp -r $LOCAL_SCRATCH/logs student@$REMOTE_HOST:$REMOTE_PATH/
```

Essentially everything after line 12 is written as if you are on the node itself. 
> Write your job script as if you are on the compute node.
{.is-info}

## Running the Script
Let's train our model. On the VSCode terminal, run,
```bash
sbatch submit_cats.sh
```
To see your job in the queue,
```bash
student@head01:~/animals$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                49   compute     cats testuser  R       1:03      1 node01
```
`R` means that your job is running. You can also see how long it has been running for and the node assigned to it.

To cancel your job,
```bash
scancel 49
```
`49` is your JobID. You identify it through `squeue`. When your job is done, it will disappear from `squeue`.

Look at your *File Exporer* panel on the left. The trained model is sitting right there! We also have our log files. The `.err` file gives you any and all errors encountered including those for you python script. The `.log` gives you the Terminal outputs all along the execution. Check them to know how the training went.

## Deciding Your Rescources
How many CPUs do you need? Do you need a GPU for your training or not? These are common questions that might baffle you when creating a job script. Here are some pointers,
- Keep atleast 4 CPUs for your job.
- Assign atleast 4GB RAM. The cap is 100GB. Also, don't forget to add 'G' when using `gen-script`.
- A GPU will make your job run many times faster. Do that for intense training sessions with larger number of epochs. (You can assign 2 GPUs per node - but don't bag them).
- There are also other parameters but the `gen-script` ignores them for basic usage. Learn how to create advanced scripts by following along.

## Summing Up
Congratulations, you trained your first model on Atlas. This was a simple Cats-vs-Dogs CNN but with your basics to handle datasets and script locations, you can train any model!

Next: 