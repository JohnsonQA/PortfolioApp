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