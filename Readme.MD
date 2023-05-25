# Welcome to the Intel Dev Cloud

- [Welcome to the Intel Dev Cloud](#welcome-to-the-intel-dev-cloud)
  - [Sign Up](#sign-up)
  - [SSH Setup](#ssh-setup)
  - [Environment Setup](#environment-setup)
  - [Additional Software](#additional-software)
  - [Head Node vs Compute Nodes](#head-node-vs-compute-nodes)
  - [Common Commands](#common-commands)
  - [Sample GPU Test Code](#sample-gpu-test-code)
  - [VSCode](#vscode)
  - [Jupyter](#jupyter)
  - [Some Example Scripts](#some-example-scripts)
  - [Where to get Support](#where-to-get-support)

---  


## Sign Up<div id='sign-up'/>

### Beta Trial Open Now  

The Intel Developer Cloud (IDC) trial is open to prequalified Intel customers and approved developers.  

To get started: <a href="https://www.intel.com/content/www/us/en/secure/forms/developer/standard-registration.html?tgt=www.intel.com/content/www/us/en/secure/developer/devcloud/cloud-launchpad.html">Sign up for a standard Intel® Developer Zone account.</a>  You will need an RSA Public key, see the next section if you need more information on setting up your SSH authentication and client.

Explore the <a href="https://www.intel.com/content/www/us/en/secure/developer/devcloud/cloud-launchpad.html">service catalog.</a>

Schedule and deploy the service with <a href="https://scheduler.cloud.intel.com/">Intel® Developer Cloud management console.</a>

---  
## SSH Setup<div id='ssh-setup'/>

### ssh-keygen

ssh-keygen is a tool for creating new authentication key pairs for SSH. Such key pairs are used for automating logins, single sign-on, and for authenticating hosts.  The IDC uses SSH Keys exclusively and you will never use a password for authentication.

### RSA Public Key Creation

To create a key use the `ssh-keygen` utility found in your terminal application.  Windows powershell, Windows Subsytem for Linux (WSL) Terminal, Linux or MAC Terminal

For WSL, Linux and MAC clients enter the below command
```bash
ssh-keygen -f ~/.ssh/idc-key -t rsa -b 4096
```
For PowerShell enter:
```bash
 ssh-keygen -f C:\Users\YourID\.ssh\idc-key -t rsa -b 4096
 ```

This will result in two files being generated: `idc-key` and `idc-key.pub` take care of these files as they are the private and public key pairs that will be tied to your IDC account.

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

Ensure that `.ssh` has 600 privilege bits set  `-rw-------`  

Ensure that `config` has 600 privilege bits set  `-rw-------`  

Ensure that `.idc-key` and `idc-key.pub` have 400 privilege bits set `-r--------`  

In this configuration from the terminal future connections are established by entering:

```bash
ssh idc
```
You are allowed up to 2 connections to the IDC.  

---  
## Head Node vs Compute Nodes<div id='head-node-vs-compute-node'/>

Upon initial connection to the IDC, you are connected to the head node.  This environment is a standard Ubuntu 22.04.02 LTS environment including `dev-essential` and the <a href="https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit.html#gs.yh16wk">Intel oneAPI Basekit.</a>  This IDC utilizes <a href="https://slurm.schedmd.com/quickstart.html">SLURM</a> to manage job scheduling and resource management.   As a cluster workload manager, Slurm has three key functions. First, it allocates exclusive and/or non-exclusive access to resources (compute nodes) to users for some duration of time so they can perform work. Second, it provides a framework for starting, executing, and monitoring work (normally a parallel job) on the set of allocated nodes. Finally, it arbitrates contention for resources by managing a queue of pending work.

The `head` node is a shared resource and typical development can be accomplished here.  There are `no accelerators` on the head node.  In addition, users are resource constrained via time, compute, memory and storage so for maximum peformance submit your job to the `compute node` which will run your code using all resources available and if your code can make use of `Intel(R) Data Center GPU Max 1100's` there are 4 in each `compute node`.  

---  
## Environment Setup<div id='environment-setup'/>  

Enter `source /opt/intel/oneapi/setvars.sh` and the development environment will be initialized.

You can build and run code on the head node and if you find a library that you need is not installed please jump to the support section and open a ticket.  The request will be evaluated and you will be notified if the IDC can support your request.    


---  
## Additional Software<div id='additional-software'/>

It's possible to install additional software if regular user permissions are the only requirements.  For example to install <a href="https://www.intel.com/content/www/us/en/developer/articles/technical/get-started-with-intel-distribution-for-python.html">the Intel® Distribution for Python</a> via <a href="https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh">miniconda</a> follow these steps:

1.  `Install miniconda`
2.  `conda update conda`
3.  `conda config --add channels intel`
4.  `conda create -n idp intelpython3_core python=3.10`
5.  `conda activate idp`

Keep in mind you have 10GB of storage in your home directory for all software and data.

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

## VSCode<div id='vscode'/>



VSCode Server has been implemented on the IDC. Many plugins have been enabled providing a capable development IDE. From the root of your home directory enter:

```bash
code
```
You will be presented with a dialogue like the below:

```bash
1. Read all steps before proceeding
2. Store the password and ssh command shown below
    password:
      xxxxxxxxxxxxxxxxxxxxxxxxxx
    ssh command:
      ssh uXXXXXX@idcbatch.eglb.intel.com -L 8001:localhost:8001
3. Hit Enter here and type exit to end this SSH session
4. Launch a new SSH session with stored ssh command
5. Browse on local system http://localhost:8001
```
If you setup the `.ssh/config` you can port forward to your local host by entering the following:

```bash
ssh idc -L8001:localhost:8001
```

---  
## Jupyter<div id='jupyter'/>

Interested in running Jupyterlab?  That is possible with some additional steps and because this is installed in your user directory you can choose any additional packages and configure Jupyterlab to your liking.  If you installed `miniconda` the only requirement follow these additional steps:

1. `conda create --clone base --name jupyter`
2. `conda activate jupyter`
3. `conda install -c conda-forge jupyterlab`
4. `jupyter lab --no-browser` <-- Make sure to start in ~/

If all went well you will be provided a port and a token.  To login you will need to port forward the remote session to a local session.  Follow these steps which depend on you having setup your ssh/.config correctly.

From a new terminal enter:

```bash
ssh idc -L 8888:localhost:8888
```
Your port will likely be different so replace `8888` with what was provided to you.  Open your browser and enter `localhost:8888` on the address bar.  Paste the token that was provided to you when you initialized the server as your password and use Jupyter lab as usual.

---  
## Some Example Scripts<div id='some-example-scripts'/>

```bash
#!/bin/bash
#SBATCH -A <account> 
#SBATCH -n 2
#SBATCH --time=00:05:00 
#SBATCH --error=job.%J.err 
#SBATCH --output=job.%J.out

srun ./my_a.out
```

---  
## Where to get Support<div id='where-to-get-support'/>

Contact us <a href="https://www.intel.com/content/www/us/en/support/contact-intel.html#support-intel-products_67709:59441:231482">here.</a>

---  




