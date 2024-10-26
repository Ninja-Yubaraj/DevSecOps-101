Stage 3 involves **automating deployment** for your static or dynamic website on a Linux server. This will save you time when updating your site by avoiding manual steps like logging into the server, copying files, and restarting services. The following steps will guide you in creating an automated deployment process using **Git**, **bash scripts**, or **Git hooks**.

### **Stage 3: Automating Deployment in Detail**

---

### **Option 1: Automating Deployment Using a Bash Script**

You can write a simple bash script to automate the process of pulling the latest changes from your Git repository and updating the web server. Here’s how to do it:

#### **Step 1: Set Up a Git Repository**
1. **On your server**, navigate to your website directory (e.g., `/var/www/html/mywebsite`):

    ```bash
    cd /var/www/html/mywebsite
    ```

2. **Initialize a Git repository** (if not already done):

    ```bash
    git init
    git remote add origin https://github.com/username/mywebsite.git
    ```

3. Pull the latest changes from your Git repository:

    ```bash
    git pull origin main
    ```

#### **Step 2: Write a Bash Script for Deployment**
You can automate the pull and restart process with a bash script.

1. **Create a new deployment script**:

    ```bash
    sudo nano /var/www/html/mywebsite/deploy.sh
    ```

2. Add the following content to the script:

    ```bash
    #!/bin/bash

    # Navigate to your website directory
    cd /var/www/html/mywebsite

    # Pull the latest changes from GitHub
    echo "Pulling latest changes from GitHub..."
    git pull origin main

    # Restart the Nginx server (optional for static sites)
    echo "Restarting Nginx..."
    sudo systemctl restart nginx

    # Restart your application if dynamic (e.g., Node.js or Django)
    if [ -f /var/www/html/mywebsite/app.js ]; then
        echo "Restarting Node.js app with PM2..."
        pm2 restart mywebsite
    fi

    if [ -f /var/www/html/mywebsite/venv/bin/gunicorn ]; then
        echo "Restarting Gunicorn for Django..."
        sudo systemctl restart mywebsite
    fi

    echo "Deployment completed successfully!"
    ```

3. **Make the script executable**:

    ```bash
    sudo chmod +x /var/www/html/mywebsite/deploy.sh
    ```

#### **Step 3: Set Up Automatic Deployment**
To automate the deployment process, you can schedule the script to run periodically using **cron** or trigger it manually when required.

1. **Edit the crontab** to run the script at regular intervals (e.g., every 10 minutes):

    ```bash
    crontab -e
    ```

2. Add the following cron job to run the deployment script:

    ```bash
    */10 * * * * /var/www/html/mywebsite/deploy.sh >> /var/www/html/mywebsite/deploy.log 2>&1
    ```

This cron job will run the script every 10 minutes, pull the latest code from GitHub, and restart Nginx or your application automatically.

---

### **Option 2: Automating Deployment Using Git Hooks**

If you want the deployment to happen automatically whenever you push code to your GitHub or GitLab repository, you can use **Git hooks**.

#### **Step 1: Set Up a Post-Receive Git Hook**

Git hooks allow you to automate tasks whenever certain Git actions occur. For deployment, you can use the **post-receive hook**.

1. **Set up a bare Git repository** on your server (this will act as a remote):

    ```bash
    sudo mkdir -p /var/repo/mywebsite.git
    cd /var/repo/mywebsite.git
    git init --bare
    ```

2. **Create a `post-receive` hook** inside the Git repository:

    ```bash
    sudo nano /var/repo/mywebsite.git/hooks/post-receive
    ```

3. Add the following content to the post-receive hook:

    ```bash
    #!/bin/bash

    # Define the web directory
    TARGET="/var/www/html/mywebsite"
    GIT_DIR="/var/repo/mywebsite.git"

    # Navigate to the web directory and pull the latest changes
    echo "Deploying to $TARGET"
    cd $TARGET || exit
    git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f

    # Restart the application if necessary (Node.js, Django, etc.)
    if [ -f $TARGET/app.js ]; then
        echo "Restarting Node.js app with PM2..."
        pm2 restart mywebsite
    fi

    if [ -f $TARGET/venv/bin/gunicorn ]; then
        echo "Restarting Gunicorn for Django..."
        sudo systemctl restart mywebsite
    fi

    echo "Deployment completed successfully!"
    ```

4. **Make the post-receive hook executable**:

    ```bash
    sudo chmod +x /var/repo/mywebsite.git/hooks/post-receive
    ```

#### **Step 2: Push Code to Trigger Deployment**
Now, whenever you push code to the remote repository (`/var/repo/mywebsite.git`), the post-receive hook will automatically deploy it to `/var/www/html/mywebsite`.

1. Add the remote repository on your local machine:

    ```bash
    git remote add server ssh://username@server-ip:/var/repo/mywebsite.git
    ```

2. Push your changes to the server:

    ```bash
    git push server main
    ```

3. The code will be deployed to `/var/www/html/mywebsite` and any services (e.g., Nginx, PM2, Gunicorn) will be restarted as needed.

---

### **Option 3: Automating Deployment Using a CI/CD Tool (GitHub Actions)**

You can use **GitHub Actions** to automate the deployment process every time you push code to your repository.

#### **Step 1: Create a GitHub Actions Workflow**
1. In your GitHub repository, create a new folder called `.github/workflows`:

    ```bash
    mkdir -p .github/workflows
    ```

2. Create a deployment YAML file inside that folder (e.g., `deploy.yml`):

    ```bash
    nano .github/workflows/deploy.yml
    ```

3. Add the following content to define the deployment workflow:

    ```yaml
    name: Deploy to Server

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

        - name: Copy files via SSH
          uses: appleboy/scp-action@v0.1.1
          with:
            host: ${{ secrets.SERVER_IP }}
            username: ${{ secrets.USERNAME }}
            key: ${{ secrets.SSH_KEY }}
            source: "."
            target: "/var/www/html/mywebsite"

        - name: Restart Nginx
          run: |
            ssh ${{ secrets.USERNAME }}@${{ secrets.SERVER_IP }} 'sudo systemctl restart nginx'
    ```

#### **Step 2: Set Up GitHub Secrets**
1. In your GitHub repository, go to **Settings > Secrets**.
2. Add the following secrets:
   - `SERVER_IP`: The IP address of your server.
   - `USERNAME`: The username for SSH.
   - `SSH_KEY`: Your private SSH key (base64-encoded).

#### **Step 3: Push Code to Trigger Deployment**
Now, every time you push code to the `main` branch, GitHub Actions will automatically deploy the code to your server via SSH and restart Nginx.

---

### **Outcome of Stage 3**
By following these steps, you’ll have an automated deployment system set up. You can choose between:
1. **Bash script automation** using `cron`.
2. **Git hooks** to automatically deploy when code is pushed to the server.
3. **CI/CD pipelines** like GitHub Actions to automate deployment from a GitHub repository.

These methods ensure that every update to your website is deployed quickly and consistently without manual intervention.