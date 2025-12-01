---
layout: post
title: Cloud-based simple SOC Lab (Google Cloud Platform)
subtitle: SOC Lab
categories: SOC
tags: [SOC, lab setup]
---

# 🛡️ SOC Lab on Google Cloud

Hola!! It's been a while!!
Welcome to my Security Operations Center (SOC) lab documentation. This guide walks through the setup of a cloud-based SOC lab using Google Cloud Platform [GCP](https://cloud.google.com), featuring attacker simulation, log collection, and SIEM analysis.

---

## 📘 Overview

**Purpose**:  
Build a functional SOC lab for:
- Threat simulation
- Log aggregation
- SIEM testing (Wazuh)

**Components**:
- Attacker VM (Kali Linux)
- SIEM VM (Wazuh)
- Victim VMs (Ubuntu & Windows)
- Custom VPC and firewall rules

---

## 🧱 Architecture Diagram

<img width="671" height="301" alt="Updated drawio" src="https://github.com/user-attachments/assets/36d6fe3a-4294-4707-a73e-d111c1712935" />

---

## 🌐 Networking Configuration

### VPC & Subnet

We'll use this to configure our network to allow the VMs communicate.

```yaml
VPC Name: soc-vpc
Subnet: soc-subnet
CIDR: 10.0.0.0/24
Region: us-central1
```

---

## 💻 Virtual Machines

### Attacker VM
```yaml
Name: attacker-vm
OS: Kali Linux
Tools: Nmap. Metasploit, etc.
```

### SIEM VM
```yaml
Name: siem-vm
OS: Ubuntu 22.04
Software: Wazuh
Log Input: Wazuh Agents, Syslog, HEC
```

### Victim VMs
```yaml
Name: ubuntu-victim
OS: Ubuntu 22.04
Agents: Filebeat, Wazuh agent
```
```yaml
Name: windows-victim
OS: Windows server 2022
Agents: Sysmon, Splunk Forwarder
```

---

## ⚙️ Setup Commands

First enable the compute engine API in the Google cloud shell
```bash
gcloud services enable compute.googleapis.com
```

### Create VPC & Subnet
```bash
gcloud compute networks create soc-vpc --subnet-mode=custom
gcloud compute networks subnets create soc-subnet \
  --network=soc-vpc \
  --region=us-central1 \
  --range=10.0.0.0/24
```
<img width="700" height="454" alt="Screenshot From 2025-09-06 19-14-53" src="https://github.com/user-attachments/assets/b1c5ac6a-8b5b-45ef-aff1-c8e30dd320b1" />

### Firewall Rules

First rule in place allows SSH conections to the VMs.
```bash
gcloud compute firewall-rules create allow-ssh \
  --network=soc-vpc \
  --allow=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=ssh-access
```

Next, we'll add the firewall rule to allow rdp access to our windows VM.
```bash
gcloud compute firewall-rules create allow-rdp \
  --network=soc-vpc \ 
  --allow=tcp:3389 \ 
  --source-ranges=0.0.0.0/0 \ 
  --target-tags=rdp-access
```

Finally, we add the rule to allow access to Wazuh when we install it.
```bash
gcloud compute firewall-rules create allow-splunk-ui \
  --network=soc-vpc \
  --allow=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=wazuh-access
```

### Create Attacker VM
```bash
gcloud compute instances create attacker-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --subnet=soc-subnet \
  --tags=ssh-access \
  --image-project=debian-cloud \
  --image-family=debian-12 \
  --provisioning-model=SPOT \
  --boot-disk-size=50GB
```

### Create SIEM VM
```bash
gcloud compute instances create siem-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --subnet=soc-subnet \
  --tags=ssh-access,wazuh-access \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=100GB
```

### Create Victim VMs
```bash
gcloud compute instances create ubuntu-victim \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --subnet=soc-subnet \
  --tags=ssh-access \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --provisioning-model=SPOT \
  --boot-disk-size=40GB
```
```bash
gcloud compute instances create windows-victim \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --subnet=soc-subnet \
  --tags=rdp-access \
  --image-family=windows-2022 \
  --image-project=windows-cloud \
  --provisioning-model=SPOT \
  --boot-disk-size=50GB
```
After the four VMs are created you should have all instances available in the Compute Engine page.

<img width="700" height="162" alt="Screenshot From 2025-09-08 12-10-24" src="https://github.com/user-attachments/assets/e63be246-f2b9-4359-91b9-f7416e7d5244" />

### Attack setup

Next, let's setup the attack framework on the Attacker VM. The choice was between Caldera and Atomic red team, but we'll use Caldera. You can reference the [Caldera Documentation](https://caldera.readthedocs.io/en/latest/index.html). You may ask, why Caldera. Imagine having to research, select and type out every single command for any exploit you want to execute on a victim...that requires a lot of work. With Caldera, exploitation and cleanup can be done easily with the use of agents. Once an agent is installed on a victim, numerous operations can be carried out on that victim remotely, making the attack actions easier. Let's start our setup.

Prerequisites
```yaml
Python 3.8 or later (with pip3)
NodeJS v16 or later (for Caldera v5)
```
```bash
sudo apt update
sudo apt install python3-pip nodejs npm git
git clone https://github.com/mitre/caldera.git --recursive
cd caldera
pip3 install -r requirements.txt
python3 server.py --insecure --build
```
In case you experience an error with the build, regarding magma or vue, this command helped me.
```bash
cd ~/caldera/plugins/magma
npm install vue@3.2.47 @vue/compiler-sfc@3.2.47
npm run build
cd ~/caldera
python3 server.py --insecure
```

Since we cannot no rdp into our Attacker VM, we need to create another firewall rule to allow traffic over port 8888. This port is used by Caldera UI. Enabling this rule would allow us to access the UI on our browser.
```bash
gcloud compute firewall-rules create allow-caldera-ui \
  --network=soc-vpc \
  --allow=tcp:8888 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=attacker-access
```
```bash
gcloud compute instances add-tags attacker-vm \
  --zone=us-central1-a \
  --tags=attacker-access
```
Now we have our Attacker VM set. Next up, we'll install and setup our SIEM.

P.S Getting Caldera to work fully was a "little" frustrating and the fix was only a distro change away 😮‍💨

### SIEM Setup
We'll start with installing the SIEM. Using the quickstart guide provided by wazuh

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
Once the assistant finishes the installation, the output shows the access credentials and a message that confirms that the installation was successful.

After that we can install and configure the agents on the ubuntu and windows VMs.
For the Ubuntu machine, run these commands
```bash
apt-get install gnupg apt-transport-https
```
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```
```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```
```bash
apt-get update
```
```bash
WAZUH_MANAGER="siem-vm-ip" apt-get install wazuh-agent
```
```bash
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

For Windows, you can download the installer [here](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.1-1.msi). Run the installer on the windows machine and follow the prompts, adding the SIEM VM's IP address in the appriopriate field. You can confirm the agents are properly configured by checking the endpoints summary tab on Wazuh dashboard.

Next, we install a few extra things to make the experience a bit more interesting. We're going to add TheHive and Cortex to the setup so we can assign and investigate alerts and run jobs using analyzers and responders. I tried numerous ways to install TheHive and Cortex with so may failures. I eventually settled on using docker by creating a docker compose file, you can find that [here](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/blob/main/assets/docker-compose.yml). The only change you'll have to make is linking the application.conf file for cortex. Once you save the file in a directory, run this command `sudo docker-compose up`. The command tuns for a while and when Cortex and TheHive start receiving requests through the API, we can confirm the install is done. 

The next step is integrating Cortex and TheHive. Login to Cortex, create an organisation with a user and afterwards create an API key attached to that user. Next, login to TheHive dashboard with the admin account. Head to Platform Management and then to the Cortex tab. Add a Cortex server with these details
```yaml
Server name - CORTEX-SERVER
Server URL - http://siem-vm-ip:9001
API Key - <The API Key you just created in Cortex>
Leave the remaining settings as they are
```
You can then test server connection to confirm your settings are correct.

Next we connect TheHive to Wazuh so we can get alerts in TheHive. Now, we create an organisation on TheHive with a user and then generate an API key to use for this integration. Then we edit the ossec config file
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Next we add the code into the <ossec_config> block preferrably under the global tag
```yaml
<integration>
  <name>thehive</name>
  <hook_url>http://<THEHIVE_IP>:9000/api/alert</hook_url>
  <level>10</level>
  <api_key>Your_TheHive_API_Key</api_key>
  <alert_format>json</alert_format>
</integration>
```
This ensures that the only alerts that pop up on TheHive are medium severity to prevent low severity/information alerts. This seals up the integration.

The next thing we need to do is setup the firewall rules for the Wazuh agents to communicate with the server.
```bash
gcloud compute firewall-rules create allow-wazuh-agents \
  --network=soc-vpc \
  --direction=INGRESS \
  --priority=1000 \
  --action=ALLOW \
  --rules=tcp:1514,tcp:1515,udp:1514 \
  --source-ranges=<Internal IP addresses of the victim VMs separated by commas> \
  --target-tags=wazuh-server \
  --description="Allow Wazuh agents to communicate with Wazuh Manager on the default network"
```
Next we give access to TheHive and Cortex
```bash
gcloud compute firewall-rules create allow-cortex \
  --network=soc-vpc \
  --allow=tcp:9001 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=cortex-access

gcloud compute firewall-rules create allow-thehive \
  --network=soc-vpc \
  --allow=tcp:9000 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=thehive-access
```
Next
```bash
gcloud compute instances add-tags siem-vm \
--tags=wazuh-server,thehive-access,cortex-access,wazuh-access
```
That is a majority of the firewall rules done.

Now I ran into another challenge. the lab wasn't really secure. The firewall rules were not doing enough to restrict access to the VMs. I realized that the next best option was to secure all access with IAP.

#### Now what is IAP?
IAP which stands for Identity-Aware Proxy, is a Google Cloud security service that controls access to your applications and VMs based on user identity and context, instead of exposing them directly to the internet. In other words, you can access web apps and VMs without using an external IP address. With this security control in place, we can securely access the VMs in our setup without having to worry about any external unauthorized users trying to gain access to the machines.

Here's how we'll set ours up. We'll ensure that a google account of our choosing has the appriopriate access via IAM i.e. Identity and Access Management (P.S IAP is one of the solutions that IAM brings to the table)

First, we'll enable IAP on the project
```bash
gcloud services enable iap.googleapis.com
```
Next, we enable OS Login on the VMs
```bash
gcloud compute project-info add-metadata \
  --metadata enable-oslogin=TRUE
```
Then, we grant the approved users the appriopriate roles
```bash
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="user:approveduser@gmail.com" \
  --role="roles/compute.osAdminLogin"

gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="user:approveduser@gmail.com" \
  --role="roles/iap.tunnelResourceAccessor"
```
We also need to create the firewall rules for IAP to work properly
```bash
gcloud compute firewall-rules create iap-access \
  --network=soc-vpc \
  --priority=1000 \
  --direction=INGRESS \
  --action=ALLOW \
  --target-tags=iap-access \
  --source-ranges=35.235.240.0/20 \
  --rules=tcp
```
We can then tag our VMs
```bash
gcloud compute instances add-tags vm-name \
  --tags=iap-access \
  --zone=us-central1-a
```
Finally, to completely restrict access via IAP only, we're going to remove the external IP addresses.
```bash
gcloud compute instances delete-access-config VM_NAME \
    --access-config-name="External NAT" \
    --zone=ZONE
```
If you're not sure about teh access config name, run this command
```bash
gcloud compute instances describe VM_NAME --zone=ZONE
```
And there you have it, the lab is secure and ready to run.

To connect to the VMs via SSH, you can run this command
```bash
gcloud compute ssh vm-name --zone=ZONE --tunnel-through-iap
```
To access the tools like Caldera, Wazuh, TheHive and Cortex we would need to use an IAP tunnel. Once this is done, these services can be accessed locally via localhost. For example, to access TheHive this is the command to run
```bash
gcloud compute start-iap-tunnel siem-vm 9000 \
  --zone=ZONE \
  --local-host-port=localhost:9000
```
To access the windows VM, you can forward the rdp port using this command
```bash
gcloud compute start-iap-tunnel windows-victim 3389 \
  --zone=ZONE \
  --local-host-port=localhost:3389
```

### Conclusion
Now we are really ready to test our lab and investigate cases and alerts. In case you have any issues or concerns with the lab setup, you can send your feedback to th3cyb3rw0lf@proton.me (my email).
For now keep stopping the bad guys, H4x0r!!
### CIAO!!
