# Azure-Webpage-Deployment
### CI/CD pipeline for deploying a custom web page to an Azure VM.

This project demonstrates my hands-on skills in cloud computing, web server management, DNS configuration, and DevOps automation. I deployed a custom webpage on an Azure VM and set up a CI/CD pipeline with GitHub Actions to automatically update the webpage upon code changes. This documentation details the entire process—from VM creation to troubleshooting CI/CD errors—showcasing my practical abilities and problem-solving skills.
### Table of Contents

 **Project Overview**
   - Step 1: Azure VM Setup
   - Step 2: Connecting to the VM & Installing Apache2
   - Step 3: Hosting a Custom Web Page
   - Step 4: DNS Configuration
   - Step 5: CI/CD Pipeline Setup
   - Troubleshooting & Resolutions
# Project Overview

**Objective:** Deploy a custom HTML webpage on an Azure VM and automatically update it via a CI/CD pipeline using GitHub Actions.
**Skills Demonstrated:**
   -  Cloud VM provisioning and configuration (Azure)
   -  Web server management (Apache2)
   -  Domain and DNS configuration
   -  DevOps automation with Git and GitHub Actions
   -  Troubleshooting and resolving deployment errors

## Step 1: Azure VM Setup

1.    **VM Creation & Inbound Port Rules:**
      -  Created an Azure VM with inbound port rules for SSH (22) and HTTP (80).
      -  This ensured remote access (SSH) and public web access (HTTP).

2.    **Networking Skills:**
      - Configured the Azure Network Security Group (NSG) to allow necessary traffic.

## Step 2: Connecting to the VM & Installing Apache2

1.    SSH Connection Using Remmina:
      - Connected to the VM using Remmina via SSH.
      - This step demonstrated remote server management skills.

2.    Installing Apache2:
- Installed Apache2 with:
   -       sudo apt update && sudo apt install apache2 -y
- Verified that the default Apache page was accessible by visiting the VM's public IP.
## Step 3: Hosting a Custom Web Page

1.    **Replacing the Default Page:**
        **Important Note:**
      Always replace the default index.html file with your custom version; do not create a new file alongside the default one.
      - Issue: Creating a new file without replacing the default can lead to confusion and wasted time troubleshooting.
      - Replaced the existing /var/www/html/index.html with my custom HTML page.
      - Confirmed that the webpage was updated by visiting the domain/public IP.
## Step 4: DNS Configuration

1.    **Domain Pointing:**
        - Configured a DNS record to point a custom domain to the Azure VM, instead of relying solely on the public IP.
        - This added professionalism and showcased knowledge of domain management.
## Step 5: CI/CD Pipeline Setup

1.    **Repository Creation & Local Setup:**
      -   Created a dedicated Git repository named azure-webpage-deployment.
        - Cloned it locally using VS Code:

-     git clone https://github.com/<your-username>/azure-webpage-deployment.git

-    Pushed the initial webpage code to the remote repository.
2. **Creating the GitHub Actions Workflow:**

   - Inside the repository, created the directory .github/workflows and a file deploy.yml that defines the deployment steps.
-    Key Steps in deploy.yml:
       - Checkout Code:
        Uses actions/checkout to pull the latest code.
      -  Set Up SSH:
        Uses webfactory/ssh-agent to load the SSH private key from GitHub Secrets:

`ssh-private-key: ${{ secrets.AZURE__SSH__PRIVATE__KEY }}`

**Note:** *The secret name must exactly match the one configured in GitHub (e.g., *AZURE__SSH__PRIVATE__KEY*).*
- File Deployment & Apache Restart:
Uses *scp* to copy files to /var/www/html/ and *ssh* to restart Apache on the Azure VM.
- Crucial Reminder:
Do not forget to *git add* and commit the workflow file.
3. **Configuring GitHub Secrets:**

-   **Generated an SSH key pair:**

      -     ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

- Added the public key (id_rsa.pub) to the Azure VM's ~/.ssh/authorized_keys.
- Stored the private key and the Azure VM IP in GitHub Secrets:

   - AZURE__SSH__PRIVATE__KEY: Contains the content of your private key.
   - AZURE_VM_IP: Contains your Azure VM’s public IP address.

# Troubleshooting & Resolutions
## Error 1: Empty SSH Private Key Argument

   ### Error Message:
-    "Error: The ssh-private-key argument is empty. Maybe the secret has not been configured, or you are using a wrong secret name in your workflow file."
     -   Cause & Solution:
          -   Cause: Incorrect or mismatched secret name (e.g., using underscores incorrectly).
      -  Solution:
           - Ensure the secret name in GitHub (e.g., AZURE__SSH__PRIVATE__KEY) exactly matches the reference in your workflow.
           - Copy the complete private key correctly.

## Error 2: Host Key Verification Failed

###    Error Message:
` "Host key verification failed. scp: Connection closed"`
  -  Cause & Solution:
       - Cause: The workflow was prompting for host key verification, causing a hang.
       - Solution:
           - Add -o StrictHostKeyChecking=no to the scp and ssh commands:

               -     scp -o StrictHostKeyChecking=no -r * username@${{ secrets.AZURE_VM_IP }}:/var/www/html/
               -     ssh -o StrictHostKeyChecking=no username@${{ secrets.AZURE_VM_IP }} "sudo systemctl restart apache2"
        
## Error 3: Permission Denied on File Upload

 ###   Error Message:
` "scp: dest open '/var/www/html/index.html': Permission denied"`
-   Cause & Solution:
       - Cause: The default user on the Azure VM did not have write permissions to /var/www/html/.
       - Solution:
          -  SSH into the Azure VM and change the directory's ownership:

-     sudo chown -R linux:linux /var/www/html/

- After adjusting permissions, re-run the workflow.
