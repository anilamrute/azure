# ☁️ Azure DevOps & Cloud Concepts - Quick Reference

This repository contains simplified Azure concepts, workflows, and configurations for DevOps interviews.

---

## 1. Cloud Service Models (The "Pizza" Analogy)
*How much do you manage vs. how much does Azure manage?*

| Service Model | Concept | Simple Explanation | Example |
| :--- | :--- | :--- | :--- |
| **IaaS** (Infrastructure as a Service) | **"Rent the Kitchen"** | I rent the hardware (VM, Storage), but I must manage the OS, updates, and software. | Azure Virtual Machines (VM) |
| **PaaS** (Platform as a Service) | **"Order a Pizza"** | Azure manages the hardware *and* the OS. I just bring my code or data. | Azure SQL Database, App Service |
| **SaaS** (Software as a Service) | **"Eat at a Restaurant"** | I use the software directly. I manage nothing. | Outlook, Office 365, Gmail |

---

## 2. Core Concepts: Resources & Resource Groups

### 🔹 What is a Resource?
Any single service you create in Azure is a **Resource**.
* *Examples:* Virtual Machine, SQL Database, Public IP, Network Interface.

### 🔹 What is a Resource Group (RG)?
A logical "folder" or container that holds related resources.
* **Analogy:** If you have a file cabinet, the "Resource Group" is a specific folder for a specific project.
* **Why use it? (Interview Answer):**
    1.  **Lifecycle Management:** If I delete the Resource Group, everything inside (VM, IP, Disk) is deleted. Good for cleaning up "Dev" environments.
    2.  **Billing:** I can track exactly how much the "Payment Project" costs by checking its Resource Group.
    3.  **Organization:** Keeps "Dev" resources separate from "Prod" resources.

> **Note:** A resource can belong to only **one** Resource Group at a time.

---

## 3. Virtual Machines (VM) & Jenkins Deployment

### 🔹 Why Virtualization?
* **Old Way (Physical Servers):** One server = One App. This wastes money if the app is small.
* **New Way (Virtualization):** One Physical Server -> **Hypervisor** -> Runs multiple "Virtual" Machines. Efficient!

### 🔹 Workflow: Deploying Jenkins
1.  **Create VM:** Azure Portal -> Search "VM" -> Select Image (Ubuntu) -> Choose Size (Standard_B1s for free tier).
2.  **Login:** Use SSH Key (safer than password).
3.  **Install Jenkins:** Connect to the VM and run the installation commands (Git/Java/Jenkins).
4.  **Open the Port (Critical Step):**
    * By default, Azure blocks strange ports.
    * Go to **Networking** -> **Inbound Port Rules**.
    * Add Rule: Allow **TCP** on Port **8080**.
    * *Check:* `ps -ef | grep jenkins`

### 🔹 Autoscaling (VM Scale Sets)
* **Scenario:** We receive unlimited traffic, and the server crashes.
* **Solution:** **Virtual Machine Scale Sets (VMSS)**.
* **How it works:** If CPU Usage > 75%, Azure automatically creates a *new* VM. If traffic drops, it deletes the extra VM to save money.

---

## 4. Azure Networking (The "Secure House" Analogy)

### 🔹 VNet (Virtual Network)
* **Concept:** My private network in the cloud.
* **Why?** Security. If Hacker A attacks "VNet A", "VNet B" is safe because they are totally isolated.

### 🔹 Subnet
* **Concept:** Dividing the big VNet into smaller "rooms".
* *Example:* Web Subnet (Public), Database Subnet (Private).

### 🔹 NSG (Network Security Group)
* **Concept:** The **Security Guard** for your Subnet.
* It is a list of rules: "Allow SSH from my IP" or "Block everything else".

### 🔹 ASG (Application Security Group)
* **Concept:** Grouping VMs by their "Job" instead of IP address.
* *Benefit:* Instead of saying "Allow Port 80 on IP 10.0.0.5", I say "Allow Port 80 on all **WebServers**".

---

## 5. Advanced Networking

### 🔹 VNet Peering
* Connecting two different VNets so they can talk to each other (like building a bridge between two isolated islands).

### 🔹 VPN Gateway
* A secure tunnel connecting your **Physical Office** (On-Premise) to **Azure** over the internet.

### 🔹 Application Gateway
* A smart Load Balancer for websites (Layer 7). It creates a "single entry point" and routes traffic to the right backend pool.

---

## 6. Security Architecture: Firewall & Bastion

### 🔹 Secure Deployment Flow
`User` -> `Azure Firewall` -> `Web Subnet` -> `VM (Nginx)`

1.  **Azure Firewall:** A managed security service that filters traffic for the *entire* VNet.
    * **DNAT (Destination NAT):** A rule that says "Traffic hitting the Firewall Public IP should be forwarded to the Private VM IP".
2.  **Azure Bastion:**
    * **Problem:** We should never open Port 22 (SSH) to the public internet.
    * **Solution:** Use Bastion. It lets you SSH into the VM securely **through the Azure Portal** (browser) using SSL. No Public IP needed on the VM.

---

## 7. VM Automation: User Data vs. Custom Data

* **Custom Data:** The "Old Way". A script (bash/powershell) you upload while creating the VM. It runs *once* during provisioning. Hard to retrieve later.
* **User Data:** The "New Way". Similar to Custom Data but **persistent**. You can retrieve it anytime from the Azure Instance Metadata Service (IMDS).

---

## 8. Azure Storage Services

* **Why Azure Storage?** High Durability (3 copies of data by default), Performance, and Security (Integrated with Active Directory).

| Service | Best For |
| :--- | :--- |
| **Blob Storage** | **"Unstructured Data"**. Images, Videos, Logs, Backups. (Like S3 in AWS). |
| **File Storage** | **"Shared Folder"**. A standard network drive (SMB/NFS) that multiple VMs can mount at the same time. |
| **Table Storage** | **"NoSQL"**. Storing massive amounts of structured data (key-value) cheaply. |
| **Queue Storage** | **"Messaging"**. Storing messages to decouple parts of an application (Producer -> Queue -> Consumer). |

---

## 9. Azure CLI (Command Line Interface)

**Why use it?** Speed and Automation. Clicking in the portal is slow; scripts are fast.

### 🔹 Common Commands
```bash
# 1. Login to Azure
az login

# 2. Check Version
az version

# 3. Create a Resource Group
az group create --name "MyResourceGroup" --location "eastus"

# 4. Create a VM
az vm create --resource-group "MyResourceGroup" --name "MyVM" --image "UbuntuLTS"

# 5. Delete a Resource Group (Cleanup)
az group delete --name "MyResourceGroup"
