### **Deploying Flask Application on Debian VPS**

This guide is tailored for Debian-based systems and provides a full setup for deploying a Flask application using Nginx, Gunicorn, and optionally setting up a domain with HTTPS.

---

## **1. Prerequisites**
- Debian VPS with public IP.
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
For better security, create a user:
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
1. Create a file `webhook.py`:
   ```bash
   nano webhook.py
   ```

2. Add this sample Flask application code:
   ```python
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def home():
       return "Welcome to Flask on Debian VPS!"

   if __name__ == "__main__":
       app.run()
   ```

3. Test the application:
   ```bash
   python3 webhook.py
   ```

   Visit `http://<your_server_ip>:5000` to confirm the app is running.

4. Deactivate the virtual environment:
   ```bash
   deactivate
   ```

---

## **4. Deploy Flask with Gunicorn**

### **4.1 Create a Gunicorn Systemd Service**
1. Create the service file:
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

3. Start and enable the service:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start flaskapp
   sudo systemctl enable flaskapp
   ```

4. Verify the service is running:
   ```bash
   sudo systemctl status flaskapp
   ```

---

## **5. Configure Nginx as a Reverse Proxy**

### **5.1 Create an Nginx Configuration File**
1. Create a configuration file:
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

3. Enable the site and reload Nginx:
   ```bash
   sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
   sudo nginx -t
   sudo systemctl reload nginx
   ```

4. Visit `http://<your_server_ip>` to test the app.

---

## **6. Add a Custom Domain (Optional)**

### **6.1 Point Your Domain to Your Server**
Update your DNS settings with your domain registrar:
- Add an **A Record** pointing `@` to your VPS IP.
- Add another **A Record** pointing `www` to your VPS IP.

Verify the DNS resolution:
```bash
nslookup <your_domain>
```

### **6.2 Update Nginx Configuration**
Edit `/etc/nginx/sites-available/flaskapp` to include your domain:
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
Run Certbot:
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Certbot will verify your domain ownership and update your Nginx configuration for HTTPS.

### **7.3 Test HTTPS**
Visit `https://yourdomain.com` to verify HTTPS is working.

### **7.4 Enable Automatic Renewal**
Test Certbot's renewal process:
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
Modify your Flask app (e.g., `webhook.py`) as needed:
```bash
nano ~/flaskapp/webhook.py
```

### **9.2 Restart the Flask Service**
Restart Gunicorn to apply changes:
```bash
sudo systemctl restart flaskapp
```

---

## **10. Managing the Virtual Environment**

### **10.1 Activate Virtual Environment**
To activate:
```bash
source ~/flaskapp/venv/bin/activate
```

### **10.2 Deactivate Virtual Environment**
To deactivate:
```bash
deactivate
```

---

## **Conclusion**
This guide covers deploying Flask on Debian with optional domain and HTTPS configuration. Let me know if you need further clarifications or customizations!
