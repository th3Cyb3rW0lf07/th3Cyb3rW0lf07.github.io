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
  --image-project=kalilinux-images \
  --image-family=kali-linux \
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
After the four VMs are created you should have all instances available in the Compute Engine page

<img width="700" height="162" alt="Screenshot From 2025-09-08 12-10-24" src="https://github.com/user-attachments/assets/e63be246-f2b9-4359-91b9-f7416e7d5244" />

That's the progress so far...Updates coming soon, stay tuned!!
