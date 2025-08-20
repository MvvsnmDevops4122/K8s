
# 🌐 LAB: Hostinger Domain → AWS Route 53 → EC2 (Apache Web Server)

## Part 1 – Buy Domain & Prepare EC2

1. Buy Domain from Hostinger  
   Example: `kkdevopsb4.shop`

2. Launch EC2 Instance (Ubuntu Preferred)  
   - Choose Ubuntu 22.04 LTS (free-tier eligible).  
   - Assign Elastic IP → ensures permanent IP (doesn’t change).  

3. Configure Security Group  
   - Allow TCP 80 (HTTP) – from anywhere.  
   - Allow TCP 443 (HTTPS) – if SSL planned.  
   - Allow TCP 22 (SSH) – only from your IP.  

4. SSH into EC2  
   ```bash
   ssh -i my-key.pem ubuntu@<Elastic-IP>
````


## Part 2 – Install Apache Web Server

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

✅ Test in browser → `http://<Elastic-IP>` → You should see Apache2 Default Page.



## Part 3 – Create Hosted Zone in Route 53

1. Go to AWS → Route 53 → Hosted zones → Create hosted zone.
2. Domain name: `kkdevopsb4.shop`.
3. Type: **Public hosted zone**.
4. Create hosted zone.
5. Copy the 4 NS (Name Server) values.

---

## Part 4 – Update Nameservers in Hostinger

1. Login → Hostinger → Domains → `kkdevopsb4.shop` → DNS / Nameservers
2. Select **Use custom nameservers** → Paste 4 values from Route 53
3. Disable **DNSSEC** (important).



## Part 5 – Create A Records in Route 53

* **Root domain**

  * Record name: (leave blank)
  * Type: A – IPv4 address
  * Value: `<Elastic-IP>`
  * TTL: 300

* **www domain**

  * Record name: www
  * Type: A – IPv4 address
  * Value: `<Elastic-IP>`
  * TTL: 300



## Part 6 – Verify DNS

```bash
nslookup kkdevopsb4.shop 8.8.8.8
nslookup www.kkdevopsb4.shop 8.8.8.8
```

✅ Both should return `<Elastic-IP>`.

---

## Part 7 – Deploy Your Website

1. Remove default Apache page:

   ```bash
   sudo rm /var/www/html/index.html
   ```

2. Upload your site files:

   ```bash
   sudo vi /var/www/html/index.html
   ```

Example HTML page:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CarePlus Hospital</title>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #0f172a; color: #e2e8f0; }
    header { display: flex; justify-content: space-between; align-items: center; background: #1e293b; padding: 15px 40px; }
    header h1 { color: #38bdf8; margin: 0; }
    nav a { color: #e2e8f0; text-decoration: none; margin: 0 15px; }
    nav a:hover { color: #38bdf8; }
    .hero { text-align: center; padding: 80px 20px; background: linear-gradient(145deg, #0f172a, #1e293b); }
    .hero h2 { font-size: 2.5rem; margin-bottom: 20px; color: #fff; }
    footer { background: #1e293b; color: #cbd5e1; text-align: center; padding: 20px; margin-top: 30px; }
  </style>
</head>
<body>
  <header>
    <h1>CarePlus Hospital</h1>
    <nav>
      <a href="#departments">Departments</a>
      <a href="#doctors">Doctors</a>
      <a href="#appointment">Appointment</a>
      <a href="#contact">Contact</a>
    </nav>
  </header>
  <section class="hero">
    <h2>Compassionate care, advanced medicine.</h2>
    <p>Multispeciality hospital with modern facilities.</p>
    <button>Book Appointment</button>
  </section>
  <footer>
    <p>&copy; 2025 CarePlus Hospital | All Rights Reserved</p>
  </footer>
</body>
</html>
```



## Part 8 – Enable HTTPS (Optional)

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache -d kkdevopsb4.shop -d www.kkdevopsb4.shop
```

✅ SSL will auto-renew.



## 🔄 Final Flow Diagram (Simplified)

```
User Browser
     │
     ▼
Domain: kkdevopsb4.shop (Hostinger)
     │
(Custom Nameservers)
     ▼
AWS Route 53 (Public Hosted Zone)
     │
(A Records → Elastic IP)
     ▼
AWS EC2 Instance (Apache Web Server)
     │
Website Content (HTML / App)
```

```

---

👉 This will render properly on GitHub with syntax highlighting and sections.  

Do you also want me to **append a Troubleshooting section** (DNS not resolving, SSL failure, Apache issues) at the bottom of this `README.md`?
```
