# Slurm Cloud Bursting Using CycleCloud

This repository provides detailed instructions and scripts for setting up Slurm bursting using CycleCloud on Microsoft Azure, allowing you to seamlessly scale your Slurm cluster into the cloud for additional compute resources.

## Overview

Slurm bursting enables the extension of your on-premises Slurm cluster into Azure for flexible and scalable compute capacity. CycleCloud simplifies the management and provisioning of cloud resources, bridging your local infrastructure with cloud environments.


## Prerequisites  

Before proceeding, ensure you have the following requirements in place:  

- **Operating System:** `AlmaLinux HPC 8.x`, `Ubuntu HPC 20.04`, and `Ubuntu HPC 22.04` (Supported platform images in CycleCloud)  
- **CycleCloud Version:** `8.x`  
- **CycleCloud-Slurm Project:** `3.0.x`  
- **CycleCloud Server:** Must be up and running, and accessible via the `cyclecloud` CLI.

## Setup Instructions

### 1. On CycleCloud VM:


- Ensure CycleCloud VM is running and accessible via `cyclecloud` CLI.
- `cyclecloud initialize`
- Clone this repository and run the `sh 01_prep_cyclecloud-slurm_template.sh` script in the cyclecloud directory.

```bash
git clone https://github.com/vbalbarin/slurm-cloud-bursting-using-cyclecloud.git
cd slurm-cloud-bursting-using-cyclecloud/cyclecloud/
```

- Execute `01_prep_cyclecloud-slurm_template.sh` to generate the cluster template file.

```bash
sh 01_prep_cyclecloud-slurm_template.sh
```

Output :

```bash
[srvadmin@cyclecloud-vm cyclecloud]$ sh 01_prep_cyclecloud-slurm_template.sh 
Project version: 3.0.12
Slurm version:  24.05.4-2
Template location : slurm-3.0.12/templates/slurm-headless.txt
Please refer README for customizing the template for Headless Slurm cluster
[srvadmin@cyclecloud-vm cyclecloud]$ 
```

The above output shows the Cyclecloud-Slurm Project version available in your cyclecloud enviorment, supported slurm version in the project and the template for creating headless slurm cluster.

- Edit the template from the given template location (`Template location : slurm-3.0.12/templates/slurm-headless.txt`) using your favorite editor and make the following adjustment to create a headless template.
- Ensure that the `slurm` and `munge` UID and GID are included in the template under the `[[node defaults]]` and `[[[configuration]]]` sections if they are not already present. This ensures consistency with the scheduler's UID and GID for `munge` and `slurm`.

```bash    
[[node defaults]]     
    [[[configuration]]]
    ....
        #slum and munge users and setting their ids
        slurm.user.name = slurm
        slurm.user.uid = 11100
        slurm.user.gid = 11100
        munge.user.name = munge
        munge.user.uid = 11101
        munge.user.gid = 11101
```
- Remove the following sections completely in the template to prepare a headless template.

 ```bash    
        [[node scheduler]]
        [[nodearray scheduler-ha]]
        [[nodearray login]]
        [[[parameter SchedulerMachineType]]]
        [[[parameter loginMachineType]]]
        [[[parameter NumberLoginNodes]]]
        [[[parameter SchedulerHostName]]]
        [[[parameter SchedulerImageName]]]
        [[[parameter LoginImageName]]]
        [[[parameter SchedulerClusterInitSpecs]]]
        [[[parameter LoginClusterInitSpecs]]]
        [[[parameter SchedulerZone]]]
        [[[parameter SchedulerHAZone]]]
```


- Once the headless template is prepared then run `02_cyclecloud_build_cluster.sh` script to import the headless cluster to cyclecloud.
- in this example we use `hpc10` as the cluster name.

```bash
sh 02_cyclecloud_build_cluster.sh
```

Output:

```bash
[srvadmin@cyclecloud-vm cyclecloud]$ sh 02_cyclecloud_build_cluster.sh 
Enter Cluster Name: hpc10
Cluster Name: hpc10
Importing Cluster
Importing cluster Slurm_HL and creating cluster hpc10....
-----------
hpc10 : off
-----------
Resource group: 
Cluster nodes:
Total nodes: 0
Cluster Name: hpc10
Project version: 3.0.10
Slurm version:  24.05.4-2
[srvadmin@cyclecloud-vm cyclecloud]$ 
```

