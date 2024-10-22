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

```

# Automated Deployment Setup Using GitHub Webhook

This guide explains how to set up automated deployment for your Laravel project using a GitHub webhook and a simple shell script. This method ensures that every time you push code to your GitHub repository, the latest changes are automatically pulled into your server.

## Step 1: Create a Webhook in Your GitHub Repository

1. Go to your GitHub repository.
2. Navigate to **Settings** > **Webhooks**.
3. Click **Add webhook**.
4. In the **Payload URL** field, enter the URL to the webhook endpoint on your Laravel application (e.g., `https://your-domain.com/webhook-handler`).
5. Set the **Content type** to `application/json`.
6. Under **Which events would you like to trigger this webhook?**, choose **Just the push event**.
7. Click **Add webhook**.

## Step 2: Create the `deploy.sh` Script on Your Server

On your server, you need a shell script that will automatically pull the latest code from the GitHub repository. Here's how you can do it:

1. SSH into your server and navigate to your `public_html` directory:
   ```bash
   cd /your/path/to/public_html
   ```
2. Create a deploy.sh script:
  ```bash
  nano deploy.sh
  ```
3. Add the following content to the deploy.sh file:
```bash
#!/bin/bash
cd /your/path/to/public_html
git pull https://<your-personal-access-token>:x-oauth-basic@github.com/your-username/your-repo.git main
```
Replace:
. /your/path/to/public_html with the correct path to your Laravel application.
. <your-personal-access-token> with your GitHub personal access token.
. your-username and your-repo with your GitHub username and repository name.

4. Save and exit the editor.
5. Make the deploy.sh script executable:
  ```bash
  chmod +x deploy.sh
  ```

## Step 3: Laravel Webhook Endpoint

In your Laravel application, create a route that will handle the webhook from GitHub. This route will execute the deploy.sh script upon receiving the push event.

1. Open your routes/web.php file (or any routes file where you want to define the webhook route).

2. Add the following route:

```bash

use Symfony\Component\Process\Process;
use Symfony\Component\Process\Exception\ProcessFailedException;

Route::post('/webhook-handler', function () {
    // Run the deploy script
    $process = new Process(['/bin/bash', '/your/path/to/public_html/deploy.sh']);
    
    try {
        $process->mustRun(); // This will throw an exception if the command fails
    } catch (ProcessFailedException $exception) {
        return response('Deployment failed: ' . $exception->getMessage(), 500);
    }

    return response('Deployment completed successfully.', 200);
});
```

3. Replace /your/path/to/public_html/deploy.sh with the actual path to the deploy.sh script on your server.

## Step 4: Test the Webhook

1. Make a change to your code and push it to the main branch of your GitHub repository.
2. GitHub will trigger the webhook, which in turn runs the deploy.sh script on your server.
3. Verify that the latest changes have been pulled to your server by checking the public_html directory or visiting your application.

## Conclusion

With this setup, every push to your GitHub repository will automatically pull the latest code to your server, ensuring continuous deployment. This simple automation saves time and reduces the chances of deployment issues.

For security purposes, make sure to protect your GitHub access token and consider restricting it to necessary permissions only.

If you encounter any issues or need further assistance, feel free to consult the official documentation or reach out to the community for support.
