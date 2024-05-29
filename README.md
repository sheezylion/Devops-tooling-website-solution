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

<img width="974" alt="Screenshot 2024-05-29 at 15 43 51" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/b5a229e9-be83-4610-934a-51a0b6c5a2f2">

1. Export the mounts for Webservers' subnet cidr(IPv4 cidr) to connect as clients. For simplicity, all 3 Web Servers are installed in the same subnet but in production set up, each tier should be separated inside its own subnet or higher level of security. Set up permission that will allow the Web Servers to read, write and execute files on NFS.

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

Result:

<img width="814" alt="Screenshot 2024-05-29 at 15 50 39" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/30b7ef95-f20c-4b5c-8754-898c03746b8f">


2. Configure access to NFS for clients within the same subnet (example Subnet Cidr - 172.31.16.0/20)

Result:
<img width="1395" alt="Screenshot 2024-05-29 at 15 55 59" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/9dad5bd3-c5d1-43d3-8443-6ed77608c232">

```
sudo vi /etc/exports
```

Result:

<img width="644" alt="Screenshot 2024-05-29 at 15 57 48" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/506f9ac3-940d-498c-a735-e09346fdf327">

```
/mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
```

Result:

<img width="917" alt="Screenshot 2024-05-29 at 15 58 35" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/e19e68de-cbc8-45be-a7bb-14b2ac76d208">

```
sudo exportfs -arv
```

Result:

<img width="694" alt="Screenshot 2024-05-29 at 15 59 29" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/1805c8e4-7934-4ebb-8c28-009795279f86">

3. Check which port is used by NFS and open it using the security group (add new inbound rule)

```
rpcinfo -p | grep nfs
```

Result:

<img width="722" alt="Screenshot 2024-05-29 at 16 00 31" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/decccc4d-e319-47c3-8373-48775c00587b">

Note: For NFS Server to be accessible from the client, the following ports must be opened: TCP 111, UDP 111, UDP 2049, NFS 2049. Set the Web Server subnet cidr as the source

Result:

<img width="1363" alt="Screenshot 2024-05-29 at 16 05 25" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/d69d519e-e15b-4637-8467-718532a56d22">

### Step 4: Configure the Database Server
1. Launch an Ubuntu EC2 instance and name it - DB Server

Result:

<img width="1435" alt="Screenshot 2024-05-29 at 16 08 21" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/ef46b8d0-2835-4012-9d49-5062229f24a8">

2. Access the instance to begin configuration.

```
ssh -i ~/Downloads/demo-pair.pem ubuntu@54.196.165.213
```

Result:

<img width="842" alt="Screenshot 2024-05-29 at 16 10 55" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/99168fd0-f040-43e1-a10b-87f7aafe1f17">

3. Update and upgrade Ubuntu

```
sudo apt update && sudo apt upgrade -y
```

Result:

<img width="850" alt="Screenshot 2024-05-29 at 16 11 49" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/e6a9371f-85a0-4e8c-886e-e71692ba1de7">

4. Install mysql server

```
sudo apt install mysql-server
```

Result:

<img width="840" alt="Screenshot 2024-05-29 at 16 16 05" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/b1868053-b2d7-49e5-82e3-ec87398798bd">

5. Run mysql secure script

```
sudo mysql_secure_installation
```

Result:

<img width="842" alt="Screenshot 2024-05-29 at 16 20 00" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/3b690446-98de-4b88-8f67-592b968a2abe">

6. Create a database and name it tooling

7. Create a database user and name it webaccess

8. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

```
sudo mysql
```

Result:

<img width="727" alt="Screenshot 2024-05-29 at 16 22 38" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/347630f2-c483-4705-a3ea-3ac65c67b2a4">

```
CREATE DATABASE tooling;
CREATE USER 'webaccess'@'172.31.0.0/20' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.0.0/20' WITH GRANT OPTION;
FLUSH PRIVILEGES;
show databases;

use tooling;
select host, user from mysql.user;
exit
```

Result:

<img width="852" alt="Screenshot 2024-05-29 at 16 25 46" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/d4914bd5-1d95-4d4f-af24-111d7a047796">

9. Set Bind Address and restart MySQL

```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

Result:

<img width="849" alt="Screenshot 2024-05-29 at 16 28 06" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/0233fe54-d4d4-4aa8-b7e4-27f8f5888fa9">

```
sudo systemctl restart mysql
sudo systemctl status mysql
```

Result:

<img width="856" alt="Screenshot 2024-05-29 at 16 29 01" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/5cccb0ae-b957-4f47-852d-2b440c2069ca">

10. Open MySQL port 3306 on the DB Server EC2.

Result:

<img width="1398" alt="Screenshot 2024-05-29 at 16 31 37" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/bd66ae3a-2123-48c2-b596-334da760b285">

### Step 5: Prepare the Web Servers

There is need to ensure that the Web Servers can serve the same content from a shared storage solution, in this case - NFS and MySQL database. One DB can be accessed for read and write by multiple clients. For storing shared files that the Web Servers will use, NFS is utilized and previousely created Logical Volume lv-apps is mounted to the folder where Apache stores files to be served to the users (/var/www).

This approach makes the Web server stateless which means they can be replaced when needed and data (in the database and on NFS) integrtity is preserved

In further steps, the following was done:

- Configured NFS on all 3 web servers
- Deployed a tooling application to the Web Servers into a shared NFS folder
- Configured the Web Server to work with a single MySQL database

#### Web Server 1

1. Launch a new EC2 instance with RHEL Operating System

Result:

<img width="1410" alt="Screenshot 2024-05-29 at 16 35 59" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/c882ad07-6d8a-46fa-84b2-7191be6deedc">

2. Access the instance to begin configuration.

```
ssh -i ~/Downloads/demo-pair.pem ec2-user@34.203.245.68
```

Result:

<img width="565" alt="Screenshot 2024-05-29 at 16 37 50" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/641aea86-6ca2-4827-9f87-6fd9f5cc2d2c">

3. update the package manager

```
sudo yum update -y
```

4. Install NFS Client

```
sudo yum install nfs-utils nfs4-acl-tools -y
```

Result:

<img width="561" alt="Screenshot 2024-05-29 at 16 38 48" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/d0b11aaa-c19e-4fa7-b7e4-b8f740ea6164">

5. Mount /var/www/ and target the NFS server's export for apps. NFS Server private IP address = 172.31.29.32

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.29.32:/mnt/apps /var/www
```

6. Open the text editor using vi

   ```
   sudo vi /etc/fstab
   ```

Add the following line

```
172.31.29.32:/mnt/apps /var/www nfs defaults 0 0
```

Result:

<img width="570" alt="Screenshot 2024-05-29 at 16 57 36" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/91fa2ba4-5ca6-481c-980f-b67594d5cb83">

7. Install Remi's repoeitory, Apache and PHP

```
sudo yum install httpd -y
```

Result:

<img width="569" alt="Screenshot 2024-05-29 at 17 12 55" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/cef3e94a-01a4-4048-bfe8-67d910dc739d">

```
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

Result:

<img width="561" alt="Screenshot 2024-05-29 at 17 14 14" src="https://github.com/sheezylion/Devops-tooling-website-solution/assets/142250556/a327fcf5-16b6-4ade-a03a-ff1a0d86bc39">

```
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

```
sudo dnf module reset php
```
