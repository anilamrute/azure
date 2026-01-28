# Azure Hands‑On Tutorial — VM + Jenkins, Storage Account, ACR + Pipeline

This tutorial walks you through:
- Deploying an Ubuntu VM and installing Jenkins (cloud-init) with Bicep and Azure CLI.
- Creating a Storage Account (Bicep + CLI).
- Creating an Azure Container Registry (ACR) and a CI workflow to build/push a Docker image (Bicep + GitHub Actions).

Prereqs
- Azure account & az CLI installed and logged in: `az login`
- Azure CLI extensions: `az extension add --name resource-graph` (optional)
- GitHub repository (for CI workflow)
- Basic familiarity with Bicep or willingness to use CLI scripts below

How to use this repository
1. Choose either the Bicep templates (recommended) or the CLI scripts in the `scripts/` directory.
2. Follow steps in the relevant section, capture screenshots as instructed, and then continue to verification.

Sections:
1) VM + Jenkins (quick start)
2) Storage Account
3) ACR + Pipeline (GitHub Actions)
4) Cleanup

---

1) VM + Jenkins (quick start)

Goal: Create a small Ubuntu VM, install Jenkins using cloud-init, open port 22 (SSH) and 8080 (Jenkins UI), and verify Jenkins is accessible.

Recommended resource names (change as needed)
- Resource group: rg-jenkins-demo
- Location: eastus
- VM name: jenkins-vm
- Admin user: azureuser

Use the Bicep template `vm-jenkins.bicep` or the script `deploy-vm-jenkins.sh`.

Bicep deploy (example)
- Create resource group:
  - `az group create -n rg-jenkins-demo -l eastus`
- Deploy:
  - `az deployment group create -g rg-jenkins-demo --template-file vm-jenkins.bicep --parameters adminUsername=azureuser sshKey=@~/.ssh/id_rsa.pub`

What the Bicep does:
- Creates vnet + subnet
- Creates a Network Security Group allowing SSH and Jenkins (8080)
- Creates a public IP and NIC
- Deploys an Ubuntu VM with cloud-init `customData` that installs Java & Jenkins and starts Jenkins

Screenshot guidance
- After `az deployment` completes, capture:
  - Screenshot 1: Azure Portal → Resource Group page showing resources (file: screenshots/rg-jenkins-resources.png)
  - Screenshot 2: Public IP page showing assigned IP (file: screenshots/jenkins-public-ip.png)
  - Screenshot 3: Jenkins UI in browser: `http://<public-ip>:8080` (file: screenshots/jenkins-ui.png)

Verify manually
- SSH: `ssh azureuser@<public-ip>`
- Check jenkins process: `sudo systemctl status jenkins`
- Retrieve initial admin password:
  - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

If using CLI script
- `chmod +x deploy-vm-jenkins.sh`
- `./deploy-vm-jenkins.sh rg-jenkins-demo eastus jenkins-vm azureuser ~/.ssh/id_rsa.pub`

---

2) Storage Account

Goal: Create a general-purpose v2 Storage Account and demonstrate uploading a blob from CLI.

Bicep deploy
- `az group create -n rg-storage-demo -l eastus`
- `az deployment group create -g rg-storage-demo --template-file storage-account.bicep --parameters storageAccountName=mystorageacctdemo`

CLI commands to test
- Get keys:
  - `az storage account keys list -g rg-storage-demo -n mystorageacctdemo`
- Create a container:
  - `az storage container create --account-name mystorageacctdemo -n demo-container`
- Upload a file:
  - `az storage blob upload --account-name mystorageacctdemo -c demo-container -f README.md -n README.md`

Screenshot guidance
- Screenshot: Storage Account Overview (file: screenshots/storage-account-overview.png)
- Screenshot: Uploaded blob container contents (file: screenshots/storage-container-blobs.png)

---

3) ACR + Pipeline (GitHub Actions example)

Goal: Create ACR, build and push a Docker image automatically from GitHub Actions.

Bicep to create ACR
- `az group create -n rg-acr-demo -l eastus`
- `az deployment group create -g rg-acr-demo --template-file acr.bicep --parameters registryName=myacrdemo12345`

Enable admin user for quick tests (not recommended for production)
- `az acr update -n myacrdemo12345 --admin-enabled true`
- Get credentials:
  - `az acr credential show -n myacrdemo12345`

GitHub Actions workflow
- Add `.github/workflows/acr-build.yml` to your repo (see provided file)
- Secrets needed in GitHub repo:
  - `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID` (if using azure/login)
  - Or use `AZURE_ACR_LOGIN_USERNAME` & `AZURE_ACR_LOGIN_PASSWORD` if using docker/login

Example quick build without GitHub Actions:
- `az acr build -t myacrdemo12345.azurecr.io/myapp:latest -r myacrdemo12345 .`

Screenshot guidance
- Screenshot: ACR Overview in Portal (file: screenshots/acr-overview.png)
- Screenshot: GitHub Actions run success with build logs (file: screenshots/github-actions-build.png)
- Screenshot: ACR repository showing pushed image tag (file: screenshots/acr-repo-image.png)

---

4) Cleanup

Resource group deletion (removes everything in the group):
- `az group delete -n rg-jenkins-demo -y --no-wait`
- Repeat for each demo RG (rg-storage-demo, rg-acr-demo)

---

Appendix: Useful commands
- `az login`
- `az account show`
- `az group create -n <rg> -l <region>`
- `az deployment group create -g <rg> --template-file <file>.bicep --parameters "param=value"`

References
- Azure CLI: https://learn.microsoft.com/cli/azure
- Bicep docs: https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview
- Jenkins docs: https://www.jenkins.io/doc/
