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
 
 
 9. **We will export the mounts for webservers’ subnet cidr to connect as clients**. 
    
    For simplicity, you will install all three Web Servers inside the same subnet, 
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
 
 
 ![image](https://user-images.githubusercontent.com/67065306/134594762-8b3b1dc5-56ad-424f-ac75-cd21c3416da2.png)


10. **We will check which port is used by NFS and open it using Security Groups (add new Inbound Rule)**

    rpcinfo -p | grep nfs
 
 ![image](https://user-images.githubusercontent.com/67065306/134595003-95ca2714-9160-4c46-bc82-c260b0dd10bb.png)

**Important note:** In order for NFS server to be accessible from our client, we must also open following ports: 
   
   TCP 111
   
   UDP 111
   
   UDP 2049
  
 ![image](https://user-images.githubusercontent.com/67065306/134595459-3d36b8b7-79c0-496d-8bf2-4de3428e7384.png)
 

**STEP 2 — CONFIGURE THE DATABASE SERVER**
   **Step 2 — Configure the database server**
   
**1. Install MySQL server**

    sudo apt update
    
    sudo apt install mysql-server -y
    
 ![image](https://user-images.githubusercontent.com/67065306/134681335-bdd0126f-3e3a-42b8-8375-b9988174f85f.png)
    
    sudo systemctl start mysql
    
    sudo systemctl enable mysql

![image](https://user-images.githubusercontent.com/67065306/134680967-0cb1446d-ca33-4000-894a-896c9ac880a5.png)

   sudo systemctl status mysql

![image](https://user-images.githubusercontent.com/67065306/134681760-96288b65-a47a-4224-b6e4-97e29debc4a9.png)

**2. Create a database and name it tooling**

    sudo mysql_secure_installation
    
 ![image](https://user-images.githubusercontent.com/67065306/134683153-793f26e2-d1fe-43da-9ed1-5171dfff9edd.png)

   sudo mysql
   
   mysql> create database tooling;
   
   mysql> show databases;

 ![image](https://user-images.githubusercontent.com/67065306/134685405-1ddc4110-4396-446c-accd-761d352235a5.png)

**3. Create a database user and name it webaccess**

      mysql> CREATE USER 'webaccess'@'%' IDENTIFIED WITH mysql_native_password BY 'M3xxxx';
      
      mysql> GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'%' WITH GRANT OPTION;

      mysql> flush priviliges;

![image](https://user-images.githubusercontent.com/67065306/134689254-07cadeb6-6297-439f-ae88-55537e2b194c.png)

![image](https://user-images.githubusercontent.com/67065306/134690275-f857a34b-7dde-41a4-9452-d1d1d7f5ff01.png)

**4. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr**

        sudo nano /etc/mysql/mysql.conf.d/mysql.cnf    edit the bind address to 0.0.0.0
        
        sudo systemctl restart mysql      
 
  ![image](https://user-images.githubusercontent.com/67065306/134695161-813e3190-9de9-4b49-acd5-5a5050cd93cf.png)

 
**Step 3 — Prepare the Web Servers**
   
We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.
We know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

During the next steps we will do following:

Configure NFS client (this step must be done on all three servers)

Deploy a Tooling application to our Web Servers into a shared NFS folder

Configure the Web Servers to work with a single MySQL database


1. We will Launch a new EC2 instance with RHEL 8 Operating System

2. Install NFS client

      sudo yum install nfs-utils nfs4-acl-tools -y
      
      sudo systemctl start nfs-server
      
      sudo systemctl enable nfs-server
      
      sudo systemctl status nfs-server
      
  ![image](https://user-images.githubusercontent.com/67065306/134740236-ad3f2fdc-007e-4b94-b84c-5c937ee727c3.png)
  
  3. sudo mkdir /var/www

  4. sudo mount -t nfs -o rw,nosuid 172.31.1.188:/mnt/apps /var/www

 ![image](https://user-images.githubusercontent.com/67065306/134740583-cef8c8dd-8c39-468e-bec5-14ce6f13d4d9.png)
 
 5. update fstab file with the entry below
 
     172.31.1.188:/mnt/apps  /var/www nfs defaults 0 0
 
 ![image](https://user-images.githubusercontent.com/67065306/134740858-9fa220fa-d437-439e-843c-8c68e6cb1dd2.png)


6. **install Apache**

      sudo yum install httpd -y
      
 ![image](https://user-images.githubusercontent.com/67065306/134741222-4a9d972e-4397-4cb9-9fef-ee858223ea33.png)
 
 Start and enable the httpd service.
 
 ![image](https://user-images.githubusercontent.com/67065306/134741391-da457768-ed6b-4e61-9792-83a9bd0fafff.png)

list the file, ls -l /var/www

![image](https://user-images.githubusercontent.com/67065306/134742072-79e94f20-6564-4c90-bea4-564c668e9697.png)

**Repeat steps 1-6 for another 2 Web Servers.**
 
 This is for web2 server
 
 ![image](https://user-images.githubusercontent.com/67065306/134745975-13f8c2ad-6a3e-47c0-bcb4-3c14aae4fcc4.png)

 This is for web3 server
 
 ![image](https://user-images.githubusercontent.com/67065306/134746029-5b163ec1-42cf-48f3-96b5-fad5b582b466.png)


7. We will Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. 
   If we see the same files – it means NFS is mounted correctly. We can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.
   
    on the webserver
   
![image](https://user-images.githubusercontent.com/67065306/134743702-7c730786-ba5b-47ca-a142-18c0cec1681c.png)

    on the NFS Server 
 
![image](https://user-images.githubusercontent.com/67065306/134743758-1ec7a145-1990-4fa0-8851-bc9f4a8b6f6e.png)


8. We will Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. 
   Repeat step №4 to make sure the mount point will persist after reboot.
   
   update vi /etc/fstab with the following entry.
    
       172.31.1.188:/mnt/apps /var/www nfs defaults 0 0
  
 9. We will fork the tooling source code from Darey.io Github Account to our Github account.
 
    git clone https://github.com/darey-io/tooling.git
    
    On web1 server
 
 ![image](https://user-images.githubusercontent.com/67065306/134769639-ba52246b-5dc7-4da4-9927-669d8110e628.png)
 
 This is what is in the tooling directory.
 
 ![image](https://user-images.githubusercontent.com/67065306/134769928-a79a3b50-3dcf-429b-85ed-7e4032d34230.png)

    On web2 server
 
 ![image](https://user-images.githubusercontent.com/67065306/134769672-0f000aba-a116-459b-a93c-68117d2b0dbe.png)

   On web3 server
   
 ![image](https://user-images.githubusercontent.com/67065306/134769696-12b2c12b-d2ad-47f4-8777-0898d4229a76.png)
 

 10. Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html
 
      Note 1: Do not forget to open TCP port 80 on the Web Server.
      
  ![image](https://user-images.githubusercontent.com/67065306/134770513-f9b99ed2-372f-490a-94db-6d7b0f0fcce1.png)

  ![image](https://user-images.githubusercontent.com/67065306/134770541-9c8d2b28-d221-4d0c-a46a-738e58ce0f73.png)
  
  ![image](https://user-images.githubusercontent.com/67065306/134770558-0401d211-4cb3-4585-92a6-db584859c1fd.png)


![image](https://user-images.githubusercontent.com/67065306/134772784-de640aa3-1ced-4d28-b3af-7b390d953421.png)

![image](https://user-images.githubusercontent.com/67065306/134772802-bfc06db0-d7dd-4baa-820e-4d2280a09c89.png)


The following error was reported for the httpd daemons service.

![image](https://user-images.githubusercontent.com/67065306/134776614-5038ee05-a8b5-446d-a6cb-d26708f08d1e.png)

Note 2: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0
        To make this change permanent – open following config file 
        
        sudo vi /etc/sysconfig/selinux and set SELINUX=disabled
        
  ![image](https://user-images.githubusercontent.com/67065306/134785341-8dc7311a-7c7f-4215-9701-5e853af77dd8.png)
  
  

        
 
              
              

      
      
      
      
      



   
