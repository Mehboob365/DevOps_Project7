**DEVOPS TOOLING WEBSITE SOLUTION**

  **STEP 1 – PREPARE NFS SERVER**

    **Step 1 – Prepare NFS Server**
    
1. Spin up a new EC2 instance with RHEL Linux 8 Operating System with 4 instances. 
   
   Name them as NFS, Web1, Web2 and Web3.
 
![image](https://user-images.githubusercontent.com/67065306/134575378-64ee6245-bca1-47e4-847d-95ebee5bfe34.png)
 
2. Note the Availability Zone (AZ) for NFS server and create a volume in the same AZ. Attaching the volume to NFS Server.

3. Next, will connect to the NFS Linux Server with windows terminal.

![image](https://user-images.githubusercontent.com/67065306/134576456-85ccbe4c-6e5e-4df7-bdab-174831cfb452.png)

4. Run the command lsblk to show the Physical Disk added, i.e. xvdf

![image](https://user-images.githubusercontent.com/67065306/134576732-64a250a6-b47a-4894-bd33-bca0fcaa1884.png)

5. Next, we will create a partition of the physical disk by running the following command;

     sudo gdisk /dev/xvdf/
     
  ![image](https://user-images.githubusercontent.com/67065306/134577509-7907ed58-9725-41f7-9a1f-fcf4fde5e22c.png)
  
  we can see the partition xvdf1 added to the disk xvdf;
  
  ![image](https://user-images.githubusercontent.com/67065306/134578843-b45861ff-5f40-40ce-b53c-4b73b0079231.png)


6. Next we will create physical volume on the partition created in the previous step.

  So before we use the pvcreate command, we need to install it as follows;
  
   i).  sudo yum install lvm2 -y
   
 ![image](https://user-images.githubusercontent.com/67065306/134579454-fa3ee3a7-f2a2-43b3-b51d-77348fb4ec52.png)

   ii). sudo pvcreate /dev/xvdf1   (This creates a physical volume on the partition xvdf1)
   
 ![image](https://user-images.githubusercontent.com/67065306/134579660-2ef033d1-9355-4dd7-9a05-471751f01428.png)

   iii). sudo pvs  (This will show the physical volume created)
 
 ![image](https://user-images.githubusercontent.com/67065306/134579844-5442fb9e-e0b4-4977-8987-2466ac945983.png)

  iv). Next, we will create a volume group as follows;
  
       sudo vgcreate vg-webdata /dev/xvdf1
       
 ![image](https://user-images.githubusercontent.com/67065306/134580072-fa427c38-b689-43c1-887d-da088fe02fa5.png)

 v). Next we will create 3 logical volumes  as follows
 
     sudo lvcreate -n lv-apps -L 2G vg-webdata
 
     sudo lvcreate -n lv-logs -L 2G vg-webdata
     
     sudo lvcreate -n lv-opts -l 100%FREE vg-webdata
     
   ![image](https://user-images.githubusercontent.com/67065306/134581696-bc4d0dd0-88a2-4925-90cc-5f93a8628395.png)

Vi). To show the status of the logical volumes, will run the following;

    sudo lvs
    
    ![image](https://user-images.githubusercontent.com/67065306/134582129-8cf6846a-1011-43b4-94ca-d30543b450e0.png)

vii). Next we will create a new filesystem using the mkfs.xfs command


     
8. 
  2. Based on our LVM experience from Project 6, Configure LVM on the Server.
    
   Instead of formating the disks as ext4 you will have to format them as xfs
   
   Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs
   
   Create mount points on /mnt directory for the logical volumes as follow:
   
   Mount lv-apps on /mnt/apps – To be used by webservers
   
   Mount lv-logs on /mnt/logs – To be used by webserver logs
   
   Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

   
