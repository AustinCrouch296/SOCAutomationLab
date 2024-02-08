## Introduction
The primary objective of this lab is to create an automated Security Operations Center (SOC) environment that mirrors real-world incident detection, analysis, and response scenarios. Utilizing open-source software, this lab can be created for free, either deployed entirely on-premises or partially in the cloud (utilizing a trial Digital Ocean account). 

The key components of this lab are all free and open-source tools:
- Wazuh - A SIEM and XDR solution for event collection, analysis, and detection.
- Shuffle - A Security Orchestration, Automation, and Response (SOAR) platform that automates incident response processes.
- TheHive - A case management tool for efficient documentation and collaboration between SOC analysts.

By exploring automation's role in incident response, we can learn how to utilize these tools to accelerate threat detection, streamline SOC workflows, and also gain valuable insights into the operations that take place in a real SOC environment.

<div align="center">
  <img src="/images/soc-automation-diagram.png"></br>
  <b>Diagram:</b> Data flow through our Environment
</div>
<br/>

## Prerequisites
Before setting up the lab environment, ensure you have the following prerequisites:
- **Hypervisor**: You'll need a hypervisor to create virtual machines for your lab environment. I recommend using [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads).
- **Operation System Images**:
  - **Windows 10 ISO**: Download the [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10) to serve as our Client PC
  - Choose one of the following options:
    - **On-Premises Deployment**: Download the [Ubuntu 22.04.3 LTS ISO](https://ubuntu.com/download/desktop) for local installation.
    - **Cloud Deployment**: Sign up for a [Digital Ocean account](https://www.digitalocean.com/?refcode=e2ce5a05f701) to utilize their free $200 trial.
<br/>

## Understanding the Concepts:
### Wazuh Agent:
The Wazuh agent installed on our Windows 10 Client endpoint serves as our first line of defense by continuously monitoring the system for security events and log data. The agent encrypts and forwards this data securely to our central management server, the Wazuh Manager.

### Wazuh Manager:
Deployed in the cloud environment, the Wazuh Manager acts as a centralized hub for receiving, processing, and analyzing security events collected by the Wazuh agents. It analyzes the incoming data in real-time, using predefined rulesets and anomaly detection algorithms to identify potential security incidents. Upon detection, the Wazuh Manager triggers alerts and performs predefined responsive actions to mitigate threats promptly.

<br/>

<div align="center">
  <img src="/images/soc-automation-wazuh.png"></br>
  <b>Source:</b> <a href="https://documentation.wazuh.com/current/release-notes/release-4-3-0.html">Wazuh Documentation</a>
</div>

### Shuffle:
Shuffle serves as our Security Orchestration, Automation, and Response (SOAR) platform, plays a crucial role in automating incident response processes. Upon receiving alerts from the Wazuh Manager, Shuffle orchestrates various tasks:
- **Enrichment with OSINT**: Shuffle uses open-source intelligence (OSINT) to enhance it's understanding of detected indicators of compromise (IOCs).
- **Case Management with TheHive**: Shuffle integrates with TheHive to automatically create cases for each detected security incident. This allows for structured investigation, coordination, and documentation of response efforts.
- **Alert Notifications via Email**: Shuffle notifies the SOC Analysts about newly generated alerts. This ensures timely awareness and responsive actions.

### TheHive:
Operating as as cloud-based incident response and case management tool, TheHive streamlines the management of security incidents. It provides a centralized platform for SOC teams to collaborate, analyze, and respond to threats effectively.

<br/>

<div align="center">
  <img src="/images/soc-automation-thehive.png"></br>
  <b>Source:</b> <a href="https://github.com/TheHive-Project/TheHive/blob/main/images/Current_cases.png">TheHive</a>
</div>

### Response Workflow:
Upon receiving notification emails, SOC analysts interact with Shuffle to execute response actions. These actions encompass **remediation steps**, **communication with stakeholders**, and leveraging it as a as a **feedback loop** for continuous improvement.

<br/>

<div align="center">
  <img src="/images/soc-automation-workflow.png"></br>
  <b>Diagram:</b> SOC Automation Project Workflow</div>
</div>
<br/>

## Installation
This lab involves setting up three main devices:
- Windows 10 with Sysmon
- Wazuh Server
- TheHive Server

### Setting up Windows 10 Client with Sysmon

- Create the Windows 10 Virtual Machine (VM) using a [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10).
- Download [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) from Sysinternals and the [sysmonconfig.xml](https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml) file from this Github repository.
  - Extract Sysmon.zip and move sysmonconfig.xml into the folder.
  - Run PowerShell as Administrator, navigate to the Sysmon folder, and use the command `.\Sysmon64.exe -i .\sysmonconfig.xml`

<img src="/images/soc-automation-sysmon-download.png"></br>

You can verify that Sysmon was installed successfully with:
- **Services.msc**: Confirm the presence of Sysmon64
- **Event Manager**: Application and Services Logs > Microsoft > Windows > Sysmon

<img src="/images/soc-automation-sysmon-verify.png"></br>

### Wazuh Server
You can choose to either set this up in the cloud, or on a virtual machine:
#### Option 1. On Premises (Virtual Machines): 
Use Ubuntu 22.04 and follow the [Wazuh Quickstart Guide](https://documentation.wazuh.com/current/quickstart.html).
> *Wazuh uses a substantial amount of computing resources, it is recommended to use 4 vCPUs, 8GB RAM, and 50 GB of storage.*

#### Option 2. In the Cloud (Digital Ocean): 
Create a new Droplet (cloud server)
- OS: Ubuntu 22.04 (LTS), Basic CPU, 8 GB RAM
- Set a password, and change Hostname to 'Wazuh'

Create a Firewall:
> We will be accessing to this VM using SSH over the internet, so let's configure a firewall to mitigate potential security threats.
- Navigate to: Networking > Firewalls
- Create Inbound rules to limit Incoming TCP and UDP traffic to your IP address.

<br/>
<img src="/images/soc-automation-wazuh-firewall.png"></br>

Attach the Firewall:
- Navigate to Droplets > Wazuh > Networking > Firewalls, click Edit.
- Select the Firewall we created: Droplets > Add Droplets > "Wazuh".

SSH into the VM:
- Update packages:
		`apt-get update && apt-get upgrade -y`
- Install Wazuh with the curl command located on the [Wazuh Quickstart Guide](https://documentation.wazuh.com/current/quickstart.html). <br/>`curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a`

<br />

**We can now access the Wazuh Dashboard at https\://\[WAZUH-DROPLET-IP\]/ and use the credentials to sign-in.**

<img src="/images/soc-automation-wazuh-install-finish.png">
<img src="/images/soc-automation-wazuh-web-app.png">

### Install TheHive
You can choose to either set this up in the cloud, or on a virtual machine:
#### Option 1. Virtual Machine: 
Use Ubuntu 22.04 and follow this <a href="/TheHive-Install-Guide">guide</a> to download TheHive & pre-requisites.

#### Option 2. In the Cloud (Digital Ocean): 
Create a new Droplet (cloud server)
- OS: Ubuntu 22.04 (LTS), Basic CPU, 8 GB RAM
- Set a password, and change Hostname to 'TheHive'

Attach the Firewall:
- Navigate to Droplets > TheHive > Networking > Firewalls, click Edit.
- Select the Firewall we created: Droplets > Add Droplets > "TheHive".

SSH into the VM:
- Follow this <a href="/TheHive-Install-Guide">guide</a> for the commands to use to download TheHive and its pre-requisites. (Java, Cassandra, ElasticSearch, TheHive)

## Configurations
Coming soon!
