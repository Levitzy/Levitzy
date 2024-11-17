Below is the **Full Documentation for Deploying a Flask Server on Ubuntu VPS**, including steps to activate and deactivate a virtual environment, adding a custom domain, and setting up HTTPS.

---

# **Deploying a Flask Application on Ubuntu VPS**

This guide walks you through deploying a Flask application on an Ubuntu VPS with Nginx, Gunicorn, and optional domain and HTTPS configuration.

---

## **1. Prerequisites**
- Ubuntu VPS with public IP.
- SSH access to the server.
- A registered domain name (optional).
- Basic knowledge of Linux commands.

---

## **2. Initial Setup**

### **2.1 Update the System**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-venv python3-pip nginx git -y
```

### **2.2 Create a User (Optional)**
Create a user for your Flask application:
```bash
sudo adduser <username>
sudo usermod -aG sudo <username>
su - <username>
```

---

## **3. Flask Application Setup**

### **3.1 Create the Project Directory**
```bash
mkdir -p ~/flaskapp
cd ~/flaskapp
```

### **3.2 Create a Python Virtual Environment**
```bash
python3 -m venv venv
source venv/bin/activate
```

### **3.3 Install Flask and Gunicorn**
```bash
pip install flask gunicorn
```

### **3.4 Create the Flask Application**
1. Create a `webhook.py` file (or another file for your Flask app):
   ```bash
   nano webhook.py
   ```

2. Add the following sample code:
   ```python
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def home():
       return "Welcome to Flask on VPS!"

   if __name__ == "__main__":
       app.run()
   ```

3. Test the application:
   ```bash
   python3 webhook.py
   ```

   Visit `http://<your_server_ip>:5000` to confirm it's running.

4. **Deactivate the Virtual Environment**:
   ```bash
   deactivate
   ```

---

## **4. Deploy Flask with Gunicorn**

### **4.1 Create a Gunicorn Systemd Service**
1. Create a systemd service file:
   ```bash
   sudo nano /etc/systemd/system/flaskapp.service
   ```

2. Add the following configuration:
   ```ini
   [Unit]
   Description=Gunicorn instance to serve Flask App
   After=network.target

   [Service]
   User=<username>
   Group=www-data
   WorkingDirectory=/home/<username>/flaskapp
   ExecStart=/home/<username>/flaskapp/venv/bin/gunicorn --workers 3 --bind unix:/home/<username>/flaskapp/flaskapp.sock webhook:app

   [Install]
   WantedBy=multi-user.target
   ```

3. Reload systemd and start the service:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start flaskapp
   sudo systemctl enable flaskapp
   ```

4. Verify that Gunicorn is running:
   ```bash
   sudo systemctl status flaskapp
   ```

---

## **5. Configure Nginx as a Reverse Proxy**

### **5.1 Create an Nginx Configuration File**
1. Create a new Nginx configuration file:
   ```bash
   sudo nano /etc/nginx/sites-available/flaskapp
   ```

2. Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name <your_domain> <your_server_ip>;

       location / {
           proxy_pass http://unix:/home/<username>/flaskapp/flaskapp.sock;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

3. Enable the site and test Nginx:
   ```bash
   sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
   sudo nginx -t
   sudo systemctl reload nginx
   ```

4. Visit `http://<your_server_ip>` to test your application.

---

## **6. Add a Custom Domain (Optional)**

### **6.1 Point Your Domain to Your Server**
In your domain registrar's DNS settings:
- Add an **A Record** pointing `@` to your VPS IP.
- Add another **A Record** pointing `www` to your VPS IP.

Verify the DNS resolution:
```bash
nslookup <your_domain>
```

### **6.2 Update Nginx Configuration**
Modify `/etc/nginx/sites-available/flaskapp` to include your domain:
```nginx
server_name yourdomain.com www.yourdomain.com;
```

Reload Nginx:
```bash
sudo systemctl reload nginx
```

---

## **7. Enable HTTPS with Certbot**

### **7.1 Install Certbot**
```bash
sudo apt install certbot python3-certbot-nginx
```

### **7.2 Obtain an SSL Certificate**
Run Certbot to generate and configure HTTPS:
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Certbot will:
- Verify domain ownership.
- Automatically update your Nginx configuration to use HTTPS.

### **7.3 Test HTTPS**
Visit `https://yourdomain.com` to verify secure access.

### **7.4 Enable Automatic Renewal**
Certbot sets up auto-renewal by default. Test the renewal process:
```bash
sudo certbot renew --dry-run
```

---

## **8. Managing Flask and Nginx**

### **8.1 Common Commands**
- Start/Stop/Restart Flask service:
  ```bash
  sudo systemctl start flaskapp
  sudo systemctl stop flaskapp
  sudo systemctl restart flaskapp
  ```

- Check Flask service logs:
  ```bash
  sudo journalctl -u flaskapp
  ```

- Reload Nginx:
  ```bash
  sudo systemctl reload nginx
  ```

---

## **9. Updating the Flask Application**

### **9.1 Edit Your Flask App**
Edit your Flask application (e.g., `webhook.py`) and save changes:
```bash
nano ~/flaskapp/webhook.py
```

### **9.2 Restart the Flask Service**
Restart the Gunicorn service to apply changes:
```bash
sudo systemctl restart flaskapp
```

---

## **10. Deactivating and Reactivating Virtual Environment**
- **Activate Virtual Environment**:
   ```bash
   source ~/flaskapp/venv/bin/activate
   ```

- **Deactivate Virtual Environment**:
   ```bash
   deactivate
   ```

---

This documentation provides a complete setup for deploying Flask on Ubuntu VPS with optional domain and HTTPS. Let me know if you need further assistance!
