Below is a comprehensive step-by-step **Flask Deployment Documentation** to deploy a Flask application on a VPS. It includes options to configure a custom domain, add a username, and secure the server with HTTPS.

---

## **Flask Deployment on VPS with Optional Domain and HTTPS**

### **1. Prerequisites**
- A VPS with a public IP address.
- SSH access to the VPS.
- A registered domain name (optional).
- Basic knowledge of Linux commands.

---

### **2. Setting Up the VPS**

#### **2.1 Update and Install Required Packages**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-venv python3-pip nginx git -y
```

---

### **3. Create a Flask Application**

#### **3.1 Create a User (Optional)**
If you want to create a specific user for your Flask project:
```bash
sudo adduser <username>
sudo usermod -aG sudo <username>
```

Switch to the new user:
```bash
su - <username>
```

#### **3.2 Set Up the Flask Application**
1. Create a project directory:
   ```bash
   mkdir -p ~/flaskapp
   cd ~/flaskapp
   ```

2. Create a Python virtual environment:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

3. Install Flask and Gunicorn:
   ```bash
   pip install flask gunicorn
   ```

4. Create a `webhook.py` file (or use `app.py`) as the entry point:
   ```bash
   nano webhook.py
   ```
   Example Flask app:
   ```python
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def home():
       return "Welcome to Flask on VPS!"

   if __name__ == "__main__":
       app.run()
   ```

5. Test your application locally:
   ```bash
   python3 webhook.py
   ```

---

### **4. Deploy Flask with Gunicorn**

#### **4.1 Create a Gunicorn Service File**
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

4. Check the status:
   ```bash
   sudo systemctl status flaskapp
   ```

---

### **5. Configure Nginx**

#### **5.1 Create an Nginx Configuration File**
1. Create a new file:
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

3. Enable the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
   ```

4. Test and reload Nginx:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

---

### **6. Add a Custom Domain (Optional)**

#### **6.1 Point Your Domain to the VPS**
Update your domain's DNS settings to point to your VPS public IP address:
- **A Record**: `@` → `YOUR_VPS_IP`
- **A Record**: `www` → `YOUR_VPS_IP`

Use a DNS propagation checker to confirm the settings.

#### **6.2 Update Nginx Configuration**
Modify the `server_name` in `/etc/nginx/sites-available/flaskapp`:
```nginx
server_name yourdomain.com www.yourdomain.com;
```

Reload Nginx:
```bash
sudo systemctl reload nginx
```

---

### **7. Enable HTTPS with Certbot**

#### **7.1 Install Certbot**
```bash
sudo apt install certbot python3-certbot-nginx
```

#### **7.2 Obtain and Install the SSL Certificate**
Run Certbot:
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Certbot will:
1. Verify your domain ownership.
2. Configure Nginx for HTTPS.
3. Redirect HTTP to HTTPS.

#### **7.3 Test Renewal**
```bash
sudo certbot renew --dry-run
```

---

### **8. Test the Deployment**

1. **Local Test**: Access via the server IP:
   ```bash
   http://<your_server_ip>
   ```

2. **Domain Test**: Access via your domain:
   ```bash
   https://yourdomain.com
   ```

3. Check Nginx logs if there are issues:
   ```bash
   sudo tail -n 20 /var/log/nginx/error.log
   ```

---

### **9. Maintenance Commands**

- Restart Flask service:
  ```bash
  sudo systemctl restart flaskapp
  ```

- Restart Nginx:
  ```bash
  sudo systemctl reload nginx
  ```

- Check system logs:
  ```bash
  sudo journalctl -u flaskapp
  ```

---

This documentation ensures a step-by-step approach for deploying Flask on a VPS, with optional domain integration and HTTPS setup. Let me know if you need further clarification!
