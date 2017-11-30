# Entoto High Performace Computing Cluster

A quick reference to access Entoto's High Performance Computing  Cluster at Chemistry Department, AAU.



## Get an account

You need to be affiliated to AAU and have a sponsor. 

To get an account approved, follow [this steps.]()

## Log in

Once you have been approved, you can access HPC from:

 1. Within the AAU network:

```bash
ssh username@10.4.17.30
```

Once logged in, the root should be:
`/home/username`, so running `pwd` should print:

```bash
[username@log-0 ~]$ pwd
/home/username
```

2. An off-campus location:

First, login to the bastion host:

```bash
ssh username@entoto.aau.edu.et
```

Then login to the cluster:

```bash
ssh entoto.aau.edu.et
```

## File Systems

You can get acces to three filesystems: `/home`, `/scratch`, and `/archive`.

Scratch is a file system mounted on Prince that is connected to the compute nodes where we can upload files faster. Notice that the content gets periodically flushed.

```bash
[username@log-0 ~]$ cd /scratch/username
username@log-0 ~]$ pwd
/scratch/username
```

`/home` and `/scratch` are separate filesystems in separate places, but you should use `/scratch` to store your files.

## Loading Modules

Slurm allows you to load and manage multiple versions and configurations of software packages.

To see available package environments:
```bash
module avail
```

To load a model:
```bash
module load [package name]
```

For example if you want to use Tensorflow-gpu:
```bash
module load openmpi-x-
module load gpaw/1.0
module load espresso/3.6
module load tensorflow/python3.6/1.3.0
```

To check what is currently loaded:
```bash
module list
```

To remove all packages:
```bash
module purge
```

To get helpful information about the package:
```bash
module show torch/gnu/20170504
```
Will print something like 
```bash
--------------------------------------------------------------------------------------------------------------------------------------------------
   /share/apps/modulefiles/torch/gnu/20170504.lua:
--------------------------------------------------------------------------------------------------------------------------------------------------
whatis("Torch: a scientific computing framework with wide support for machine learning algorithms that puts GPUs first")
whatis("Name: torch version: 20170504 compilers: gnu")
load("cmake/intel/3.7.1")
load("cuda/8.0.44")
load("cudnn/8.0v5.1")
load("magma/intel/2.2.0")
...
```
`load(...)` are the dependencies that are also loaded when you load a package.


## Interactive Mode: Request CPU

You can submit batch jobs in prince to schedule jobs. This requires to write custom bash scripts. Batch jobs are great for longer jobs, and you can also run in interactive mode, which is great for short jobs and troubleshooting.

To run in interactive mode:

```bash 
[username@log-0 ~]$ srun --pty /bin/bash
```

This will run the default mode: a single CPU core and 2GB memory for 1 hour.

To request more CPU's:

```bash
[username@log-0 ~]$ srun -n4 -t2:00:00 --mem=4000 --pty /bin/bash
[username@c26-16 ~]$ 
```
That will request 4 compute nodes for 2 hours with 4 Gb of memory.


To exit a request:
```
[username@c26-16 ~]$ exit
[username@log-0 ~]$
```

## Interactive Mode: Request GPU

```bash
[username@log-0 ~]$ srun --gres=gpu:1 --pty /bin/bash
[username@gpu-25 ~]$ nvidia-smi
Mon Oct 23 17:49:19 2017
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 367.48                 Driver Version: 367.48                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           On   | 0000:12:00.0     Off |                    0 |
| N/A   37C    P8    29W / 149W |      0MiB / 11439MiB |      0%   E. Process |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```


## Submit a job

You can write a script that will be executed when the resources you requested became available.

A simple CPU demo:

```bash
## 1) Job settings

#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --time=5:00:00
#SBATCH --mem=2GB
#SBATCH --job-name=CPUDemo
#SBATCH --mail-type=END
#SBATCH --mail-user=itp@aau.edu.et
#SBATCH --output=slurm_%j.out
  
## 2) Everything from here on is going to run:

cd /scratch/username/demos
python demo.py
```

Request GPU:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:4
#SBATCH --time=10:00:00
#SBATCH --mem=3GB
#SBATCH --job-name=GPUDemo
#SBATCH --mail-type=END
#SBATCH --mail-user=itp@aau.edu.et
#SBATCH --output=slurm_%j.out

cd /scratch/NYUNetID/trainSomething
source activate ML
python train.py
```

Submit your job with:

```bash
sbatch myscript.s
```

Monitor the job:

```bash
squeue -u $USER
```



## Setting up a tunnel

To copy data between your workstation and the NYU HPC clusters, you must set up and start an SSH tunnel.

What is a tunnel?

> "A tunnel is a mechanism used to ship a foreign protocol across a network that normally wouldn't support it."<sup>[1](http://www.enterprisenetworkingplanet.com/netsp/article.php/3624566/Networking-101-Understanding-Tunneling.htm)</sup>

1. In your local computer root directory, and if you don't have it already, create a folder called `/.shh`:
```bash
mkdir ~/.ssh
```

2. Set the permission to that folder:
```bash
chmod 700 ~/.ssh
```

3. Inside that folder create a new file called `config`:
```bash
touch config
```

4. Open that file in any text editor and add this: 
```bash
# first we create the tunnel, with instructions to pass incoming
# packets on ports 8024, 8025 and 8026 through it and to specific
# locations
Host hpcgwtunnel
   HostName gw.entoto.aau.edu.et
   ForwardX11 no
   LocalForward 8025 entoto.aau.edu.et:22
   LocalForward 8026 entoto.aau.edu.et:22
   Username 
# next we create an alias for incoming packets on the port. The
# alias corresponds to where the tunnel forwards these packets
Host dumbo
  HostName localhost
  Port 8025
  ForwardX11 yes
  User NetID

Host prince
  HostName localhost
  Port 8026
  ForwardX11 yes
  User NetID
```

Be sure to replace the `NetID` for your AAU NetId

## Transfer Files

To copy data between your workstation and the NYU HPC clusters, you must set up and start an SSH tunnel. (See previous step)


1. Create a tunnel
```bash
ssh hpcgwtunnel
```
Once executed you'll see something like this:

```bash
Last login: Wed Nov  8 12:15:48 2017 from 74.65.201.238
username5@Ntoto~>$
```

This will use the settings in `/.ssh/config` to create a tunnel. **You need to leave this open when transfering files**. Leave this terminal tab open and open a new tab to continue the process.

2. Transfer files

### Between your computer and the HPC

- A File:
```bash
scp /Users/local/data.txt UserName@Ntoto:/scratch/UserName/path/
```

- A Folder:
```bash
scp -r /Users/local/path UserName@Ntoto:/scratch/UserName/path/
```

### Between the HPC and your computer

- A File:
```bash
scp NYUNetID@Ntoto:/scratch/UserName/path/data.txt /Users/local/path/
```

- A Folder:
```bash
scp -r username@Ntoto:/scratch/UserName/path/data.txt /Users/local/path/ 
```



