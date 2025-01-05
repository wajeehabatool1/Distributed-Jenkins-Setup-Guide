# Distributed-Jenkins-Setup-Guide
![image](https://github.com/user-attachments/assets/f2da7ddb-37d9-4fc8-9ecb-c8ebe8dc6246)
---
## Overview
This repository offers a step-by-step guide for setting up a distributed Jenkins architecture. It explains how to configure both Jenkins Master and Worker nodes, set up SSH for secure communication, and provides the essential commands for a smooth deployment.

---
## Prerequisites
1. **One VM for the Jenkins Master**
2. **One VM for the Jenkins Slave/Worker**
---
## Installing & Configuring Jenkins on Master Node
1. Install java
``` bash
sudo apt update
sudo apt install openjdk-17-jre
```
2. Verify jaba installation
``` bash
java -version
```
3. Now, install jenkins
``` bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
4. Start Jenkins and check the status
``` bash
sudo systemctl start jenkins
sudo systemctl status jenkins
```
5. To ensure Jenkins starts on boot:
``` bash
sudo systemctl enable jenkins
```
6. Access jenkins on browser on : `http://<controller-vm-public-ip>:8080`
7. Find the Jenkins unlock key and open Jenkins in your browser
``` bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
8. Enter the initialAdminPassword
   
   ![image](https://github.com/user-attachments/assets/b16412e7-8c87-4afe-a7cc-0e4e33123040)
9. Click on install suggested plugin
    
   ![image](https://github.com/user-attachments/assets/4d9da132-6cf2-41ae-8a80-8fd4a6a5d6c3)
10. Create first admin url
    
    ![image](https://github.com/user-attachments/assets/fb86a363-934f-486a-8ba6-83b0a5019178)
11. Configure jenkins url
    
    ![image](https://github.com/user-attachments/assets/4cef2e68-e83b-4151-908e-0858adf36c3f)
---
## Configuring Slave/Worker
1. Ensure Java is installed (Jenkins requires Java to run)
   ``` bash
   sudo apt update
   sudo apt install openjdk-11-jdk -y
   ```
---
## Set Up SSH Key-Based Authentication
1. Generate an SSH key pair on jenkins master node
   ``` bash
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/jenkins_master_to_worker
   ```
2. This will create
   - `~/.ssh/jenkins_master_to_worker` (private key)
   - ` ~/.ssh/jenkins_master_to_worker.pub` (public key)
3. Before copying the public key to worker node, we have to make sure worker node allows ssh public key authentication and defines the path to file that holds the public keys
   - Open ssh daemon configuration file
     ``` bash
     sudo vi /etc/ssh/sshd_config
     ```
   - Uncomment these two fields
     ``` bash
     PubkeyAuthentication yes
     AuthorizedKeysFile .ssh/authorized_keys
     ```
   - Restart the ssh service
     ``` bash
     sudo systemctl restart ssh
     ```
4. now, we can copy public key to worker node in two ways
   - we can use `ssh-copy-id` command
     ``` bash
     ssh-copy-id -i ~/.ssh/jenkins_master_to_worker.pub username@worker_ip
     ```
   - we can manually copy the public key from the master and paste it into
     - copy the public key from the master node
       ``` bash
       cat ~/.ssh/jenkins_master_to_worker.pub
       ```
     - Open the authorized_keys file (a placeholder for the public key ) on the worker node and paste the copied public key in it
       ``` bash
        vi ~/.ssh/authorized_keys
       ```
  5. Test ssh connectivity
     ``` bash
     ssh -i ~/.ssh/jenkins_master_to_worker username@worker-ip
     ```
 ---
 ## Configure Jenkins Master to Connect with Worker
 1. Configure the Jenkins Master node to securely connect to the Worker node
    ``` bash
    sudo mkdir -p /var/lib/jenkins/.ssh
    sudo chown -R jenkins:jenkins /var/lib/jenkins/.ssh
    sudo cp ~/.ssh/known_hosts /var/lib/jenkins/.ssh
    ```
2. For jenkins to understand the private key into the desired format, convert the private key into pem format
   ``` bash
   ssh-keygen -p -m PEM -f ~/.ssh/jenkins_master_to_worker
   ```
3. On the worker node , create the following directory, we will use this directory to save the jenkins related logs and data
   ``` bash
   mkdir -p /home/wajeeha/jenkins_agent
   chmod 700 /home/wajeeha/jenkins_agent
   ```
4. Sets up the Credentials
   we will set credentials required by the master to connect to worker
   - Go to `Dashboard` -> `Manage Jenkins` -> `Credentials`
   - Select **Kind of Credentials as** : ` SSH Username with private key`
   - Make sure you add the **Username**  same as you **workernode username**
     
     ![image](https://github.com/user-attachments/assets/79b67b9d-e780-4405-a18c-54af370c619d)
5. Go to `Manage Jenkins` -> `Nodes` click on `New Node`
6. Name the agent , then click on `Permanent Agent`
   
   ![image](https://github.com/user-attachments/assets/cf267718-bd86-4df1-9a34-ebcf8672e9f3)
7. Remeber we created this `jenkins_agent` directory on the worker node , add the path to that directory in `remote root directory field`
   
   ![image](https://github.com/user-attachments/assets/15175409-ab04-4251-9919-de4e0627ade3)
8. Select `Lauch via ssh agent` method from `Launch method`
    
   ![image](https://github.com/user-attachments/assets/388ed465-fcaa-4d13-9ce3-57f329a24325)
  
9. add the **Host Ip** and selects the credential we have created for this purpose
10. Select `Known Host file Verification Strategy`
    
    ![image](https://github.com/user-attachments/assets/88bc109b-ebe5-405c-b58f-5f2bf2c818e7)
11. After saving changings, then jenkins master will start initializing ssh connection with the worker, click on the logs option to see the details
12. If you get this output saying `gent successfuly Connected`, it means you have successfuly connected the worker node with master node
    
    ![image](https://github.com/user-attachments/assets/496e8ad8-f640-48b9-9780-253e6728d852)





   



       









