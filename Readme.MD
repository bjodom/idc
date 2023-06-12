# Batch Mode Acces to PVC-Enabled SPR Systems in the Intel® Developer Cloud

- [Welcome to the Intel Dev Cloud](#welcome-to-the-intel-dev-cloud)
  - [Sign Up](#sign-up)
  - [SSH Setup](#ssh-setup)
  - [Head Node vs Compute Nodes](#head-node-vs-compute-nodes)
  - [Environment Setup](#environment-setup)
  - [Additional Software](#additional-software)
  - [Common Commands](#common-commands)
  - [Sample GPU Test Code](#sample-gpu-test-code)
  - [Jupyter](#jupyter)
  - [Some Example Scripts](#some-example-scripts)
  - [Where to get Support](#where-to-get-support)

---  


## Sign Up<div id='sign-up'/>

The Intel Developer Cloud (IDC) trial is open to prequalified Intel customers and approved developers.  

To get started: <a href="https://www.intel.com/content/www/us/en/secure/forms/developer/standard-registration.html?tgt=www.intel.com/content/www/us/en/secure/developer/devcloud/cloud-launchpad.html">Sign up for a standard Intel® Developer Zone account.</a>  You will need an RSA Public key, see the next section if you need more information on setting up your SSH authentication and client.

Schedule and deploy the service with <a href="https://scheduler.cloud.intel.com/">Intel® Developer Cloud management console.</a>

---  
## SSH Setup<div id='ssh-setup'/>

### ssh-keygen

ssh-keygen is a tool for creating new authentication key pairs for SSH. Such key pairs are used for automating logins, single sign-on, and for authenticating hosts.  The IDC uses SSH Keys exclusively and you will never use a password for authentication.

### SSH Public Key Creation

To create a key use the `ssh-keygen` utility found in your terminal application.  Windows powershell, Windows Subsytem for Linux (WSL) Terminal, Linux or MAC Terminal

For WSL, Linux and MAC clients enter the below command
```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519_idc -C "you@email.com"
```
For PowerShell enter:
```bash
ssh-keygen -o -a 100 -t ed25519 -f C:\Users\YourID\.ssh\id_ed25519_idc
```

The passphrase is optional and you can hit enter for no pass phrase.  This will result in two files being generated: `id_ed25519_idc` and `id_ed25519_idc.pub` take care of these files as they are the private and public key pair that will be tied to your IDC account.

### SSH .config Client Setup

To make accessing the IDC convenient, it is recommended to setup a `.ssh\config` file.

```bash
Host idc #←YOU CAN CALL IT ANYTHING
Hostname idcbetabatch.eglb.intel.com
User uXXXXXX #← THIS WAS ASSIGNED TO YOU AFTER SIGNING UP
IdentityFile ~/.ssh/idc-key
#ProxyCommand /usr/bin/nc -x YourProxy:XXXX %h %p # Uncomment if necessary
ServerAliveInterval 60
ServerAliveCountMax 10
StrictHostKeyChecking no # Frequent changes in the setup are taking place now, this will help reduce the knownhosts errors.
UserKnownHostsFile=/dev/null
```
### For Intel Employees:  Yours should look like below:

```bash
Host idcbeta
Hostname idcbetabatch.eglb.intel.com
User uXXXXXX #← THIS WAS ASSIGNED TO YOU AFTER SIGNING UP
IdentityFile ~/.ssh/id_rsa_idc2
ProxyCommand /usr/bin/nc -x proxy-dmz.intel.com:1080 %h %p
ServerAliveInterval 60
ServerAliveCountMax 10
StrictHostKeyChecking no
UserKnownHostsFile=/dev/null
```


Ensure that `.ssh` has 600 privilege bits set  `-rw-------`  

Ensure that `config` has 600 privilege bits set  `-rw-------`  

Ensure that `.idc-key` and `idc-key.pub` have 400 privilege bits set `-r--------`  

In this configuration from the terminal future connections are established by entering:

```bash
ssh idcbeta
```
You are allowed up to 2 connections to the IDC.  

---  
## Head Node vs Compute Nodes<div id='head-node-vs-compute-node'/>

Upon initial connection to the IDC, you are connected to the head node.  This environment is a standard Ubuntu 22.04.02 LTS environment including `dev-essential` and the <a href="https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit.html#gs.yh16wk">Intel oneAPI Basekit.</a>  This IDC utilizes <a href="https://slurm.schedmd.com/quickstart.html">SLURM</a> to manage job scheduling and resource management.   As a cluster workload manager, Slurm has three key functions. First, it allocates exclusive and/or non-exclusive access to resources (compute nodes) to users for some duration of time so they can perform work. Second, it provides a framework for starting, executing, and monitoring work (normally a parallel job) on the set of allocated nodes. Finally, it arbitrates contention for resources by managing a queue of pending work.

### Head Node

The `head` node is a login node and a method of authentication, no development work can occur on the `head node`.  There are `no accelerators` on the head node.  From the head node you can do file management, launch and manage SLURM jobs on the dedicated partition or launch an interactive job on one of the worker nodes. Your home directory is automatically mounted on the worker node when you connect.  It has a maximum of 20GB of storage.  There is a data directory where training and datasets can be stored.  `/home/common/data`

### Interactive Worker Nodes

#### Go interactive!

```bash
srun -p pvc-shared --pty bash
source /opt/intel/oneapi/setvars.sh
```

The `interactive worker nodes` are resource constrained in that they are shared resources, so please be courteous.  Running code on the interactive session is just like being on a local session and you can run code without submitting to a queue.  There is at least one PVC in each worker node.  For maximum peformance submit your job to the `pvc partition` which will run your code using all resources available and if your code can make use of `Intel(R) Data Center GPU Max 1100's` there are 4 in each `non interactive node`.  Keep in mind this is a one at a time job, so you might have to wait awhile.

--- 
## Environment Setup<div id='environment-setup'/>  

Enter `source /opt/intel/oneapi/setvars.sh` and the oneAPI development environment will be initialized.

Enter `conda env list` and activate the python environment of your choice.  Both Tensorflow and Pytorch environments have Jupyter installed.  If you don't like those environments create your own conda environment and customize to your liking.

## Jupyter<div id='jupyter'/>

This will get better, there are security issues that need to be overcome in future versions of IDC, for now these are the overview of steps to run `Jupyter-lab`:

1. Login to the head node.
2. Launch an interactive session
3. Find the IP of the interactive session.  
4. Activate a conda environment that has jupyter-lab
5. Launch jupyter-lab and take note of the port
6. From another terminal port forward to your localhost
7. Launch a browser and paste the long link from the other terminal tab
8. Have fun with Jupyter-lab!

The details, enter these from a terminal:

```bash
ssh idcbeta
srun --pty bash
echo $(ip a | grep -v -e "127.0.0.1" -e "inet6" | grep "inet" | awk {'print($2)}' | sed 's/\/.*//')
conda activate pytorch_xpu
jupyter-lab --ip 10.10.10.X
```
Take note of the IP and the port that jupyter launches on, it will look something like this:

```bash
http://10.10.10.8:8888/lab?token=9d83e1d8a0eb3ffed84fa3428aae01e592cab170a4119130
``` 

Your port will likely be different so replace `8888` with what was provided to you.  From a `new terminal` enter:

```bash
ssh idc -L 8888:10.10.10.X:8888
```

 Open your browser and enter `localhost:8888` or shift+click the link in the other terminal, or paste the token that was provided to you when you initialized the server as your password and use Jupyter lab as usual.

---   

## Additional Software<div id='additional-software'/>

It's possible to install additional software if regular user permissions are the only requirements.  For example to install <a href="https://www.intel.com/content/www/us/en/developer/articles/technical/get-started-with-intel-distribution-for-python.html">the Intel® Distribution for Python</a> follow these steps:  Miniconda is already installed, but you will be creating a virtual environment in your home directory.  

1.  `conda activate base`
2.  `conda update conda`
3.  `conda config --add channels intel`
4.  `conda create -n idp intelpython3_core python=3.10`
5.  `conda activate idp`

Keep in mind you have 20GB of storage in your home directory for all software and data.

---  
## Common Slurm Commands<div id='common-commands'/>

```bash
sinfo -al (What Nodes are available)
PARTITION AVAIL  TIMELIMIT   JOB_SIZE ROOT OVERSUBS     GROUPS  NODES       STATE NODELIST
pvc*         up    2:00:00          1   no       NO        all      3        idle idc-beta-batch-pvc-node-[01-03]

squeue -al (How many jobs are in the queue)
JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)

sbatch -p {PARTITION-NAME} {SCRIPT-NAME}
srun -p {PARTITION-NAME} {SCRIPT-NAME}
scancel {JOB-ID}

Go interactive with a compute node
srun -p {PARTITION-NAME} -n 1 -t 00-00:10 --pty bash -i (with time and specific node.)
srun --job-name "u-pick" --pty bash -i (First available.)
```

---  
## Sample GPU Test Code<div id='sample-gpu-test-code'/>

Here is a sample GPU test code that demonstrates functionality and how to offload the application execution to a `compute node`.  Follow the below steps:

Step 1.  Copy the code below into a file.

```C++
#include <sycl/sycl.hpp>
using namespace sycl;
int main() {
//# Create a device queue with device selector
  queue q(gpu_selector_v);
//# Print the device name
  std::cout << "Device: " << q.get_device().get_info<info::device::name>() << "\n";
  return 0;
}
```
Step 2.  Save the file as `getdev.cpp`

Step 3.  Enter the following commands to compile and run the application.

```bash
source /opt/intel/oneapi/setvars.sh

icpx -fsycl getdev.cpp

srun a.out
```

If successful it should return Device: Intel(R) Data Center GPU Max 1100.  Demonstrating that you successfully compiled a SYCL application and offloaded it's execution to a GPU on the compute node.

---  
## Some Example Scripts<div id='some-example-scripts'/>

```bash
#!/bin/bash
#SBATCH -A <account> 
#SBATCH -p pvc
#SBATCH --error=job.%J.err 
#SBATCH --output=job.%J.out

srun ./my_a.out
```

---  
## Where to get Support<div id='where-to-get-support'/>

Contact us at `slurm-pvc-uat@intel.com` this is an internal email address.  

---  
