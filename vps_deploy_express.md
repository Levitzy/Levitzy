Below is a full documentation on how to deploy an **Express.js app** on an **Ubuntu VPS** server. The goal is to create a persistent, production-ready server for your application.

---

# **Complete Guide to Deploy an Express.js App on a VPS Running Ubuntu**

---

## **1. Prerequisites**
- A VPS with Ubuntu installed (e.g., from AWS, DigitalOcean, Linode).
- Root or a user with `sudo` privileges.
- A domain name (optional but recommended).
- Basic knowledge of Linux commands.

---

## **2. Setting Up the VPS**

1. **Update and Upgrade the System**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Required Tools**:
   Install `curl` and other basic utilities:
   ```bash
   sudo apt install curl -y
   ```

3. **Install Node.js and npm**:
   Use the NodeSource repository for the latest version:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt install -y nodejs
   ```
   Verify installation:
   ```bash
   node -v
   npm -v
   ```

---

## **3. Transfer Your Express App to the VPS**

1. **Clone from Git Repository**:
   If your app is in a Git repository:
   ```bash
   git clone <repository-url>
   cd <repository-folder>
   ```

2. **Upload Files Directly**:
   If not using Git, transfer files via SCP:
   ```bash
   scp -r /path/to/your/project user@vps-ip:/path/to/destination
   ```

3. **Install Dependencies**:
   Navigate to the project folder and install the dependencies:
   ```bash
   cd /path/to/your/project
   npm install
   ```

4. **Test Your App**:
   Start the app:
   ```bash
   node index.js
   ```
   Access the server by visiting:
   ```
   http://<vps-ip>:<port>
   ```

---

## **4. Install `pm2` to Manage Your App**

`pm2` is a process manager for Node.js that ensures your app runs persistently.

1. **Install pm2**:
   ```bash
   sudo npm install -g pm2
   ```

2. **Start Your App with pm2**:
   ```bash
   pm2 start index.js --name "express-app"
   ```

3. **Save the Process List**:
   ```bash
   pm2 save
   ```

4. **Set Up pm2 Startup Script**:
   ```bash
   pm2 startup
   ```
   Follow the instructions to enable pm2 on boot.

5. **Check Running Processes**:
   ```bash
   pm2 list
   ```

---

## **5. Configure Nginx as a Reverse Proxy**

1. **Install Nginx**:
   ```bash
   sudo apt install nginx -y
   ```

2. **Create a Configuration File**:
   ```bash
   sudo nano /etc/nginx/sites-available/express-app
   ```

3. **Add the Following Configuration**:
   Replace `<your-domain>` and `localhost:3000` with your domain name and app port:
   ```nginx
   server {
       listen 80;
       server_name your-domain.com;

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

4. **Enable the Configuration**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/express-app /etc/nginx/sites-enabled/
   sudo nginx -t  # Test the configuration
   sudo systemctl reload nginx
   ```

---

## **6. Set Up SSL with Certbot**

1. **Install Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. **Obtain an SSL Certificate**:
   Replace `your-domain.com` with your actual domain name:
   ```bash
   sudo certbot --nginx -d your-domain.com
   ```

3. **Test SSL Auto-Renewal**:
   ```bash
   sudo certbot renew --dry-run
   ```

---

## **7. Test and Verify**

1. Access your app in a browser:
   ```
   https://your-domain.com
   ```

2. Check the app logs (if needed):
   ```bash
   pm2 logs
   ```

---

## **8. Bonus: Troubleshooting**

1. **Check If the App Is Running**:
   ```bash
   pm2 list
   ```

2. **Restart the App**:
   ```bash
   pm2 restart express-app
   ```

3. **Stop the App**:
   ```bash
   pm2 stop express-app
   ```

4. **View Nginx Logs**:
   ```bash
   sudo tail -f /var/log/nginx/error.log
   ```

5. **Check Port Usage**:
   ```bash
   sudo netstat -tuln | grep :3000
   ```

---

## **9. Automate Server Security (Optional)**

1. **Set Up a Firewall**:
   ```bash
   sudo ufw allow 'Nginx Full'
   sudo ufw enable
   ```

2. **Keep Your Server Updated**:
   Automate security updates:
   ```bash
   sudo apt install unattended-upgrades -y
   sudo dpkg-reconfigure --priority=low unattended-upgrades
   ```

---

## **10. Summary**

Your Express app is now deployed on an Ubuntu VPS with:
- A reverse proxy using Nginx.
- Persistent app management with `pm2`.
- SSL encryption using Certbot.

Let me know if you need further clarifications or assistance!
