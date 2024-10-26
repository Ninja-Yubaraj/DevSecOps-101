Here’s a detailed breakdown of **Stage 2: Linux Web Server Setup** where you will set up a basic web server to host a static website on a Linux server.

### **Stage 2: Linux Web Server Setup (Detailed Guide)**

**Prerequisites:**
- You have a Linux server (can be a VPS like your **Hostinger VPS** or a local virtual machine).
- You have basic knowledge of Linux commands and SSH access to your server.

#### Step 1: SSH into Your Linux Server

First, you need to access your Linux server remotely via SSH.

1. Open your terminal and enter the following command to connect to your server (replace `username` and `server-ip` with your own credentials):
   
   ```bash
   ssh username@server-ip
   ```

2. If you’re using SSH keys for authentication (recommended for security), make sure you have the private key available and use it like this:

   ```bash
   ssh -i /path/to/private-key username@server-ip
   ```

#### Step 2: Install Nginx or Apache

You can choose either **Nginx** or **Apache** as your web server. For simplicity, we’ll cover **Nginx**, but if you prefer Apache, I can guide you through that too.

1. **Update your system’s package list** to make sure you have the latest versions of the software:
   
   ```bash
   sudo apt update
   ```

2. **Install Nginx** with the following command:

   ```bash
   sudo apt install nginx -y
   ```

   This installs the web server and starts it automatically.

3. To check if Nginx is running, you can use:

   ```bash
   sudo systemctl status nginx
   ```

4. You can now check if your Nginx web server is working by opening your browser and navigating to `http://your-server-ip`. You should see the default Nginx welcome page.

#### Step 3: Configure Nginx to Serve Your Static Website

1. **Create a directory** for your website files. We’ll use `/var/www/html/mywebsite` as an example:

   ```bash
   sudo mkdir -p /var/www/html/mywebsite
   ```

2. **Copy your static website files** (HTML, CSS, JS, etc.) into this directory. If you have them locally, you can use `scp` to copy the files from your local machine to your server. For example:

   ```bash
   scp -r /path/to/local/website/ username@server-ip:/var/www/html/mywebsite
   ```

3. **Set proper permissions** so that Nginx can serve the files. Use these commands to give the correct ownership to the web server user (usually `www-data` for Nginx):

   ```bash
   sudo chown -R www-data:www-data /var/www/html/mywebsite
   ```

4. **Edit the Nginx configuration** to point to your website’s directory. Create a new configuration file or modify the default one.

   To create a new config file, do the following:

   ```bash
   sudo nano /etc/nginx/sites-available/mywebsite
   ```

   Add the following configuration (make sure to replace `mywebsite.com` with your actual domain or server IP):

   ```nginx
   server {
       listen 80;
       server_name mywebsite.com www.mywebsite.com;

       root /var/www/html/mywebsite;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

5. **Enable the new configuration** by creating a symbolic link from `sites-available` to `sites-enabled`:

   ```bash
   sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/
   ```

6. **Test the configuration** to make sure there are no errors:

   ```bash
   sudo nginx -t
   ```

7. **Reload Nginx** to apply the changes:

   ```bash
   sudo systemctl reload nginx
   ```

#### Step 4: Configure the Firewall

Make sure that the firewall on your Linux server allows HTTP traffic. If you’re using **UFW** (Uncomplicated Firewall), you can allow HTTP and HTTPS traffic like this:

1. Check the current firewall status:

   ```bash
   sudo ufw status
   ```

2. Allow traffic on port 80 (HTTP) and 443 (HTTPS):

   ```bash
   sudo ufw allow 'Nginx Full'
   ```

3. Enable UFW (if not already enabled):

   ```bash
   sudo ufw enable
   ```

4. Verify that the rules have been added:

   ```bash
   sudo ufw status
   ```

#### Step 5: Test the Setup

1. Open a web browser and navigate to `http://your-server-ip` or `http://your-domain`. You should see your static website loaded from the Nginx server.

2. If you have a domain name, update your DNS settings to point to your server's IP, so the website is accessible via `http://your-domain.com`.

#### Step 6: Optional – Set up HTTPS with Let's Encrypt

1. Install **Certbot**, the tool for requesting SSL certificates from Let's Encrypt:

   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. Request a certificate for your domain:

   ```bash
   sudo certbot --nginx -d mywebsite.com -d www.mywebsite.com
   ```

3. Follow the on-screen instructions. Certbot will automatically configure Nginx to use HTTPS.

4. Verify that HTTPS is working by visiting your site at `https://mywebsite.com`.

5. Certbot will also set up automatic certificate renewal, but you can verify this by running:

   ```bash
   sudo certbot renew --dry-run
   ```

### **Outcome of Stage 2:**
At this point, your Linux server will have Nginx installed, hosting your static website. It will also have a basic firewall setup to allow only necessary traffic, and you can optionally secure your site with HTTPS using Let's Encrypt. Your website will be live and accessible via the server’s IP or domain.
