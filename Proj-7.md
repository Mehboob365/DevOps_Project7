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

  
7. 
  2. Based on our LVM experience from Project 6, Configure LVM on the Server.
    
   Instead of formating the disks as ext4 you will have to format them as xfs
   
   Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs
   
   Create mount points on /mnt directory for the logical volumes as follow:
   
   Mount lv-apps on /mnt/apps – To be used by webservers
   
   Mount lv-logs on /mnt/logs – To be used by webserver logs
   
   Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

   
