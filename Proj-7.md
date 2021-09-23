**DEVOPS TOOLING WEBSITE SOLUTION**

  **STEP 1 – PREPARE NFS SERVER**

    **Step 1 – Prepare NFS Server**
    
1. **Spin up a new EC2 instance with RHEL Linux 8 Operating System with 4 instances.** 
   
   Name them as NFS, Web1, Web2 and Web3.
 
![image](https://user-images.githubusercontent.com/67065306/134575378-64ee6245-bca1-47e4-847d-95ebee5bfe34.png)
 
2. **Note the Availability Zone (AZ) for NFS server and create a volume in the same AZ. Attaching the volume to NFS Server.**

3. **Next, will connect to the NFS Linux Server with windows terminal.**

![image](https://user-images.githubusercontent.com/67065306/134576456-85ccbe4c-6e5e-4df7-bdab-174831cfb452.png)

4. **Run the command lsblk to show the Physical Disk added, i.e. xvdf**

![image](https://user-images.githubusercontent.com/67065306/134576732-64a250a6-b47a-4894-bd33-bca0fcaa1884.png)

5. **Next, we will create a partition of the physical disk by running the following command;**

     sudo gdisk /dev/xvdf/
     
  ![image](https://user-images.githubusercontent.com/67065306/134577509-7907ed58-9725-41f7-9a1f-fcf4fde5e22c.png)
  
  we can see the partition xvdf1 added to the disk xvdf;
  
  ![image](https://user-images.githubusercontent.com/67065306/134578843-b45861ff-5f40-40ce-b53c-4b73b0079231.png)


6. **Next we will create physical volume on the partition created in the previous step**.

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

        sudo mkfs.xfs /dev/vg-webdata/lv-apps

  ![image](https://user-images.githubusercontent.com/67065306/134582636-859742f0-7622-4a66-8a90-0dff02e698ee.png)
  
        sudo mkfs.xfs /dev/vg-webdata/lv-logs

  ![image](https://user-images.githubusercontent.com/67065306/134582690-c37b8de5-711d-4213-ac1a-40df02ac2ef2.png)
  
        sudo mkfs.xfs /dev/vg-webdata/lv-opts

  ![image](https://user-images.githubusercontent.com/67065306/134582778-2544fa1c-cfc6-4fae-9013-24c4109a31b8.png)

   
 viii). Next we will Create mount points on /mnt directory for the logical volumes as follow:
 
      First we will create 3 directories;sudo mkdir /mnt/logs
      
          sudo mkdir /mnt/apps
          
          sudo mkdir /mnt/logs
          
          sudo mkdir /mnt/opts
          
   ![image](https://user-images.githubusercontent.com/67065306/134584051-707b92f1-653d-4d8a-be32-68cbb2638c1e.png)

  We can see the 3 directories are owned by root.
  
  ix). Next we mount 
  
  Create mount points on /mnt directory for the logical volumes as follow:
    
     Mount lv-apps on /mnt/apps – (To be used by webservers)
    
     Mount lv-logs on /mnt/logs – (To be used by webserver logs)

     Mount lv-opt on /mnt/opt – (To be used by Jenkins server in Project 8)
  
           sudo mount /dev/vg-webdata/lv-apps /mnt/apps   
           
  ![image](https://user-images.githubusercontent.com/67065306/134588297-c7543c87-68fa-4536-9a34-2fc47a1cdde1.png)

           sudo mount /dev/vg-webdata/lv-logs /mnt/logs
           
  ![image](https://user-images.githubusercontent.com/67065306/134588374-4b3050a8-7386-4c51-baf9-4e5e20fd259b.png)
  
           sudo mount /dev/vg-webdata/lv-opts /mnt/opts
           
  ![image](https://user-images.githubusercontent.com/67065306/134588012-9d46df52-a6d3-46f7-8f83-ce5c4ba597e7.png)

 
 x. We will next run df -h to show the how they mounted.
 
 ![image](https://user-images.githubusercontent.com/67065306/134588733-7a03ceb7-b79b-4e83-ab7c-7d15b26513b1.png)


7. **Next, we will Install NFS server, configure it to start on reboot and make sure it is up and running**
   For the configuration to be saved after reboot, we need to add the details to the fstab file.
   i). We run the following to show the UUID of the devices;
  
      sudo blkid
   
 ![image](https://user-images.githubusercontent.com/67065306/134589939-7c744975-7059-4380-862c-2dc79d887c5e.png)

  ii). vi /etc/fstab 
  
  ![image](https://user-images.githubusercontent.com/67065306/134590694-5ce5f44f-2f08-4584-8d3b-13d50ba638bd.png)

 
 iii). Mount all filesystems (of the given types) mentioned in fstab
 
         sudo mount -a
        
 iv) Reload daemon
 
       sudo systemctl daemon-reload
       
  ![image](https://user-images.githubusercontent.com/67065306/134591218-dd55eebf-7280-4bd1-a8fd-f69e193ae3b3.png)

  v). Run the following to see the mounts and configs;
  
       df -h
       
  ![image](https://user-images.githubusercontent.com/67065306/134591599-3a476653-52ef-45af-97ff-da5896b721c0.png)

   
8. **Install NFS server, configure it to start on reboot and make sure it is u and running**

      sudo yum -y update 
      
      sudo yum install nfs-utils -y
      
      sudo systemctl start nfs-server.service
      
      sudo systemctl enable nfs-server.service
      
      sudo systemctl status nfs-server.service
      
 ![image](https://user-images.githubusercontent.com/67065306/134593061-08a43458-c49d-482b-9faa-affbc903b24d.png)
 
 
 9. We will export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install all three Web Servers inside the same subnet, 
    but in production set up we would want to separate each tier inside its own subnet for higher level of security.
    To check our subnet cidr – open EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:
    
 
 First we make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

        sudo chown -R nobody: /mnt/apps
        
        sudo chown -R nobody: /mnt/logs
        
        sudo chown -R nobody: /mnt/opts


        sudo chmod -R 777 /mnt/apps
        
        sudo chmod -R 777 /mnt/logs
        
        sudo chmod -R 777 /mnt/opts

      sudo systemctl restart nfs-server.service
      
  ![image](https://user-images.githubusercontent.com/67065306/134594124-3c5c9b78-e487-4b34-98f9-bd357201e866.png)

 
   Next we update /etc/exports file with subnet cidr.
    
    sudo vi /etc/exports
 
 
 
 
      
      

   
