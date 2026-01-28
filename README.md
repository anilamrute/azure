# Azure — Simple Practical Guide

A concise, human-readable summary of common Azure concepts and simple how‑tos. Use this as a quick reference for learning IaaS/PaaS/SaaS, resources and resource groups, networking, storage, CI/CD, and AKS.

---

## Table of contents
- Overview: IaaS / PaaS / SaaS
- Resources & Resource Groups
- Deploy Jenkins on an Azure VM (high-level)
- Azure Networking (basic & advanced)
- Deploying behind a firewall & Bastion
- VM automation: user data / custom data
- Azure Storage services
- Azure CLI (quick notes)
- Azure Resource Manager (ARM) & templates
- IAM (Microsoft Entra ID / roles)
- Azure DevOps: services & CI/CD flow
- Azure CD & GitOps (ArgoCD / AKS)
- AKS (Kubernetes on Azure) — quick overview
- Useful commands & links

---

## Overview: IaaS / PaaS / SaaS
- IaaS (Infrastructure as a Service): You manage VMs, storage, OS. Example: VM + SQL Server installed by you.
- PaaS (Platform as a Service): Platform managed for you, e.g., Azure SQL Database.
- SaaS (Software as a Service): Full software delivered (no infra config). Example: Outlook, Office 365.

---

## Resources & Resource Groups
- Resource: an instance of a service (VM, SQL DB, Storage Account).
- Resource group: logical grouping of related resources (e.g., `projectname_env`).
  - Why: easier organization, permissions, billing, lifecycle management.
  - Best practice: group by project + environment (e.g., `payment_dev`, `payment_prod`).
- Mapping: many resources → one resource group (1..N relationship).

---

## Deploy Jenkins on an Azure VM (high-level)
1. Create a VM from Portal (Subscription → Resource Group → Create VM).
   - Choose region near you, pick image (e.g., Ubuntu), choose size (free tier if available).
2. Open inbound port 8080 (Network Security Group) for Jenkins UI.
3. SSH into VM and install Jenkins (or use the Jenkins repo/installation docs).
   - Example check: `ps -ef | grep jenkins`
4. Configure autoscaling (if using multiple agents) so capacity grows on demand.

Notes:
- Consider using an Azure VM Scale Set or containerized Jenkins agents for scale.

---

## Azure Networking (basic)
- VNet (Virtual Network): isolates network(s) — create VNets for grouping resources.
- CIDR: choose address space to determine number of IPs.
- Subnets: logical segmentation within a VNet.
- NSG (Network Security Group): security rules for subnet or NIC (allow/deny traffic).
- ASG (Application Security Group): group VMs for security rules.
- Route table: control network traffic flow.

Potential risk: VMs in same VNet can reach each other — follow least-privilege networking.

Advanced components:
- Application Gateway (Layer 7 load balancer / WAF)
- VNet peering: connect VNets within/ across subscriptions/regions
- VPN Gateway: connect on-prem to Azure VNet

Example topology:
VNet → Application Gateway → Route Table → Subnet → VM

---

## Deploying behind Azure Firewall & Bastion
- Typical pattern:
  - VNet → Azure Firewall → Subnet (web) → VM (nginx)
  - Use Azure Bastion for secure management access to VMs (no public IP on VM required).
- Firewall DNAT rules:
  - Map public IP:port -> private VM IP:port. Set source to your IP for security.
- After resource creation:
  - Connect via Bastion to the private VM
  - Install/configure web server (nginx)
  - Add DNAT rule in Firewall policy pointing to the VM's private IP

---

## VM automation: user data vs custom data
- On Linux, use cloud-init (passed via `customData`) to run provisioning scripts on first boot.
- If you need scripts to run after every reboot, use VM extensions, systemd services, or startup scripts configured inside the OS.
- For repeatable infrastructure provisioning, prefer VM extensions or configuration management tools (Ansible, Chef, etc.)

---

## Azure Storage Services
Why use Azure Storage? Durability, performance, security (integration with Azure AD).

- Blob Storage: unstructured objects (images, video, large files).
- File Storage: managed SMB file share for VMs and containers.
- Table Storage: NoSQL key-value store.
- Queue Storage: simple message queue for background processing.

All are managed through a Storage Account.

Create a storage account via the Azure Portal or CLI:
- Example CLI: `az storage account create --name mystorageacct --resource-group myrg --location eastus --sku Standard_LRS`

---

## Azure CLI (quick notes)
- Use CLI to automate, reduce manual errors, and script tasks.
- Install docs: https://learn.microsoft.com/cli/azure/install-azure-cli
- Common commands:
  - `az --version`
  - `az login`
  - `az group create --name myrg --location eastus`
  - `az group delete --name myrg`

---

