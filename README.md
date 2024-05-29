# DevOps Tooling Website Solution
A DevOps team utilizes various tooling solutions in order to help the team carry out their day-to-day activities in managing, developing, testing, deploying and monitoring different projects.

![feature-23](https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/e971b8be-6513-4bc2-9399-135b7eb4d64d)

In this project, we will be implementing a DevOps solution that consists of the following components

## The pre-requisite for the projects is the following.

- Basic Understanding of Linux Commands

- AWS account login with EC2 instances

- Webserver Linux: Red Hat Enterprise Linux 9

- Database Server: On Ubuntu 24.04 + MySQL

- Storage Server: Red Hat Enterprise Linux 9 + NFS Server 7) Programming Language: PHP

- Code Repository : Git

The diagram below shows the architecture of the solution.

<img width="754" alt="3-Tier" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/3a1a611e-5d64-46df-b150-233c51f0ba53">

### Step 1: Prepare NFS Server

1. Ensure you log in with your details to your AWS console via the
   
2. Search EC2 on the console, click on launch instance to spin up an EC2 instance with RHEL Operating System

Result: 

<img width="786" alt="Screenshot 2024-05-29 at 15 04 35" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/dead9f44-c272-41d2-a620-05ae441d6a48">

 3. Select instance type as t2.micro, select key-pair that you've created, leave the network setting to the default settings. check the summary to double check then click on launch instance.

Results: 

<img width="779" alt="Screenshot 2024-05-29 at 15 07 37" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/2df43211-8968-4f23-8bb1-4eb07936c0b1">

<img width="1429" alt="Screenshot 2024-05-29 at 15 09 14" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/4ee28703-49ac-4df2-a72e-819cee3a985a">

### Step 2: Configure Logical volume management on the server

- Format the lvm as xfs
- Create 3 Logical volumes: lv-opt, lv-appa, lv-logs.
- Create mount points on /mnt directory for the logical volumes as follows:
   - Mount lv-apps on /mnt/apps - To be used by web servers
   - Mount lv-logs on /mnt/logs - To be used by web serveer logs
   - Mount lv-opt on /mnt/opt - To be used by Jenkins server in next project.
 
1. Create 3 volumes in the same AZ as the NFS Server ec2 each of 10GB and attache all 3 volumes one by one to the NFS Server.

Results:

<img width="788" alt="Screenshot 2024-05-29 at 15 12 54" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/60611c9e-82ae-49d8-800e-e8b8b145eb7a">

<img width="1431" alt="Screenshot 2024-05-29 at 15 14 50" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/acc1759f-3a05-4c71-a7b1-ab1b7a1c617e">

<img width="1679" alt="Screenshot 2024-05-29 at 15 20 09" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/2e55889e-048d-440d-af40-75a55099688b">

<img width="890" alt="Screenshot 2024-05-29 at 15 20 35" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/4add9d85-841e-4f8d-8374-93414bb6c287">

2. Open up the Linux terminal to begin configuration.

```
ssh -i ~/Downloads/demo-pair.pem ec2-user@54.91.44.29
```

Result: 

<img width="727" alt="Screenshot 2024-05-29 at 15 17 38" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/ee0ca5f6-56be-4bc9-ab5c-62e7126b03ba">

3. Use lsblk to inspect what block devices are attached to the server. All devices in Linux reside in /dev/ directory. Inspect with ls /dev/ and ensure all 3 newly created devices are there. Their name will likely be xvdf, xvdg and xvdh

```
lsblk
```

Result:

<img width="633" alt="Screenshot 2024-05-29 at 15 21 52" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/f2752e32-2be8-4284-b5a2-ef00502bd63c">

4. Use gdisk utility to create a single partition on each of the 3 disks

```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```
Type "n" to add a new partition, Choose 1 as the partition number Click the enter button for the first and last sector. Enter :8300 for the default file system as shown below

Results:

<img width="623" alt="Screenshot 2024-05-29 at 15 25 22" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/a5458b30-5354-41d0-838e-bf402113079c">

<img width="730" alt="Screenshot 2024-05-29 at 15 26 34" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/ecb24e0e-60ce-4c60-9c35-6524e6253534">

<img width="728" alt="Screenshot 2024-05-29 at 15 27 18" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/b6baac99-049c-4ee0-a32a-04ee3d774a61">

5. Use lsblk utility to view the newly configured partitions on each of the 3 disks

```
lsblk
```

Result:

<img width="598" alt="Screenshot 2024-05-29 at 15 28 24" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/3cdba6cc-d655-4b6f-ad39-32e21c0eb290">

6. Install lvm package

```
sudo yum install lvm2 -y
```

Result:

<img width="1680" alt="Screenshot 2024-05-29 at 15 30 04" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/fcf065fa-5e95-4799-9152-e4a1a0e0821a">

7. Use pvcreate utility to mark each of the 3 dicks as physical volumes (PVs) to be used by LVM. Verify that each of the volumes have been created successfully

```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
sudo pvs
```

Result:

<img width="734" alt="Screenshot 2024-05-29 at 15 31 25" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/7c73b9a0-c2d8-47cb-8536-0d269052791a">

8. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg. Verify that the VG has been created successfully

```
sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
sudo vgs
```

Result:

<img width="794" alt="Screenshot 2024-05-29 at 15 33 03" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/2302db22-0bda-41f1-aafa-74a31c656282">

9. Use lvcreate utility to create 3 logical volume, lv-apps, lv-logs and lv-opt. Verify that the logical volumes have been created successfully

```
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg
sudo lvs
```

Result:

<img width="783" alt="Screenshot 2024-05-29 at 15 34 10" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/0ab9b4c8-7ce9-4450-96b9-b1867bfcf1b8">

10. Verify the entire setup

```
sudo vgdisplay -v
```

Result:

<img width="961" alt="Screenshot 2024-05-29 at 15 35 23" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/2f660157-210f-4043-848e-a3effd18335f">


```
lsblk
```

Result:

<img width="747" alt="Screenshot 2024-05-29 at 15 36 11" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/95e767fe-7571-40c7-938e-52a8e2d668c1">

11. Use mkfs -t xfs to format the logical volumes instead of ext4 filesystem

```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```

Result:

<img width="817" alt="Screenshot 2024-05-29 at 15 37 16" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/f642fafa-c21a-4b34-a170-d9f3b5812be1">

12. Create mount point on /mnt directory

```
sudo mkdir /mnt/apps
sudo mkdir /mnt/logs
sudo mkdir /mnt/opt
sudo mount /dev/webdata-vg/lv-apps /mnt/apps
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```

### Step 3: Install NFS Server, configure it to start, reboot and ensure it is up and running.

```
sudo yum update -y
sudo yum install nfs-utils -y
```

Result:

<img width="1680" alt="Screenshot 2024-05-29 at 15 40 22" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/b6a876a0-c93f-440d-a126-f3c9e474bbd9">

```
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

Result:

