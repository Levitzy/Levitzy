### **Deploying Flask on Debian VPS (Root User Configuration)**

This guide is specifically designed to deploy Flask using the **root** user on a Debian-based VPS. It also includes instructions for domain configuration and HTTPS setup.

---

## **1. Prerequisites**
- Debian VPS with root access.
- Public IP and optionally, a domain name.
- Basic Linux command knowledge.

---

## **2. Initial Server Setup**

### **2.1 Update System Packages**
```bash
apt update && apt upgrade -y
```

### **2.2 Install Required Packages**
```bash
apt install python3 python3-venv python3-pip nginx git certbot python3-certbot-nginx -y
```

---

## **3. Set Up Flask Application**

### **3.1 Create Project Directory**
```bash
mkdir -p /root/flaskapp
cd /root/flaskapp
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

### **3.4 Create a Flask Application**
1. Create a file named `webhook.py`:
   ```bash
   nano /root/flaskapp/webhook.py
   ```

2. Add the following sample Flask app code:
   ```python
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def home():
       return "Welcome to Flask running as root!"

   if __name__ == "__main__":
       app.run()
   ```

3. Test the application:
   ```bash
   python3 webhook.py
   ```

   Visit `http://<your_server_ip>:5000` to confirm it's running.

4. Deactivate the virtual environment:
   ```bash
   deactivate
   ```

---

## **4. Deploy Flask with Gunicorn**

### **4.1 Create a Systemd Service**
1. Create the service file:
   ```bash
   nano /etc/systemd/system/flaskapp.service
   ```

2. Add the following:
   ```ini
   [Unit]
   Description=Gunicorn instance to serve Flask App
   After=network.target

   [Service]
   User=root
   Group=www-data
   WorkingDirectory=/root/flaskapp
   ExecStart=/root/flaskapp/venv/bin/gunicorn --workers 3 --bind unix:/root/flaskapp/flaskapp.sock webhook:app

   [Install]
   WantedBy=multi-user.target
   ```

3. Enable and start the service:
   ```bash
   systemctl daemon-reload
   systemctl start flaskapp
   systemctl enable flaskapp
   ```

4. Check the status:
   ```bash
   systemctl status flaskapp
   ```

---

## **5. Configure Nginx as a Reverse Proxy**

### **5.1 Create an Nginx Configuration File**
1. Create a file:
   ```bash
   nano /etc/nginx/sites-available/flaskapp
   ```

2. Add the configuration:
   ```nginx
   server {
       listen 80;
       server_name <your_domain> <your_server_ip>;

       location / {
           proxy_pass http://unix:/root/flaskapp/flaskapp.sock;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

3. Enable the site and reload Nginx:
   ```bash
   ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
   nginx -t
   systemctl reload nginx
   ```

4. Visit `http://<your_server_ip>` to test the application.

---

## **6. Add a Custom Domain and HTTPS**

### **6.1 Point Domain to Server**
- Update DNS with your domain registrar:
  - Add **A Record**: `@` → `<your_server_ip>`
  - Add **A Record**: `www` → `<your_server_ip>`

Verify DNS propagation:
```bash
nslookup <your_domain>
```

### **6.2 Obtain SSL Certificate**
1. Run Certbot to configure HTTPS:
   ```bash
   certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

2. Verify HTTPS:
   Visit `https://yourdomain.com` in your browser.

3. Test renewal:
   ```bash
   certbot renew --dry-run
   ```

---

## **7. Managing Flask and Nginx**

### **7.1 Flask Service Management**
- Start the Flask service:
  ```bash
  systemctl start flaskapp
  ```
- Stop the Flask service:
  ```bash
  systemctl stop flaskapp
  ```
- Restart the Flask service:
  ```bash
  systemctl restart flaskapp
  ```
- Check logs:
  ```bash
  journalctl -u flaskapp
  ```

### **7.2 Nginx Management**
- Reload Nginx:
  ```bash
  systemctl reload nginx
  ```

---

## **8. Updating the Flask Application**

1. Edit your app file (e.g., `webhook.py`):
   ```bash
   nano /root/flaskapp/webhook.py
   ```

2. Restart Gunicorn to apply changes:
   ```bash
   systemctl restart flaskapp
   ```

---

## **9. Managing the Virtual Environment**

### **9.1 Activate Virtual Environment**
```bash
source /root/flaskapp/venv/bin/activate
```

### **9.2 Deactivate Virtual Environment**
```bash
deactivate
```

---

## **10. Advanced Configurations**

### **10.1 Debugging Errors**
- Check Flask service logs:
  ```bash
  journalctl -u flaskapp
  ```
- Check Nginx logs:
  ```bash
  tail -n 20 /var/log/nginx/error.log
  ```

### **10.2 Add Additional Applications**
Repeat steps 3–5 for new applications, using unique ports or `server_name`.

---

## **Conclusion**
This comprehensive guide ensures Flask deployment on a Debian VPS with the root user. For domain and HTTPS, follow the optional steps. Let me know if further details are needed!
