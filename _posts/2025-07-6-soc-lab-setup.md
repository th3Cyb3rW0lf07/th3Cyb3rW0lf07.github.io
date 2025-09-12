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
- SIEM testing (Splunk)

**Components**:
- Attacker VM (Kali Linux)
- SIEM VM (Splunk Enterprise)
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
Name: splunk-vm
OS: Ubuntu 22.04
Software: Splunk Enterprise
Log Input: Syslog, HEC
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

Finally, we add the rule to allow access to Splunk UI when we install it.
```bash
gcloud compute firewall-rules create allow-splunk-ui \
  --network=soc-vpc \
  --allow=tcp:8000 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=splunk-access
```
You should have something like this

<img width="700" height="123" alt="Screenshot From 2025-09-08 12-23-04" src="https://github.com/user-attachments/assets/2830a447-1e22-42ec-a516-528b629f63a4" />

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
gcloud compute instances create splunk-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --subnet=soc-subnet \
  --tags=ssh-access,splunk-access \
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

Next, let's setup the attack framework on the Attacker VM. The choice was between Caldera and Atomic red team, but we'll use Caldera. You can reference the [Caldera Documentation](https://caldera.readthedocs.io/en/latest/index.html).

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

Next we boot up the splunk-vm running Ubuntu 22.04. Since we're using Splunk for this setup, you'll have to sign up on the website to download any of he packages that'll be used.

First download needed is Splunk Enterprise. At the time of this setup, the latest stable version of splunk enterprise. You can reach the download page [here](https://www.splunk.com/en_us/download/splunk-enterprise.html). Copy the command to download the .deb package for Linux. It should look like this...
```bash
wget -O splunk-10.0.0-xxxx-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.0.0/linux/splunk-10.0.0-xxx-linux-amd64.deb"
```

Then install the package
```bash
sudo dpkg -i <package-name>
```

Once that's done, you can start splunk.
```bash
sudo /opt/splunk/bin/splunk start --accept-license
```
At this point, you'll be asked to set your username and password. This set of credentials would give you access to your Splunk web UI.

You can also enable splunk to start at boot with this cmd:
```bash
sudo /opt/splunk/bin/splunk enable boot-start
```
Since we already have the splunk UI access configured in the firewall, we can access it from anywhere (for now) at 
```yaml
http://splunk-vm-ip:8000
```
<img width="800" height="791" alt="Screenshot From 2025-09-12 14-42-57" src="https://github.com/user-attachments/assets/feff9e53-4691-4aab-bb8d-177848cb95c9" />

We also need to add another firewall rule to allow use to receive data from our victim VMs.
```bash
gcloud compute firewall-rules create allow-splunk-ingest \
  --network=soc-vpc \
  --allow=tcp:9997 \
  --source-ranges=ubuntu-victim-ip,windows-victim-ip \
  --target-tags=splunk-indexer

gcloud compute instances add-tags splunk-vm \
  --zone=us-central1-a \
  --tags=splunk-indexer
```
Now, in the Splunk UI, we need to configure receiving data for the SIEM.
Go to Settings>Forwarding and Receiving>Configure receiving and set the port to 9997
<img width="403" height="229" alt="Screenshot From 2025-09-12 19-43-06" src="https://github.com/user-attachments/assets/9d7aae4c-0294-4094-8f5e-eaff2cbab899" />
<img width="926" height="221" alt="Screenshot From 2025-09-12 19-43-35" src="https://github.com/user-attachments/assets/3f367286-4df6-4d6c-973d-a00e8dec1171" />

### Victim Setup

Now we'll use Splunk Forwarder to send logs to the SIEM. You can get the download from [here](https://www.splunk.com/en_us/download/universal-forwarder.html)

#### Ubuntu
Your download command should look like this for the 64-bit Linux .deb package
```bash
wget -O splunkforwarder-10.0.0-xxx-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/10.0.0/linux/splunkforwarder-10.0.0-xxx-linux-amd64.deb"

sudo dpkg -i <package-name>
```
Start the forwarder
```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```
Add the indexer
```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server <splunk-vm-ip>:9997
```
Add log sources
```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

#### Windows
Grab the download link for windows [here](https://www.splunk.com/en_us/download/universal-forwarder.html)

Install It:
- During setup, specify the Splunk VM’s IP and port 9997.
- Choose which logs to forward: System, Security, and Application.

Verify the service is running
```bash
Get-Service splunkforwarder
```
<img width="445" height="102" alt="Screenshot From 2025-09-12 15-43-01" src="https://github.com/user-attachments/assets/ace36eb6-aa6b-4cfd-9fd1-69eef8a1223d" />

If all was done right, when we return to the Search and Reporting app on Splunk and run this query...
```yaml
index=* | stats count by host, sourcetype
```
We should get the hostnames of the victim VMs and their sources
<img width="900" height="794" alt="Screenshot From 2025-09-12 15-42-27" src="https://github.com/user-attachments/assets/e43aa438-3027-4174-b794-f8aad1a06cd4" />

Tada!!! We have data coming into the SIEM.

Next, we finish the attack VM setup and launch some attacks!!

That's the progress so far...Updates coming soon, stay tuned!!
