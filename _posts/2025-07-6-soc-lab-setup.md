---
layout: post
title: Cloud-based simple SOC Lab (Google Cloud Platform)
subtitle: SOC Lab
categories: SOC
tags: [SOC, lab setup]
---

# 🛡️ SOC Lab on Google Cloud

Hola!! It's been a while!!
Welcome to my Security Operations Center (SOC) lab documentation. This guide walks through the setup of a cloud-based SOC lab using Google Cloud Platform (GCP), featuring attacker simulation, log collection, and SIEM analysis.

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
- Victim VM (Ubuntu)
- Custom VPC and firewall rules

---

## 🧱 Architecture Diagram

<img width="2804" height="1524" alt="image" src="https://github.com/user-attachments/assets/765a6987-5514-41ea-b28f-9bdf04b6e479" />

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

### Firewall Rules

First rule in place allows SSH conections to the VMs.
```bash
gcloud compute firewall-rules create allow-ssh \
  --network=soc-vpc \
  --allow=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=ssh-access
```
<img width="1318" height="238" alt="Screenshot From 2025-09-06 19-15-05" src="https://github.com/user-attachments/assets/4fca3e4b-56a9-4adb-9d8d-2c5f83252add" />

<img width="1516" height="60" alt="Screenshot From 2025-09-06 21-26-06" src="https://github.com/user-attachments/assets/5821f97b-b609-4f9d-969e-76458c7dd305" />

There will be a second when splunk gets installed to allow http access to the Splunk UI.

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

### Victim VM
```yaml
Name: ubuntu-victim
OS: Ubuntu 22.04
Agents: Filebeat, Wazuh agent
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
<img width="1268" height="454" alt="Screenshot From 2025-09-06 19-14-53" src="https://github.com/user-attachments/assets/b1c5ac6a-8b5b-45ef-aff1-c8e30dd320b1" />


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
  --tags=ssh-access \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=100GB
```

### Create Victim VM
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
After the three VMs are created you should have all instances available in the Compute Engine page

<img width="1074" height="228" alt="Screenshot From 2025-09-06 21-21-38" src="https://github.com/user-attachments/assets/fc7976ce-f59a-4ac5-98c3-19dcfb189b28" />

That's the progress so far...Updates coming soon, stay tuned!!
