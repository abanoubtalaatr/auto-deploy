# Automated Deployment Setup Using SSH Keys with GitHub Actions CI/CD

This guide outlines the steps to set up an automatic deployment process using SSH keys. The private key will be stored in GitHub Secrets, while the public key will be added to your server. Follow these steps to enable auto-deployment when pushing to the repository.

## Steps to Setup Auto Deployment

### Step 1: Generate SSH Keys

First, you need to generate a pair of SSH keys (private and public).

Run the following command to create an RSA key pair:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

```

The private key will be saved on your local machine (by default in ~/.ssh/id_rsa).
The public key will be saved in ~/.ssh/id_rsa.pub.

### Step 2: Add the Public Key to Your Server

Now, you need to add the public key to your server so that GitHub Actions can access it.

#### Run the following command to display the contents of your public key:

```bash
cat ~/.ssh/id_rsa.pub

```

#### Copy the entire output of this command.

#### Log in to your server and open the authorized_keys file:
```bash
nano ~/.ssh/authorized_keys
```

#### Paste the copied public key into the file and save it.

### Step 3: Add the Private Key to GitHub Secrets

Next, add your private key to your GitHub repository's secrets for secure access.
 - Open your GitHub repository in a web browser.
 - Navigate to the Settings of your repository.
 - In the left sidebar, click on Secrets and variables > Actions.
 - Click on the New repository secret button.
 - Name your secret (e.g., SSH_PRIVATE_KEY).
 - Run the following command on your local machine to display your private key:
```bash
    cat ~/.ssh/id_rsa
```
 - Copy the entire output and paste it into the value field in GitHub.
 - Click Add secret to save it.

### Step 4: Configure GitHub Actions Workflow

In your repository, create or update a .github/workflows/deploy.yml file to handle the auto-deployment. Here's a basic example:

```bash
name: Deploy on Push

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up SSH
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Test SSH Connection
      run: |
        ssh -o StrictHostKeyChecking=no username@ip-of-your-server 'echo "SSH connection successful!"'
        
    - name: Deploy to Server
      run: |
        ssh -o StrictHostKeyChecking=no username@ip-of-your-server 'cd /path/to/public_html && git pull origin main && echo "Deployment completed successfully." || echo "Deployment failed."'




