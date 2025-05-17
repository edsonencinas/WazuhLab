# Building Your Cybersecurity Homelab Project: Setting Up Wazuh Open-Source SIEM in Azure

### Introduction

Are you a cybersecurity enthusiast, student, or someone eager to learn more about security monitoring? Setting up a **Wazuh SIEM** in your own Azure homelab is a fantastic way to get hands-on experience with real-world security tools.

![azure_10](https://github.com/user-attachments/assets/74644e40-8e10-4ed5-978e-e2f2cc9d31b4)

In this step-by-step guide, we'll walk through how to deploy Wazuh in Azure, giving you a powerful security monitoring setup to practice detecting threats, analyzing logs, and understanding security operations — all in your own cloud environment!

## Why Set Up a Wazuh SIEM in Azure?

- **Learn security monitoring** in a controlled environment
- **Simulate attacks** and see how Wazuh detects them
- **Understand SIEM components** like indexer, server, dashboard, agents
- **Build skills** valuable for cybersecurity careers

## What You Need Before You Start?

- **Azure account** (Free tier is fine!  Once you sign up an account in Azure you will get $200 credits. Your free credits is more than enough for this project)
- Basic familiarity with Azure portal and Linux commands
- A computer with internet access
- Lots of Patience and curiosity!

## Step 1: Create Your Azure Virtual Machines (VMs)

**We'll need atleast three VMs for this project:**

- **Wazuh Indexer**: The central component that indexes and stores alerts from the wazuh server
- **Wazuh Dashboard**: Provides web interface for analyzing, mining, and visualizing data
- **Wazuh Server**: It analyzes the data received from the Wazuh agents
- **Wazuh Agent**: Installed on other machines or VMs to send logs, data to the server  
> Note: For this tutorial we will use only one VM for Wazuh Indexer and Wazuh Dashboard to optimize resources._
  
### 1.1 Create the Wazuh Indexer/Dashboard VM

1. Log in to https://portal.azure.com.
2. Create a Resource Group, select your free tier subscription, be descriptive with the resource group name ( I name it wazuh-homelab), select your preferred region, then click **Review + creatte**.
   
![azure_1](https://github.com/user-attachments/assets/03a7c2be-90f4-45cf-bdef-9c5ea536ac9e)

3. Next, it's important that the Wazuh Indexer, Wazuh Server, Wazuh Dashboard all sit in one virtual subnet, so we need to create one. Click Create resource then find Virtual network. Select the **wazuh-homelab** resource group and let's name it **wazuh-virtualsubnet** (Naming template is all up to you). We only need few private addresses, so let's change the default subnet to /28 notation (You need some basic knowledge of networking here).

![azure_2](https://github.com/user-attachments/assets/a37faf20-2873-422a-ba0b-5953e87883cf)
![azure_3](https://github.com/user-attachments/assets/6995751d-de3e-4840-8d23-d388cd1f29e8)


5. Let's create the VM for indexer and dashboard. As I said earlier we will install the indexer and the dashboard on the same VM. Click Create a resource > Virtual Machine.

### VM Instance details:
- Let's name the virtual machine, **wazuh-indexer-dashboard**
- Be consistent with the region selection: ((Asia Pacific) South East Asia)
- For the Availability options you  can select No infrastructure redundancy required or just select one availability zone on the list
- Image: Ubuntu Server 24.04 LTS - x64 Gen 2 (popular Linux distro)
- Size: Standard_B2als_v2 - 2vcpus, 4 GiB memory (minimum system requirement of wazuh)
- Authentication: I choose SSH public key but you can select password
  
![azure_4](https://github.com/user-attachments/assets/2d9f761a-d875-4ece-bc94-0e8d9787b8e3)

- Username: **vmadmin**
- We will connect to the VM through SSH later, so we need to open the inbound port 22 (SSH)
  
![azure_6](https://github.com/user-attachments/assets/24d1620c-c5cd-4a2e-b64d-4ffa73c55938)

### Disks
- Keep the default

### Networking
- **Virtual Network**: Let's select **wazuh-virtualnet** (created earlier)
- **Subnet** : This will be filled automatically, /28 will give us 14 usable IP addresses (more than enough)
- Leave everything as is
- Select **Review + create**
    
![azure_5](https://github.com/user-attachments/assets/a6a209f8-b27c-434e-a062-8f9ef65b2279)

- Click **create** again, Azure will now create and deploy the VM instance
- You will be prompted to download the SSH private key, download and keep it. (We need it later when we connect to this VM)

![azure_7](https://github.com/user-attachments/assets/bc1065cb-201e-4bd1-be78-8f1b9e40b4f2)

### 1.2 Create the Wazuh Server VM

- Repeat the above process for additional VM
- Let's name this VM **wazuh-server**   

### 1.3 Create the Wazuh Agent VM

- Repeat the above process in 1.1 for Wazuh Agent VM
- Image: I choose **Windows 10 Pro, version 22H2 - x64** this time (Wazuh agent can be installed in Linux, Windows, MacOS, etc..) 
- Let's name this VM **wazuh-agent**
  
## Step 2: Prepare The Wazuh Manager (Indexer, Server and Dashboard)

At this point we already have three VMs, **wazuh-indexer-dashboard**, **wazuh-server**, and **wazuh-agent**. Next, we will install wazuh components to this VMs. For more detailed instructions you can visit the [Wazuh Installation Guide](https://documentation.wazuh.com/current/installation-guide/index.html).

Wazuh Indexer Installation: 
1. Connect to **wazuh-indexer-dashboard** VM through SSH.
```
ssh -i '/path/to/keyfile' username@server`
```
Example:
```
ssh -i wazuh-indexer-dashboard-key.pem vmadmin@172.108.242.113`
```
2. Let's download the wazuh installation assistant and configuration file by executing the following command:

```
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh`</br>
curl -sO https://packages.wazuh.com/4.12/config.yml`
```
3. Using **nano** editor let's edit the `config.yml` and replace the nodes name and IP addresses.

```
nano /home/vmadmin/config.yml`
```

![azure_9](https://github.com/user-attachments/assets/c9aa1026-8aef-49b1-88f8-8b34c386f49c)

> You can find the private IP address of your VM in the Networking tab.

4. Let's execute the Wazuh installation assistant with the option `--generate-config-files`.  This command will generate the file `wazuh-install-files.tar` which contain the Wazuh cluster key, certificates, and passwords necesssary for installation.

> `sudo bash wazuh-install.sh --generate-config-files`

5. It's necessary to copy the `wazuh-install-files.tar` file to all the two VMs that we created earlier and place it inside your working folder. You can use the `scp` command to copy the file securely.
   
6. Let's run the Wazuh Installation assistant by including this option `--wazuh-indexer` and the name of the node (wazuh-indexer-dashboard). The node name must be the same in the `config.yml` file.

```
sudo bash wazuh-install.sh --wazuh-indexer wazuh-indexer-dashboard`
```

7. **Cluster Initialization**: Let's run the Wazuh installation assistant, to load the new certificates information and start the cluster.

```
sudo bash wazuh-install.sh --start-cluster`
```

8. To get the _admin password_, run the following command:

```
tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "\'admin\'" -A 1`
```

9. To confirm if the installation is succesfull, run the following command. Copy and paste the correct `<ADMIN_PASSWORD>` from the output and replace the `<WAZUH_INDEXER_IP>` (check the private IP address on the VMs Networking details).

```
curl -k -u admin:<ADMIN_PASSWORD> https://<WAZUH_INDEXER_IP>:9200`
```

10. Finally, run the following command to check if the cluster is working correctly:

```
curl -k -u admin:<ADMIN_PASSWORD> https://<WAZUH_INDEXER_IP>:9200/_cat/nodes?v`
```
## Step 3: Wazuh Server Installation

1. Connect to the wazuh-server VM through SSH.
2. Donwload the Wazuh installation assistant by running the following command.

```
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh`
```

4. Run the wazuh installation assistant followed by the node name. Double check the `config.yml`, and make sure the node name is the same.

```
sudo bash wazuh-install.sh --wazuh-server wazuh-server`
```

## Step 4: Wazuh Dashboard Installation

As I stress out ealier, we will install the Wazuh Dashboard in the `wazuh-indexer-dashboard` VM.

1. We already downloaded the Wazuh installation assistant earlier when we install the wazuh-indexer, so we are now ready to install the Wazuh dashboard.
2. Run the following command with the node name of the first VM we created.

```
sudo bash wazuh-install.sh --wazuh-dashboard wazuh-indexer-dashboard`
```

3. After the installation, the output shows the access credentials and a message of successful installation.
4. Extract the `wazuh-install-files.tar` and check the passwords generated by the installation assistant in the `wazuh-passwords.txt` file. Run the following command to print them.

```
tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt`
```

5. The Wazuh server utilize TCP port 1514 and 1515 for it to function. Let's create inbound rule in the **Networking** tab of the `wazuh-server` VM.

<img src="https://github.com/user-attachments/assets/7f2338db-1709-4fd2-b06e-351c15a1bc67" width="500">

6. Let's access the Wazuh dashboard using any browser. Use the `admin` user credentials from the `wazuh-passwords.txt` file.

> - **URL**: `https://<WAZUH_DASHBOARD_IP_ADDRESS>`
> - **Username**: `admin`
> - **Password**: `<ADMIN_PASSOWRD>`

A warning message will appear stating that the certificate was not issued by a trusted authority. We will accept it since this is only for lab purpose, however, you can add the certifcate in the advanced options of the web browser. The file `root-ca.pem` can be imported to the certificate manager of the browser. You can find this file in `/etc/wazuh-indexer/certs/` folder of the Wazuh indexer.

![azure_12](https://github.com/user-attachments/assets/b7193dc7-f7e1-4530-885a-da58acbac6d0)

## Step 5: Install Wazuh Agent

Wazuh agent is a multi-platform component of Wazuh. It runs on the endpoints (computers, servers, etc.) that you want to monitor. After you install Wazuh indexer, Wazuh server, and Wazuh dashboard you are now ready to install atleast one Wazuh agent to monitor.

1. Let's connect to our wazuh-agent VM (Windows) using RDP.
   
<img src="https://github.com/user-attachments/assets/9ad1abab-16f1-43b5-b3e8-376b4fa14d55" width="500">

2. Go to the **Endpoint** tab then click **Deploy new agent**.

![azure_16](https://github.com/user-attachments/assets/4ec2449a-9e94-457c-8d41-79e950863c8c)

3. Let's select Windows as it is the OS of the first Wazuh agent we configured earlier. Follow the instructions and fill-up the needed information.

![azure_15](https://github.com/user-attachments/assets/0341e357-dcb0-445c-a902-97c8b7f88413)

- **Server Address**: This is the Public IP address of the wazuh-server.
- **Assign an agent name**: (Optional) It uses the hostname of the agent name by default but you can use different agent name.
- **Select one or more existing groups**: You can ccreate group in the Agent management tab.

4. Open the Powershell in our Windows agent, In the **Run the following commands to download and install the agent** box, copy then paste it in the Powershell terminal.

<img src="https://github.com/user-attachments/assets/cb221141-371f-4ccc-8b30-ece80e811def" width="500">

5. Finally, start the agent by executing `NET START WazuhSvc`. 
 
## Step 6: Practice and Explore!

- Generate some logs by running commands or simulating attacks
- Check the logs on your Wazuh manager
- Learn how Wazuh detects suspicious activities
- Set up alerts and notifications (advanced)

### Final Tips

- Keep your Azure resources within a budget — use free tiers!
- Experiment with different types of logs and configurations
- Join online communities (like Wazuh forums) for help
- Always practice safely — avoid testing on production environments!

### Conclusion

Congratulations! You’ve built your own cybersecurity homelab in Azure with Wazuh SIEM. This setup is a fantastic sandbox to learn about security monitoring, log analysis, and incident detection — essential skills for any cybersecurity enthusiast.

Keep experimenting, learning, and exploring — the cybersecurity world is full of challenges and opportunities!


