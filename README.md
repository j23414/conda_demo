# Conda Environment Demo

### Part 1: Getting ready to build a conda environments

Normally you'd log into Ceres or Atlas with your username and the full ssh path

```
ssh YOUR_USERNAME@ceres.scinet.usda.gov
ssh YOUR_USERNAME@atlas-login.hpc.msstate.edu
```

On Mac and Linux, I highly recommend adding the following to your `~/.ssh/config` file:

```
Host ceres
     HostName ceres.scinet.usda.gov
     User YOUR_USERNAME
     ForwardAgent yes
     ForwardX11 no
     TCPKeepAlive yes
     ServerAliveInterval 20
     ServerAliveCountMax 30
     Compression yes

Host atlas
     HostName atlas-login.hpc.msstate.edu
     User YOUR_USERNAME
     ForwardAgent yes
     ForwardX11 no
     TCPKeepAlive yes
     ServerAliveInterval 20
     ServerAliveCountMax 30
     Compression yes
```

so you can log in with:

```
ssh ceres
ssh atlas
```

After logging in, allocate an interactive node.

```
sinfo            # list partitions

# === Ceres
salloc -N 1 -n 4 -t 04:00:00 -p scavenger 

# === Atlas
salloc -N 1 -n 4 -t 04:00:00 -p atlas -A isu_gif_vrsc   # <= you will need your account name
```

load or install miniconda

```
(module avail) &> module_list.txt       # <= I tend to list all software first
grep -i conda module_list.txt

# === Ceres
module load miniconda
conda --version

# === Atlas
# Fetch the install script
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# Run the fetched script
bash Miniconda3-latest-Linux-x86_64.sh
# acceept all defaults (press enter/yes through entire download process)

# By default it places miniconda3 in home folder (remember 5gb limit)
# Ergo we move this to the project folder and link to home
mv ~/miniconda3 /project/project_folder/software/.
ln -s /project/project_folder/software/miniconda3 ~/.

# Check if miniconda is installed and working
source ~/.bashrc
conda -version
```

[Link to my notes](https://github.com/ISUgenomics/2021_workshop_transcriptomics/blob/main/Notebook_Jennifer/archived_notes/01_align_gsnap.md#install-miniconda)

### Part 2: Building a conda environment

For building and maintaining conda environments, I highly recommend using an environment.yml file. We'll use the following example.

```
name: myproject_env
channels:
  - conda-forge
  - bioconda
  - defaults
dependencies:
  - python=3.8
  - kallisto
  - salmon
  - pandas
```

Build the environment

```
conda env create -f environment.yml -p ${PWD}/myproject_env

# === Ceres
source activate ${PWD}/myproject_env

# === Atlas
conda activate ${PWD}/myproject_env

echo $PATH
```

Check that our installed software is there:

```
kallisto
salmon -h
which kallisto       # <= will show path to environment folder
which salmon
```

Can also check the python library

```
python                 # start python interpreter
import pandas as pd
exit()                 # exit python interpreter
```

When done, can deactivate the environment. 

```
# === Ceres
source deactivate

# === Atlas
conda deactivate
```

### Part 3: Using the conda environment in a Slurm Script:

Slurm script on Ceres

```
#! /usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --time=24:00:00
#SBATCH --job-name=conda_demo
#SBATCH --output=R-%x.%J.out
#SBATCH --error=R-%x.%J.err
#SBATCH --mail-user=YOUR_USERNAME@email.com
#SBATCH --mail-type=begin,end

# === Set working directory and in/out variables
cd ${SLURM_SUBMIT_DIR}

# === Load conda environment
module load miniconda
#source activate myenv_py3.8   # if default location
source activate /project/isu_gif_vrsc/demo/myproject_env   #<= remember to edit path

# === Main Program
echo "Paths:" > version.txt
which kallisto >> version.txt
which salmon >> version.txt

echo "Usage:" >> version.txt
kallisto >> version.txt
salmon -h >> version.txt

sleep 10
```

Slurm script on Atlas

```
#! /usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --time=24:00:00
#SBATCH --job-name=conda_demo
#SBATCH --output=R-%x.%J.out
#SBATCH --error=R-%x.%J.err
#SBATCH --mail-user=YOUR_USERNAME@email.com
#SBATCH --mail-type=begin,end
#SBATCH --account=isu_gif_vrsc                              #<= Atlas Requires an account name

# === Set working directory and in/out variables
cd ${SLURM_SUBMIT_DIR}

# === Load conda environment
source /home/${USER}/miniconda3/etc/profile.d/conda.sh
conda activate /project/isu_gif_vrsc/demo/myproject_env     # <= remember to edit path

# === Main Program
echo "Paths:" > version.txt
which kallisto >> version.txt
which salmon -h >> version.txt

echo "Usage:" >> version.txt
kallisto >> version.txt
salmon -h >> version.txt

sleep 10
```

Submit to slurm job scheduler

```
sbatch script.slurm

# View your job on queue
squeue -u $USER

# List most recent created files
ls -ltr
```

## Other Resources

* [Conda Cheat Sheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)
* [Install miniconda](https://docs.conda.io/en/latest/miniconda.html#linux-installers)
* [SCreating an environment with commands](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands)
* [SCINet Website: User installed software on Ceres with Conda](https://scinet.usda.gov/guide/conda/#user-installed-software-on-ceres-with-conda)
* [SCINet Forum: Atlas conda installed tools works on terminal but not in shell script via sbatch](https://forum.scinet.usda.gov/t/atlas-conda-installed-tools-works-on-terminal-but-not-in-shell-script-via-sbatch/185)
* [SCINet Forum: Running conda environments on project directories in Jupyterhub](https://forum.scinet.usda.gov/t/running-conda-environments-on-project-directories-in-jupyterhub/131)
* [MS Stream: Using Jupyter Hub on CERES - Brian Nadon](https://web.microsoftstream.com/video/4c9c0a01-5113-4cd5-a0da-63a7c68b230d)
* [MS Stream: Onboarding_2021](https://web.microsoftstream.com/video/f22c4659-40fd-4546-bbf5-4add649a870e)

## References

* [2018 Nov 30 - Anaconda (Conda) for Python - What & Why](https://youtu.be/23aQdrS58e0)
* [2020 Jul 14 - Master the basics of Conda environments in Python](https://youtu.be/1VVCd0eSkYc)
* [2019 Feb 18 - Anaconda Tutorial 2019 - Python Virtual Environment Manager](https://youtu.be/mIB7IZFCE_k)
* [2021 May 1 - The only CONDA tutorial you'll need to watch to get started](https://youtu.be/sDCtY9Z1bqE)
