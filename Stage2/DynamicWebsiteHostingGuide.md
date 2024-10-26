If the site you want to host is **not static** (e.g., it’s a dynamic site built with frameworks like **Node.js**, **Django**, **Flask**, **PHP**, etc.), you will need to make some additional adjustments to your setup. Below is a guide based on popular scenarios for hosting dynamic websites.

### **Key Differences When Hosting a Dynamic Website**

1. **Dynamic Content Processing:** Dynamic websites involve backend logic, databases, and potentially other services like authentication, session management, etc.
2. **Server-Side Framework Support:** You’ll need to configure your web server (Nginx/Apache) to pass requests to the appropriate backend service (e.g., Node.js, Python, PHP).
3. **Database Setup:** Most dynamic websites require a database (e.g., MySQL, PostgreSQL, MongoDB).

---

### **Steps to Host a Dynamic Website**

#### 1. Hosting a Node.js Application

**Step 1:** Install Node.js on your server.

```bash
sudo apt install nodejs npm -y
```

**Step 2:** Upload your Node.js application files to the server.

**Step 3:** Install the required dependencies for your app (assuming you have a `package.json` file).

```bash
cd /var/www/html/mywebsite
npm install
```

**Step 4:** Set up your app to run in the background using **PM2** (a process manager for Node.js).

```bash
sudo npm install -g pm2
pm2 start app.js --name "mywebsite"
pm2 save
pm2 startup
```

**Step 5:** Configure Nginx to reverse proxy to your Node.js app. Edit your Nginx config:

```bash
sudo nano /etc/nginx/sites-available/mywebsite
```

Add this configuration to forward HTTP requests to your Node.js app (running on port 3000, for example):

```nginx
server {
    listen 80;
    server_name mywebsite.com www.mywebsite.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Step 6:** Reload Nginx.

```bash
sudo systemctl reload nginx
```

#### 2. Hosting a Python Web Application (Flask/Django)

**Step 1:** Install Python and the required packages.

```bash
sudo apt install python3-pip python3-venv
```

**Step 2:** Create a virtual environment and install your app dependencies.

```bash
cd /var/www/html/mywebsite
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**Step 3:** Use **Gunicorn** (Python WSGI HTTP Server) to run your app.

```bash
pip install gunicorn
gunicorn --workers 3 app:app
```

**Step 4:** Set up **Gunicorn** to run your Flask/Django app as a background service using **systemd**.

Create a service file:

```bash
sudo nano /etc/systemd/system/mywebsite.service
```

Add this content:

```bash
[Unit]
Description=Gunicorn instance to serve mywebsite
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/html/mywebsite
ExecStart=/var/www/html/mywebsite/venv/bin/gunicorn --workers 3 --bind unix:mywebsite.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

Start and enable the service:

```bash
sudo systemctl start mywebsite
sudo systemctl enable mywebsite
```

**Step 5:** Configure Nginx to serve your app via Gunicorn:

```bash
sudo nano /etc/nginx/sites-available/mywebsite
```

Add this:

```nginx
server {
    listen 80;
    server_name mywebsite.com www.mywebsite.com;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/html/mywebsite/mywebsite.sock;
    }
}
```

**Step 6:** Reload Nginx.

```bash
sudo systemctl reload nginx
```

#### 3. Hosting a PHP Application

**Step 1:** Install **PHP** and **MySQL** (or the database you are using).

```bash
sudo apt install php php-fpm php-mysql -y
```

**Step 2:** Upload your PHP website files to the `/var/www/html/mywebsite` directory.

**Step 3:** Configure Nginx to pass PHP requests to PHP-FPM.

```bash
sudo nano /etc/nginx/sites-available/mywebsite
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name mywebsite.com www.mywebsite.com;

    root /var/www/html/mywebsite;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

**Step 4:** Reload Nginx.

```bash
sudo systemctl reload nginx
```

#### 4. Database Setup (If Required)

1. **Install MySQL** (or any database you need).

```bash
sudo apt install mysql-server
```

2. **Secure the installation**:

```bash
sudo mysql_secure_installation
```

3. **Create a database and user for your app**:

```sql
CREATE DATABASE mywebsite;
CREATE USER 'mywebsiteuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mywebsite.* TO 'mywebsiteuser'@'localhost';
FLUSH PRIVILEGES;
```

4. Update your web application to connect to this database with the credentials.

---

### **Outcome:**
After following these steps, you’ll be able to host a **dynamic website** with Node.js, Python (Flask/Django), or PHP, with appropriate configurations for Nginx to handle dynamic requests and serve your application. You can also secure the site with SSL, just as you would for a static site.
