**DEVOPS TOOLING WEBSITE SOLUTION**

  **STEP 1 – PREPARE NFS SERVER**

    **Step 1 – Prepare NFS Server**
    
  1. Spin up a new EC2 instance with RHEL Linux 8 Operating System
 
 ![image](https://user-images.githubusercontent.com/67065306/133677156-b9f5a592-c6ab-48ba-913e-8a3d063843a6.png)

  2. Based on our LVM experience from Project 6, Configure LVM on the Server.
    
   Instead of formating the disks as ext4 you will have to format them as xfs
   
   Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs
   
   Create mount points on /mnt directory for the logical volumes as follow:
   
   Mount lv-apps on /mnt/apps – To be used by webservers
   
   Mount lv-logs on /mnt/logs – To be used by webserver logs
   
   Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

   
