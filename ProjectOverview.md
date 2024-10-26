# **Project: Basic Web Server Setup with Automated Security Audits**

**Project Overview:**
Set up a basic static website on a Linux server, automate the deployment, and ensure security checks are regularly performed.

**Key Objectives:**
- **Web Development:** Set up a simple static HTML/CSS website.
- **Linux:** Host the website on a Linux server using **Nginx** or **Apache**.
- **DevOps:** Automate deployment of updates to the website using a simple **bash script** or **Git hooks**.
- **Cybersecurity:** Set up basic security measures, such as securing SSH, enabling a firewall, and performing automated vulnerability scans.

### **Steps:**

1. **Stage 1: Website Creation**
   - Create a simple static website using basic HTML, CSS, and possibly a small bit of JavaScript.
   - Keep it as simple as possible (e.g., a personal portfolio or a landing page for an app).

2. **Stage 2: Linux Web Server Setup**
   - Spin up a Linux server (could be your Hostinger VPS or a local virtual machine).
   - Install **Nginx** or **Apache** and host the website on it.
   - Ensure proper file permissions and basic configuration.

3. **Stage 3: Automating Deployment**
   - Write a basic **bash script** that automates the process of pulling the latest changes from a Git repository and updating the website files.
   - Optionally, use **Git hooks** to trigger the deployment automatically when you push updates to the repository.
   - Ensure that the server reloads Nginx/Apache after each update to apply changes.

4. **Stage 4: Security Enhancements**
   - Secure the Linux server:
     - Enable **UFW** or **iptables** to allow only necessary ports (e.g., port 80 for HTTP, port 443 for HTTPS, port 22 for SSH).
     - Secure SSH access (e.g., disable root login, use SSH keys).
   - Install a basic security audit tool like **Lynis** to scan the server for vulnerabilities.
   - Schedule the audit tool to run automatically (e.g., using a **cron job**) and generate a report with security recommendations.

5. **Stage 5: SSL Certificate**
   - Set up **Let's Encrypt** to secure the website with HTTPS.

6. **Stage 6: Testing & Finalizing**
   - Test the website for availability.
   - Run the security audit and apply the recommended improvements (e.g., hardening the server).

### **Outcome:**
By the end of this project, you’ll have a simple static website running on a secure Linux server with automated deployment and basic security audits. It’s a straightforward way to gain hands-on experience with Linux, DevOps automation, and cybersecurity practices without overcomplicating things.