Please make a note of the Cluster Name, Project version and Slurm version. which you will be using the next steps.


### 2. Preparing Scheduler VM:

- Deploy a VM using the supported AlmaLinux HPC or Ubuntu HPC image.
- This example uses the Microsoft Ubuntu 22.04 LTS HPC image.
- Install Git
- Clone repositoery

```bash
git clone https://github.com/vbalbarin/slurm-cloud-bursting-using-cyclecloud.git
cd  slurm-cloud-bursting-using-cyclecloud/scheduler/
```

- Create a new test user in CycleCloud `hpcuser01`.
Leave the SSH public key for now. However, take note of the generated `uid`.
- In this example we are using the `useradd_example.sh` script to create a test user `hpcuser01` and group for job submission.

(Note, it would be preferable to use a centralized system such as LDAP/Active Directory to ensure consistent provisioning of UID and GID across the nodes.)

```bash
cd slurm-cloud-bursting-using-cyclecloud/scheduler
sudo sh useradd_example.sh
```
Output:

```bash
[root@scheduler scheduler]# sh useradd_example.sh 
Enter User Name [hpcadmin]: hpcuser01
Enter UID from CycleCloud [20001]: 20002
Generating public/private ed25519 key pair.
Your identification has been saved in /shared/home/hpcuser01/.ssh/id_ed25519
Your public key has been saved in /shared/home/hpcuser01/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:wFalNZlRTjwQWoZDlzFLJTBdKZLEsnDc8QSRLJu7J0g hpcuser01@scheduler-vm
The key's randomart image is:
+--[ED25519 256]--+
|     . *X@^&=.   |
|    ..=.X%*B+    |
|     o+*ooo...   |
|     .+.         |
|       .S        |
|    E .          |
|   . . .         |
|    . o .        |
|       o         |
+----[SHA256]-----+



New password: 
Retype new password: 
passwd: password updated successfully
```


- Run the Slurm scheduler installation script (`01_slurm-scheduler-builder.sh`) and provide the `cluster name` and the `slurm version` when prompted.
- Cluster Name and Slurm version should be same taken from the previous step.
- This script will install and configure Slurm Scheduler.

```bash
sudo sh 01_slurm-scheduler-builder.sh
```

Output: 

```bash
[root@scheduler scheduler]# sh 01_slurm-scheduler-builder.sh 
------------------------------------------------------------------------------------------------------------------------------
Building Slurm scheduler for cloud bursting with Azure CycleCloud
------------------------------------------------------------------------------------------------------------------------------
 
Enter Cluster Name: hpc10
Enter the Slurm version to install (You get this from the cyclecloud_build_cluster.sh): 24.05.4-2                                            
------------------------------------------------------------------------------------------------------------------------------
 
Summary of entered details:
--------------------------
Cluster Name: hpc10
Scheduler Hostname: scheduler
NFS Server IP Address: XX.XXX.X.X
 
 Please Note down the above details for configuring cyclecloud UI
------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------
Creating Munge and Slurm users
------------------------------------------------------------------------------------------------------------------------------
Munge and Slurm users created
------------------------------------------------------------------------------------------------------------------------------
Setting up NFS server
------------------------------------------------------------------------------------------------------------------------------
<--- truncated output --->
------------------------------------------------------------------------------------------------------------------------------
Slurm installed
------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------
Configuring Slurm
------------------------------------------------------------------------------------------------------------------------------
 
------------------------------------------------------------------------------------------------------------------------------
Slurm configured
------------------------------------------------------------------------------------------------------------------------------
 
------------------------------------------------------------------------------------------------------------------------------
 Go to CycleCloud Portal and edit the hpc10 cluster configuration to use the external scheduler and start the cluster.
 Use XX.XXX.X.X IP Address for File-system Mount for /sched and /shared in Network Attached Storage section in CycleCloud GUI 
 Once the cluster is started, proceed to run  cyclecloud-integrator.sh script to complete the integration with CycleCloud.
------------------------------------------------------------------------------------------------------------------------------
 
```

