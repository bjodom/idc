# Batch Mode Access to PVC-Enabled SPR Systems in the Intel® Developer Cloud (IDC)

**This file can be read online (with drawings and pictures) at https://tinyurl.com/ReadmeIDC.**


- [Batch Mode Access to PVC-Enabled SPR Systems in the Intel® Developer Cloud](#batch-mode-access-to-pvc-enabled-spr-systems-in-the-intel-developer-cloud)
  - [Overview](#overview)
  - [Simple 1-2-3](#steps)
  - [Account Registration](#sign-up)
  - [SSH Setup](#ssh-setup)
    - [ssh-keygen](#ssh-keygen)
    - [SSH Public Key Creation](#ssh-public-key-creation)
    - [SSH .config Client Setup](#ssh-config-client-setup)
    - [For Intel Employees:  Yours should look like below:  This is assuming you are using WSL or Linux.](#for-intel-employees--yours-should-look-like-below--this-is-assuming-you-are-using-wsl-or-linux)
    - [For MingW64 users use below, line order matters.](#for-mingw64-users-use-below-line-order-matters)
  - [Head Node vs Compute Nodes](#head-node-vs-compute-nodes)
    - [Head Node](#head-node)
    - [Interactive Worker Nodes](#interactive-worker-nodes)
      - [Go interactive!](#go-interactive)
  - [Environment Setup](#environment-setup)
  - [Jupyter](#jupyter)
   - [Additional Software](#additional-software)
  - [Common Slurm Commands](#common-slurm-commands)
  - [Sample GPU Test Code](#sample-gpu-test-code)
  - [Some Example Scripts](#some-example-scripts)
  - [Running MPI](#mpi)
  - [If you use MobaXterm](#MobaXterm)
  - [Extend your access](#extend-access)
  - [Where to get Support](#where-to-get-support)
  - [Revisit This Page for Tips](#revisit-often)

--- 
## Overview<div id='overview'/>

The Intel Developer Cloud (IDC) trial is open to pre-qualified Intel customers, approved developers, and all Intel employees.  While we plan to formerly launch in the future, today you will already be gaining access to a powerful, highly functional, system, and nd one that can greatly benefit from you sharing your experinces so we can improve it.  In other words, it is not perfect and we would appreciate your help finding the rough edges.

The picture below ([most update version is online(https://tinyurl/ReadmeIDC.html)]) illustrates how to think of this system at a high level.

This is NOT a cluster, and it is not a system which will giv eyou root access.  The system is a shared system (please be respectful of others in not hogging resources unnecessarily), not a virtualized one.

This IS a system with round-the-clock access to systems with Intel GPU Max series GPUs (PVCs).

![High level view](https://github.com/jamesreinders/idc/assets/6556265/1a7a1392-1e0e-44d1-825f-b99d4ccb8458)

--- 
## Simple 1-2-3 steps <div id='steps'/>

This Readme files has a lot of detail, but you should start by focusing on only three things:

1. Get an account on Intel® Developer Zone.  If you have one, you do not need to create a new one.  If you need one, <a href="https://www.intel.com/content/www/us/en/secure/forms/developer/standard-registration.html?tgt=www.intel.com/content/www/us/en/secure/developer/devcloud/cloud-launchpad.html">sign up now</a> - it is free and instant.  Note: Intel employees also have an employee login option that is only usable internally (in the office, or externally via VPN) - just look for the "Employee Sign In" and click that instead of entering a username, etc.  If you are an Intel employee, you can create an account with any non-Intel email in order to sign in without being on the corporate network.
2. Have an Public-Private Authentication Key Pair that has ed25519 or RSA 4096 level of strength.  If you have one, you do not need to create a new one.  If you need one, follow [SSH Setup](#ssh-setup) instructions to create one (free and instant).  With your key, we recommend setting up your config file to make ssh easy (see [SSH .config Client Setup](#ssh-config-client-setup)).
3. Create and use the service - it is free and instant (no need to enter any payment information - no credit card needed). Everyone can schedule and deploy the service with <a href="https://scheduler.cloud.intel.com/">Intel® Developer Cloud management console.</a>  From there pick `Scheduled access - Intel® Max Series GPU (PVC) on 4th Gen Intel® Xeon® processors - 1100 series (4x)`.  Intel employees wanting to use their employee login, should get on the internal network (VPN) and go <a href="https://www.intel.com/content/www/us/en/developer/tools/devcloud/services.html">here</a> and click <a href="https://scheduler.cloud.intel.com/">Sign In</a>. 

These three steps will get you ON the system.  You'll find more instructions on setting up an environment, using Jupyter, and more later in this Readme.

---  
## Account Registration<div id='sign-up'/>

Here the process to get your account and access the service, described above, step-by-step.

To access the batch service, external users must register for an Intel® Developer Cloud user account, via the Sign Up button on the Intel® Developer Cloud landing page (http://cloud.intel.com) and follow the steps in the Intel Cloud registration process.
Intel employees can use their existing intel.com credentials to access the Intel® Developer Cloud portal and select the batch service via the "Employee Sign In" link on this same page.

![image](https://github.com/jamesreinders/idc/assets/6556265/519038c0-a13a-45b1-b7d2-5aadcfb50136)

To register, press the ' Create the Account' button, as indicated:

![image](https://github.com/jamesreinders/idc/assets/6556265/0088d805-ab12-4181-ba26-0688faef0bc5)

In the new registration screen, fill out the registration input fields and press "Next: Verify your email" button.

![image](https://github.com/jamesreinders/idc/assets/6556265/3f17e5ba-5645-44d4-a9b9-4107ab3e4d1d)

A confirmation is sent to the email address that was entered in the registration screen. The email contains a single-use code, illustrated below.  You should get the email within minutes, please look in SPAM and JUNK folders if you do not see it promptly.

![image](https://github.com/jamesreinders/idc/assets/6556265/f260217f-c079-48fc-9e20-631bd71b3cc8)

Enter the verification code in the field, as illustrated below and enter 'Create an account' button.

![image](https://github.com/jamesreinders/idc/assets/6556265/a48bdd36-05ef-47ee-8d97-f4ca9d27f67d)

Next, we proceed to https://scheduler.cloud.intel.com/#/systems to go to the cloud management console.

Here we need to make sure our SSH public key is in our profile.  Click the person/profile icon on the blue bar (NOT the one higher up on the same page).

![image](https://github.com/jamesreinders/idc/assets/6556265/0a42c806-a477-4364-a662-2fadc1e8b14e)

Paste in your public key

![image](https://github.com/jamesreinders/idc/assets/6556265/2add9951-5b84-4c57-99cd-046bf3ab0a37)

Click "Save Key" and then Select the "Instances" Tab.

![image](https://github.com/jamesreinders/idc/assets/6556265/c47ac23c-f762-4ab2-96f3-4f091981252e)

Check the "Scheduled..." instance, and click "Launch Instance"

![image](https://github.com/jamesreinders/idc/assets/6556265/d1e99388-ae61-4a04-988f-d56b8a6fb710)

Request access by clicking "Request Access."  Note: once you have an instance, this page would show you the information (username, etc.) in case you have forgotten it.

![image](https://github.com/jamesreinders/idc/assets/6556265/ed10f222-0ce9-4140-b4f1-2fa8e327924b)

Enter your organization/affiliation/company and an explanation.  Once your explanation is 35 characters long, you can click "Request Access" and you will be granted access immediately.

![image](https://github.com/jamesreinders/idc/assets/6556265/7ea81251-ca28-4f82-96a8-a6edc5732e8d)

Make note of you user ID, you will need it.

![image](https://github.com/jamesreinders/idc/assets/6556265/b924f350-d486-44d3-9e5d-45350e3c153c)

This is a good time to follow the [SSH .config Client Setup](#ssh-config-client-setup) instructions using your new user ID information.

Please, please, please be sure your .ssh file permissions are set correctly.  Failure to do so, is the NUMBER ONE REASON for FAILURE to be able to ssh to the instance.

Now you can ssh to the node.   "ssh myidc" (use the name you set in your ~/.ssh/config).

From here, the most likely thing you want to do is "srun --pty bash" to open a live session on a 4th Gen Xeon system with Intel Max GPUs (PVCs) - where you can compile and run code (including single node multirank MPI programs), launch Jupyter notebooks, and more!

---
## SSH Setup<div id='ssh-setup'/>

### ssh-keygen

ssh-keygen is a tool for creating new authentication key pairs for SSH. Such key pairs are used for automating logins, single sign-on, and for authenticating hosts.  The IDC uses SSH Keys exclusively and you will never use a password for authentication.

### SSH Public Key Creation

To create a key use the `ssh-keygen` utility found in your terminal application.  Windows powershell, Windows Subsystem for Linux (WSL) Terminal, Linux or MAC Terminal

For WSL, Linux and MAC clients enter the below command
```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519_idc -C "you@email.com"
```
For PowerShell enter:
```bash
ssh-keygen -o -a 100 -t ed25519 -f C:\Users\YourID\.ssh\id_ed25519_idc
```

The passphrase is optional and you can hit enter for no pass phrase.  This will result in two files being generated: `id_ed25519_idc` and `id_ed25519_idc.pub` take care of these files as they are the private and public key pair that will be tied to your IDC account.

### SSH .config Client Setup (assumes no proxy needed)

To make accessing the IDC convenient, it is recommended to setup a `.ssh\config` file.

```bash
Host myidc #←YOU CAN CALL IT ANYTHING
Hostname idcbetabatch.eglb.intel.com
User uXXXXXX #← Request "scheduled access" at https://scheduler.cloud.intel.com/#/systems" to get your user identifier.
IdentityFile ~/.ssh/id_ed25519_idc
#ProxyCommand /usr/bin/nc -x YourProxy:XXXX %h %p # Uncomment if necessary
ServerAliveInterval 60
ServerAliveCountMax 10
StrictHostKeyChecking no # Frequent changes in the setup are taking place now, this will help reduce the known hosts errors.
UserKnownHostsFile=/dev/null
```
### For Intel Employees (due to proxy - other proxy users might take note):  Yours should look like below:  This is assuming you are using WSL or Linux.

```bash
Host myidc
Hostname idcbetabatch.eglb.intel.com
User uXXXXXX #← Request "scheduled access" at https://scheduler.cloud.intel.com/#/systems" to get your user identifier.
IdentityFile ~/.ssh/id_ed25519_idc
ProxyCommand /usr/bin/nc -x proxy-dmz.intel.com:1080 %h %p
ServerAliveInterval 60
ServerAliveCountMax 10
StrictHostKeyChecking no
UserKnownHostsFile=/dev/null
```

### For MingW64 users use below, line order matters.

```bash
ProxyCommand "C:\Program Files\Git\mingw64\bin\connect.exe" -S proxy-dmz.intel.com:1080 %h %p
Host myidc
Hostname idcbetabatch.eglb.intel.com
User uXXXXXX #← Request "scheduled access" at https://scheduler.cloud.intel.com/#/systems" to get your user identifier.
IdentityFile ~/.ssh/id_ed25519_idc
ServerAliveInterval 60
ServerAliveCountMax 10
StrictHostKeyChecking no
UserKnownHostsFile=/dev/null
```

Ensure that `.ssh` has 600 privilege bits set  `-rw-------`  

Ensure that `config` has 600 privilege bits set  `-rw-------`  

Ensure that `~/.ssh/id_ed25519_idc` and `~/.ssh/id_ed25519_idc.pub` have 400 privilege bits set `-r--------`  

In this configuration from the terminal future connections are established by entering:

```bash
ssh myidc
```
You are allowed up to 4 connections to the IDC.  If you lose a connection and rejoin, you may want to look for other lost login processes and kill them off (look with ps -aux).

---  
## Head Node vs Compute Nodes<div id='head-node-vs-compute-node'/>

Upon initial connection to the IDC, you are connected to the head node.  This environment is a standard Ubuntu 22.04.02 LTS environment including `dev-essential` and the <a href="https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit.html#gs.yh16wk">Intel oneAPI Basekit.</a>  This IDC utilizes <a href="https://slurm.schedmd.com/quickstart.html">SLURM</a> to manage job scheduling and resource management.   As a cluster workload manager, SLURM has three key functions. First, it allocates exclusive and/or non-exclusive access to resources (compute nodes) to users for some duration of time so they can perform work. Second, it provides a framework for starting, executing, and monitoring work (normally a parallel job) on the set of allocated nodes. Finally, it arbitrates contention for resources by managing a queue of pending work.

### Head Node

The `head` node is a login node and a method of authentication, no development work can occur on the `head node`.  There are `no accelerators` on the head node.  From the head node you can do file management, launch and manage SLURM jobs on the dedicated partition or launch an interactive job on one of the worker nodes. Your home directory is automatically mounted on the worker node when you connect.  It has a maximum of 20GB of storage.  There is a data directory where training and datasets can be stored.  `/home/common/data`

### Interactive Worker Nodes

#### Go interactive!

```bash
srun -p pvc-shared --pty bash
source /opt/intel/oneapi/setvars.sh
```

The `interactive worker nodes` are resource constrained in that they are shared resources, so please be courteous.  Running code on the interactive session is just like being on a local session and you can run code without submitting to a queue.  There is at least one PVC in each worker node.  For maximum performance submit your job to the `pvc partition` which will run your code using all resources available and if your code can make use of `Intel(R) Data Center GPU Max 1100's` there are 4 in each `non interactive node`.  Keep in mind this is a one at a time job, so you might have to wait awhile.

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
ssh myidc
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
ssh myidc -L 8888:10.10.10.X:8888
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
## Running MPI<div id='mpi'/>

MPI is currently limited to a single node, and must be run without SLURM.  Since SLURM (srun) will be the default, you need to specify a different launcher using the -launcher option.

For instance - either of these should work:
```bash
mpirun -launcher ssh -n 128 ./a.out
mpirun -launcher fork -n 128 ./a.out
```

These are probably the same (ssh and fork), but honestly we don't know.  They seem to run in the same time.  Let us know if you decide one is a superior choice.

Visit the [MPI with SYCL example page](etc/MPI.md) for a quick example of how to get a SYCL Hello World from 40 different connections to GPUs (40 ranks).

---
## If you use MobaXterm<div id="MobaXterm">

If you like using ModaXterm, here are notes from a user (thank you Yuning!) on the steps to make it fully work:

Step 1: Get ip address  

echo $(ip a | grep -v -e "127.0.0.1" -e "inet6" | grep "inet" | awk {'print($2)}' | sed 's/\/.*//') 

example output: 10.10.10.8 

Step2: Select the tunneling tab in MobaXterm

![image](https://github.com/jamesreinders/idc/assets/6556265/9d3f1d5b-8171-41c0-a7bd-8fa5bebc9592)

Select your own private key 

![image](https://github.com/jamesreinders/idc/assets/6556265/0bd39143-398c-4977-a423-203d2b563b02)

Description automatically generated 

![image](https://github.com/jamesreinders/idc/assets/6556265/162da683-30d1-43a7-a309-452aac529293)

If you need a proxy (Intel employees do when on internal network or VPN) - set up proxy (Intel is “proxy-dmz.intel.com:1080”). 

![image](https://github.com/jamesreinders/idc/assets/6556265/1431243b-30b4-4e29-841f-dc8b4c959d45)

Edit this tunnel 

![image](https://github.com/jamesreinders/idc/assets/6556265/06dd5c30-f4ea-401f-b147-3ead4d051fc8)

Step3: Launch Jupyter notebook in MobaXterm (use the IP address 10.10.10.X you were assigned)

jupyter-lab --ip 10.10.10.X --no-browser   

---  
## Extend your access<div id='extend-access'/>

This is subject to change - here is where we are now:
1. The system should auto-extend your account if you are using it during the last week of your allocated time. If it is idle in the days before it expires, your account will disappear and we cannot restore it (asking would be futile).
2. In the final week, you can visit https://scheduler.cloud.intel.com and request an extension.  Ideally, a button will appear next ot the instance when you schedule the "View Instances" tab. Click it and fill in the form and submit. If nothing appears on the "View Instances" tab - then you need to go to the "Launch Instance" tab, check the box in frotn of "Scheduled access..." and click "Launch Instance." You should now see a button to request an extension, click it and follow instructions to submit a request.  If all else fails, please [request support - see instructions.](#where-to-get-support)

---    
## Where to get Support<div id='where-to-get-support'/>


We have a small team ramping to respond quickly: please go to the support address shown when you login and click ["Submit Service Request."](https://www.intel.com/content/www/us/en/support/contact-intel.html#support-intel-products_67709:59441:231482)

We really hope you will contact us with feedback and requests by clicking ["Submit Service Request"](https://www.intel.com/content/www/us/en/support/contact-intel.html#support-intel-products_67709:59441:231482) (goes to a small team) at [intel.com/content/www/us/en/support/contact-intel.html#support-intel-products_67709:59441:231482](https://www.intel.com/content/www/us/en/support/contact-intel.html#support-intel-products_67709:59441:231482).

You may ALSO send an email to [ReadmeIDC@intel.com](mailto:ReadmeIDC@intel.com) until July 27 for anything.  WARNING: James and Ben may be much slower to respond than the ticket system, but we may be able to help with tough questions quicker.  Feel free to send it both ways until July 27.  After July 27, please escalate to [ReadmeIDC@intel.com](mailto:ReadmeIDC@intel.com) if you do not get a response to your ticket within a day, or a plan for resolution within 3 days. Please reference your ticket numbers.

---  
## Revisit This Page for Tips<div id='revisit-often'/>

We are enhancing, extending, and refining daily!  Please check back at https://tinyurl.com/ReadmeIDC often for new tips, and inevitable changes as we get better together!

---    
