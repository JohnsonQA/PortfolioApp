# Portfolio App Deployment on AWS

This project is a Flask-based portfolio application deployed on AWS EC2 using Gunicorn and Nginx. It demonstrates a real-world setup of application hosting, reverse proxy configuration, and HTTPS enablement.

---

## Application Setup

Clone the repository and set up the Python environment:

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python3 app.py

This runs the app locally for development. For production, Gunicorn is used instead of Flask’s built-in server.

---

## Why Gunicorn

Flask’s default server is not suitable for production. Gunicorn acts as a production-grade server that:

- Handles multiple concurrent requests
- Runs the application efficiently using worker processes
- Keeps the backend isolated from direct internet traffic

The app runs internally on:

127.0.0.1:8000

---

## EC2 Deployment Steps

- Launch EC2 instance
- Install required packages:

sudo yum install git nginx -y

- Clone repository and activate virtual environment
- Install dependencies and start the app using Gunicorn
- Ensure port 8000 is accessible internally

At this stage, the app is running but only accessible via IP and port.

---

## Nginx Configuration and Routing

Nginx is used as a reverse proxy to expose the app on standard web ports.

Basic configuration:

server {
listen 80;
server_name static.app.infralabx.space www.static.app.infralabx.space
;

location / {
proxy_pass http://127.0.0.1:8000;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
}

}

### How routing works

- User sends request to domain
- Nginx receives request on port 80
- Nginx forwards request to backend (Gunicorn on port 8000)
- Response is sent back to user

Flow:

User → Nginx (80) → Gunicorn (8000) → Flask App

This setup improves security and scalability by separating frontend and backend layers.

---

## Domain Setup

Domain is purchased externally and connected using AWS Route 53.

Steps:

- Create a hosted zone in Route 53
- Update nameservers in domain provider
- Point domain to EC2 public IP
- Verify using nslookup

---

## HTTP vs HTTPS

- HTTP (port 80) sends data in plain text
- HTTPS (port 443) encrypts communication using SSL/TLS

HTTPS ensures:

- Secure data transfer
- Trusted connection via certificate
- Protection against interception

---

## SSL Setup using Certbot

SSL certificates are generated using Certbot.

sudo yum install certbot python3-certbot-nginx -y

sudo certbot --nginx
-d static.app.infralabx.space
-d www.static.app.infralabx.space

### What Certbot does

- Verifies domain ownership
- Generates SSL certificates
- Stores them in:

/etc/letsencrypt/live/your-domain/

- Updates Nginx configuration automatically
- Enables HTTPS and optional redirection

---

## HTTP to HTTPS Redirection

After SSL setup, traffic is redirected securely:

return 301 https://$host$request_uri;

This ensures:

- All HTTP requests move to HTTPS
- Full URL path and query parameters are preserved

---

## Final Architecture

User → HTTPS (443)
→ Nginx (SSL termination)
→ Gunicorn (8000)
→ Flask App

---

## Certificate Renewal

Certificates expire every 90 days. Renewal can be tested using:

sudo certbot renew --dry-run

---

## Summary

- Flask app runs on port 8000 using Gunicorn
- Nginx handles incoming traffic and routing
- Domain is mapped using Route 53
- Certbot enables HTTPS with automatic configuration
- Secure communication is enforced using redirection
