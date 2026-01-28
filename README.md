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


**10. Azure Resource Manager (ARM) Fundamentals**
* **Definition:** ARM is the central management layer for Azure. It acts as the "traffic controller" that receives requests from users and forwards them to the Azure Resource Providers.
* **Interaction Methods:**
    * CLI (Command Line Interface)
    * Azure UI (Portal)
    * SDKs
    * **IaC Tools:** ARM Templates, Bicep
* **Standardization:** All requests (whether from the Portal or a script) pass through the **Azure Resource Manager API**, ensuring consistent security and policy enforcement.

**11. Infrastructure as Code: ARM Templates**
* **Concept:** A native service to automate resource creation using declarative JSON files.
* **Flow:** `User` -> `ARM Template (JSON)` -> `ARM API` -> `Azure Resources`
* **Benefit:** Enables consistent, repeatable infrastructure deployment.

**12. IAM - Azure Identity & Access Management**
* **Authentication (AuthN):** *Who are you?* (Managed via Users & Groups).
* **Authorization (AuthZ):** *What can you do?* (Managed via Roles & Policies).
* **Microsoft Entra ID:** The centralized dashboard to manage users, roles, and authorizations.
* **RBAC (Role-Based Access Control) Workflow:**
    * Create User.
    * **Add Assignment:** Assign a specific role (e.g., Contributor) to that user.
    * **Service Principals:** Special identities created for automation tools (like Terraform/Jenkins) to manage resources securely.

**13. Azure DevOps (ADO) Platform**
* **What is it?** A Microsoft platform providing a suite of services to optimize the SDLC (Software Development Life Cycle).
* **Why use it?** Reduces time-to-market, improves productivity, and centralizes automation.
* **Key Services:**
    * **Azure Boards:** Planning phase (User stories, Tasks).
    * **Azure Repos:** Version control for code, Terraform scripts, and Ansible playbooks.
    * **Azure Pipelines:** CI/CD engine (Build -> Test -> Deploy).
    * **Azure Test Plans:** Management for manual and automated testing.
    * **Azure Artifacts:** Storage for build outcomes (JAR, WAR, Docker images).

**14. Project: CI Pipeline Setup**
* **Reference:** [Azure Zero to Hero](https://github.com/iam-veeramalla/Azure-zero-to-hero)
* **CI Workflow:**
    `GitHub Push` -> `Unit Test` -> `Static Analysis` -> `Build` -> `Docker Image` -> `Push to ACR`
* **Setup Steps:**
    1.  **Repo:** Import code from GitHub and set `main` branch as default.
    2.  **Container Registry:** Create an **Azure Container Registry (ACR)** via the portal.
* **Pipeline Configuration (YAML):**
    * **Trigger:** Set path filters (run only when code changes in specific paths).
    * **Stages:** Build Stage, Image Push Stage, End-to-End Test.
    * **Jobs/Steps:** Define tasks like `docker build`.
    * **Pool:** Name set to `Azure Pipelines` (Hosted Agent).

**15. Azure CD: Continuous Delivery (GitOps)**
* **Concept:** Modern delivery approach where the Git repository is the "Single Source of Truth."
* **Flow:** `Git Repo` -> `ArgoCD` -> `Kubernetes Cluster`
* **Objective:** After the CI pipeline pushes the image, the CD process automatically deploys the new changes to the runtime environment.
* **Steps:**
    1.  Create Kubernetes Cluster.
    2.  Login to K8s.
    3.  Install and Configure **ArgoCD**.

**16. Azure Kubernetes Service (AKS)**
* **Deployment Models:**
    1.  **On-Premises:** Physical VMs (Self-managed data plane & worker nodes). High maintenance for upgrades.
    2.  **Azure VMs (IaaS):** K8s on Cloud VMs. User still manages OS patching.
    3.  **AKS (Managed Service):** Azure manages the Control Plane. User manages Node Pools.
* **AKS Benefits:**
    * **Auto-Scaling:** Automatically adds nodes during traffic spikes.
    * **Automatic Upgrades:** Azure handles patching.
    * **Integrations:** Native support for Load Balancers (LB), Ingress, and Secrets.
    * **Cost:** Pay only for the worker nodes (VMs).


