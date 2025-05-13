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
2. Create a Resource Group, select your free tier subscription, be descriptive with the resource group name ( I name it wazuh-homelab), then click **Review + creatte**.
   
 ![azure-1](https://github.com/user-attachments/assets/387c12ee-f4d2-42d2-82f8-2f8e89821a1b)

3. Next, it's important that the Wazuh Indexer, Wazuh Server, Wazuh Dashboard all sit in one virtual subnet, so we need to create one.
  
  
5. Click Create a resource > Virtual Machine.
6. Choose Ubuntu Server 20.04 LTS (a popular Linux distro).
Fill in the details:
Name: Wazuh-Manager
Region: your choice
Size: e.g., Standard_B1s (enough for learning)
Authentication: SSH public key (or password)
Review and click Create.

### 1.2 Create the Wazuh Agent VM(s)

Repeat the above process for each additional VM you want (for example, a Windows machine or another Linux box to simulate a target system).

## Step 2: Prepare Your Wazuh Manager

2.1 Connect to your VM

Open your terminal or command prompt:

ssh your-username@

2.2 Install Wazuh Manager

Run these commands to install Wazuh:

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