## Azure Resource Manager (ARM) & templates
- Azure Resource Manager (ARM) is the control plane for creating/managing Azure resources.
- Ways to create resources:
  - Portal UI, Azure CLI, ARM templates (JSON), Bicep (recommended authoring), SDKs, Terraform
- ARM Template flow: User → ARM Template → ARM → Azure
- Bicep is a more concise, declarative alternative to raw JSON ARM templates.

---

## IAM (Identity & Access Management)
- Microsoft Entra ID (Azure AD) manages users, groups, and identity.
- Authentication: who you are (user, service principal, managed identity).
- Authorization: what you can do (role assignments, RBAC).
- Create users, groups, and role assignments in Entra ID.
- Use service principals or managed identities for non-human identities and automated access.

---

## Azure DevOps: services & CI/CD flow
What is Azure DevOps?
- A suite of services to support the entire DevOps lifecycle: Boards, Repos, Pipelines, Test Plans, Artifacts.

Key components:
- Boards: planning and tracking.
- Repos: git source control (store code, Terraform, Ansible).
- Pipelines: CI/CD (build, test, deploy).
- Test Plans: manual and automated test management.
- Artifacts: package registry (NuGet, npm, Docker images as artifacts).

Example CI flow:
1. Developer pushes code to GitHub.
2. Pipeline triggers: run unit tests, static analysis, build.
3. Build creates Docker image and pushes to Azure Container Registry (ACR).
4. CD deploys image to target (VMs, AKS, or other targets).

Setting up a repo and pipeline:
- Import GitHub repo into Azure Repos or connect GitHub directly.
- Create Dockerfile and build pipeline with stages:
  - build → test → image push → deployment
- Use pipeline templates for Docker builds.
- Configure triggers: path filters or branch policies.
- Use agent pool: `azure-pipelines-agent` (or Microsoft-hosted agents).

Useful link (example repo):
- https://github.com/iam-veeramalla/Azure-zero-to-hero

---

## Azure CD & GitOps
- Continuous Delivery: deploy changes automatically after successful CI.
- GitOps: Git is the single source of truth for desired state. Tools like Argo CD apply declared state to clusters.
- Typical GitOps flow:
  - Git (app manifests) → ArgoCD (or Flux) → Kubernetes cluster (AKS)

Install ArgoCD on AKS to manage app deployments by Git repo.

---

## AKS (Azure Kubernetes Service) — overview
Kubernetes deployment options:
1. On-prem self-managed cluster (you manage control plane & nodes).
2. Kubernetes on Azure VMs (self-managed on Azure VMs).
3. AKS (Azure managed Kubernetes) — control plane managed by Azure; you manage node pools.

Why AKS?
- Managed control plane, easier upgrades, integration with Azure features (LB, ingress, ACR, Azure AD).
- Node pools (multiple VM types), autoscaling, automatic upgrades (optional).
- Considerations: cost, maintenance windows, upgrade strategies.

---

## Useful commands & snippets

Azure CLI
- Login: `az login`
- Create resource group: `az group create --name myrg --location eastus`
- Create storage account: `az storage account create --name mystorageacct --resource-group myrg --location eastus --sku Standard_LRS`
- Delete resource group: `az group delete --name myrg`

Check Jenkins/process on VM
- `ps -ef | grep jenkins`

ACR & Docker (example)
- Create ACR: `az acr create --name myACR --resource-group myrg --sku Basic`
- Login to ACR: `az acr login --name myACR`
- Build & push:
  - `docker build -t myACR.azurecr.io/myapp:tag .`
  - `docker push myACR.azurecr.io/myapp:tag`

AKS basics
- Create AKS: `az aks create --resource-group myrg --name myAKS --node-count 3 --enable-managed-identity`
- Get credentials: `az aks get-credentials --resource-group myrg --name myAKS`
- Upgrade nodes, scale, and use node pools via `az aks` subcommands.

---

## Best practices (high level)
- Use resource groups per project + environment.
- Use RBAC and least-privilege roles.
- Use NSGs and Azure Firewall / WAF for network protection.
- Keep sensitive data in Key Vault, not source code.
- Use automation (CLI, ARM/Bicep, Terraform) for repeatability.
- Use CI for builds/tests, and GitOps/ArgoCD for continuous deployments.

---

## References
- Azure CLI install docs: https://learn.microsoft.com/cli/azure/install-azure-cli
- Azure DevOps docs: https://learn.microsoft.com/azure/devops
- AKS docs: https://learn.microsoft.com/azure/aks
- GitHub repo example: https://github.com/iam-veeramalla/Azure-zero-to-hero

---

If you want, I can:
- Convert this into a longer tutorial with commands and step-by-step screenshots.
- Create example ARM/Bicep templates or Azure CLI scripts for common tasks (VM + Jenkins, Storage account, ACR + Pipeline).