### 3. CycleCloud UI:

- Access the CycleCloud UI
- Update the public key for the `hpcuser01` under **Users** from the scheduler VM. (Click the "gear" icon.)
- Grant **Access** to the `hpcuser01`
- Edit the `hpc10` cluster settings.
- Under **Required Settings**
  - **Virtual Machines**: Select VM SKUs as per your requirements
  - **Networking**: Select the apppropriate subnet for **Subnet ID**.
- Under **Network Attached Storage**
    - Uncheck **Use Builtin NFS**.
    Enter the NFS server IP address for `/sched`
    - Uncheck **Use Builtin NFS**.
    Enter the NFS Server IP address for `/shared`
- Under **Advanced Settings**
    - Select the appropriate OS images for HPC or HTC nodearray.
    (Note, these should be the same OS image as that of the scheduler VM.)
    - Uncheck the `Return Proxy`.
- Save the settings.
- Start `hpc10` cluster

![NFS settings](images/NFSSettings.png)

### 4. On Slurm Scheduler Node:

- Integrate Slurm scheduler with CycleCloud using the `02_cyclecloud-integrator.sh` script.
- Provide CycleCloud details (username, password, and URL) when prompted.

```bash
cd  slurm-cloud-bursting-using-cyclecloud/scheduler/
sudo sh 02_cyclecloud-integrator.sh
```
Output:

```bash
[root@scheduler scheduler]# sh 02_cyclecloud-integrator.sh 
Please enter the CycleCloud details to integrate with the Slurm scheduler
 
Enter Cluster Name: hpc10
Enter CycleCloud Username: hpcadmin
Enter CycleCloud Password: 
Enter the Project version: 3.0.12
Enter CycleCloud URL (e.g., https://10.0.0.1): https://xx.xxx.x.x
------------------------------------------------------------------------------------------------------------------------------
 
Summary of entered details:
Cluster Name: hpc10
CycleCloud Username: hpcadmin
CycleCloud URL: https://xx.xxx.x.x
 
------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------
Configuring virtual enviornment and Activating Python virtual environment
------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------------------------------------------------------------
```


### 6. Testing & Job Submission:

- Log in as a test user (`hpcuser01` in this example) on the scheduler node.
- Submit a test job to verify the setup.

```bash
su - hpcuser01
```

```bash
tee hello_world.sh<<'_EOF'
#!/usr/bin/env bash
#SBATCH --job-name=hello_world      # Job name to identify
#SBATCH --output=hello_world_%j_%N.out    # Output file
#SBATCH --partition=hpc                    # Partition name to use
#SBATCH --nodes=1                            # Number of nodes 
#SBATCH --ntasks=1                            # Number of tasks 
#SBATCH --time=00:05:00                   # Max runtime (HH:MM:SS)

echo "Hello World from $SLURMD_NODENAME"
_EOF
```

```bash
sbatch hello_world.sh
```

Output:

```bash

[srvadmin@scheduler scheduler]# su - hpcuser01
Last login: Tue Feb 11 08:55:50 UTC 2025 on pts/0
[hpcuser01@scheduler ~]$ sbatch hello_world.sh
Submitted batch job 1
[hpcuser01@scheduler ~]$ squeue 
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                 1       hpc hostname    hpcuser01 CF       0:03      1 hpc10-hpc-1
[hpcuser01@scheduler ~]$ 
```

In the CycleCloud web page, you should see a node spinning up:

![Node Creation](images/nodecreation.png)

Upon completion:

```bash
hpcuser01@scheduler:~$ ls
```

Output:

```bash
hello_world.sh hello_world_1_hpc10-hpc-1.out
```

```bash
cat hello_world_1_hpc10-hpc-1.out
```

Output:

```bash
Hello World from hpc10-hpc-1
```

For further details and advanced configurations, refer to the scripts and documentation within this repository.

---

These instructions provide a comprehensive guide for setting up Slurm bursting with CycleCloud on Azure. If you encounter any issues or have questions, please refer to the provided scripts and documentation for troubleshooting steps. Happy bursting!

**NOTE:** Lockdown environment need additional changes in the way we use the project and configure it. These are tested in the non-lockdown scenarios.
