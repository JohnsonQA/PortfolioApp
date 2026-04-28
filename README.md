# PortfolioApp
Portfolio App to deploy in AWS cloud

A modern Flask-based portfolio website showcasing DevOps expertise, educational programs, and professional achievements.


## Installation Steps

1. Install Dependencies

```bash
# Create a virtual Env
python3 -m venv .venv

# Activate Virtual env
source .venv/bin/activate

# We are creating virtual env so that depencies wouldn't be stored in the local systems. It juz works on IDE

# Install dependencies 
pip3 install -r requirements.txt

# Run the app locally
python3 app.py
```

## Domain Setup: Hostinger -> Route53

```
1. Purchase a domain from Hostinger (e.g., infralabx.space)

2. Create a hosted zone in AWS Route 53:
   - Type: Public Hosted Zone

3. Copy AWS Name Servers (NS records)

4. Update them in Hostinger:
   - Domain → Nameservers → Use custom nameservers

5. Wait for DNS propagation (5 mins – 24 hours)

6. Verify:
   nslookup -type=ns infralabx.space or nslookup infralabx.space

7. Output should show AWS Name Servers
```

## Gunicorn and its purporse
> [!NOTE]
> - Flask’s built-in server is intended for development only and is not suitable for production workloads.
> - Gunicorn is a production-grade WSGI server used to run Flask applications:
>   - Handles multiple requests using worker processes
>   - Provides better performance and stability than Flask’s dev server
> - However, Gunicorn is not optimized for serving static files (e.g., images, CSS, JavaScript, videos).
> - To handle this, we use Nginx as a reverse proxy:
>   - Serves static files efficiently
>   - Handles client requests and forwards them to Gunicorn
>   - Provides additional benefits like load balancing, SSL termination, and caching

## Gunicorn Setup

- Add the dependency in `requirements.txt`:
  ```txt
  gunicorn==23.0.0
  ```

- To run the app using Gunicorn (production server):
  ```bash
  gunicorn app:app &
  ```

  ## Deploy the app in EC2 

- Install git 
- Install nginx

  ```bash
   sudo yum install git -y
   git clone <your repo>
   sudo yum install nginx -y
  ```

- Activate virtual Env in AWS

- Add IB rule to allow port 8000 to run the app 

- Update nginx.conf with the IP or domain name 

- check the curl localhost:800 

- Run the app using ip:8000 

> [!NOTE] 
- Above steps works without nginx 

- With nginx, we have to update the proxy pass location block in the server block in /etc/nginx/nginx.conf file

### Steps to point nginx config to flask app

# edit the nginx.conf which is on /etc/nginx/nginx.conf path and add the below config under the server

location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

## How nginx works with Flask App

- Nginx is configured as a reverse proxy to handle incoming HTTP requests on port 80.
- Requests are forwarded to the backend application using `proxy_pass http://127.0.0.1:8000;`, where the app (Gunicorn/Flask) is running.
- `127.0.0.1` (localhost) ensures communication happens within the same server, improving security and performance.
- This architecture decouples the frontend (Nginx) from the backend, enabling better scalability, security (HTTPS), and traffic management.

## Summary

1. Start app → Gunicorn runs on 8000
2. Start Nginx → listens on 80
3. User hits website → Nginx forwards to 8000

- Once cloned the git repo and installed nginx and git. Activate virtual env and install dependencies and run bash script. 
- App get started on 8000 port 
- Updated the proxy pass in nginx.conf
- Restart or start the nginx


## Run Gunicorn as a Service (Systemd)

### Steps

1. **Create service file**

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

2. **Add config**

```ini
[Unit]
Description=Gunicorn service
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/PortfolioApp
Environment="PATH=/home/ec2-user/PortfolioApp/.venv/bin"

ExecStart=/home/ec2-user/PortfolioApp/.venv/bin/gunicorn \
          --workers 3 \
          --bind 127.0.0.1:8000 \
          app:app

Restart=always

[Install]
WantedBy=multi-user.target
```

3. **Reload systemd**

```bash
sudo systemctl daemon-reload
```

4. **Start service**

```bash
sudo systemctl start gunicorn
```

5. **Enable on boot**

```bash
sudo systemctl enable gunicorn
```

---

### Useful Commands

```bash
sudo systemctl status gunicorn     # check status
sudo systemctl restart gunicorn    # restart service
sudo journalctl -u gunicorn -f     # view logs
```

---

### Notes

* App runs on `127.0.0.1:8000`
* Do NOT run Gunicorn manually
* Ensure port 8000 is free

---

### Flow

```text
User → Nginx → Gunicorn → Flask App
```

### Use Cert Bot to get SSL certs from CAs

```bash
# install certbot
sudo yum install certbot python3-certbot-nginx -y

# request certifictate
sudo certbot --nginx -d static.app.infralabx.space -d www.static.app.infralabx.space
# Certbot auto-modifies your nginx.conf to add SSL config.

# Certbot creates/saved the certs in the following path /etc/letsencrypt/live/static.app.infralabx.space/

# It just saves the certs but to instal we have to use below command
certbot install --cert-name static.demo.livingdevops.org
# you can autorenew them
# test auto renevew
sudo certbot renew --dry-run

# certs are saved at location
# /etc/letsencrypt/live/yourdomain.com/fullchain.pem   # certificate
# /etc/letsencrypt/live/yourdomain.com/privkey.pem      # private key
```




