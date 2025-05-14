# Building Your Cybersecurity Homelab Project: Setting Up Wazuh Open-Source SIEM in Azure

### Introduction

Are you a cybersecurity enthusiast, student, or someone eager to learn more about security monitoring? Setting up a **Wazuh SIEM** in your own Azure homelab is a fantastic way to get hands-on experience with real-world security tools.

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

**We'll need a couple of VMs:**

- **Wazuh Indexer**: The centra component that indexes and stores alerts from the wazuh server
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

- Click **create** again, after that Azure will create and deploy the VM instance
- You will be prompted to download the SSH private key, download and keep it. (We need it later during wazuh installation)

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

2.1 Connect to **wazuh-indexer-dashboard** VM

Open your terminal or command prompt:

> ssh -i '/path/to/keyfile' username@server

Example:
`ssh -i wazuh-indexer-dashboard-key.pem vmadmin@172.108.242.113`

2.2 Install Wazuh Indexer

Run these commands to install Wazuh:

`curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh`
`curl -sO https://packages.wazuh.com/4.12/config.yml`

Update packages
sudo apt update && sudo apt upgrade -y

Add Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update

Install Wazuh Manager
sudo apt install wazuh-manager -y

Start Wazuh Manager
sudo systemctl start wazuh-manager
sudo systemctl enable wazuh-manager

Your Wazuh Manager is now running!

## Step 3: Install Wazuh Agent on Other Machines

On each machine you want to monitor:

ssh your-username@

Prepare the system
sudo apt update && sudo apt upgrade -y

Add Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update

Install Wazuh Agent
sudo apt install wazuh-agent -y

3.1 Register the Agent

On the agent VM:

sudo /var/ossec/bin/agent-auth -m

Follow the prompts. This links the agent to the manager.

3.2 Start the Agent

sudo systemctl start wazuh-agent
sudo systemctl enable wazuh-agent

## Step 4: Access and Use Your Wazuh Dashboard

While setting up a full web dashboard can be complex, you can:

Use Wazuh API to check logs
Install Wazuh Kibana plugin if you want a visual dashboard (advanced)

For now, you can verify logs and alerts via the command line, or experiment with installing Wazuh Dashboard on a separate VM.

## Step 5: Practice and Explore!

Generate some logs by running commands or simulating attacks
Check the logs on your Wazuh manager
Learn how Wazuh detects suspicious activities
Set up alerts and notifications (advanced)

### Final Tips

- Keep your Azure resources within a budget — use free tiers!
- Experiment with different types of logs and configurations
- Join online communities (like Wazuh forums) for help
- Always practice safely — avoid testing on production environments!

### Conclusion

Congratulations! You’ve built your own cybersecurity homelab in Azure with Wazuh SIEM. This setup is a fantastic sandbox to learn about security monitoring, log analysis, and incident detection — essential skills for any cybersecurity enthusiast.

Keep experimenting, learning, and exploring — the cybersecurity world is full of challenges and opportunities!

**Happy hunting!**
Your journey into cybersecurity mastery starts here.

