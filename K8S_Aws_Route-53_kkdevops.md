
# 🌐 LAB: Hosting a Website Using Hostinger Domain, AWS Route 53 & EC2 (Apache Web Server)

This lab demonstrates how to host a website using a **Hostinger domain**, **AWS Route 53 DNS**, and an **EC2 instance running Apache**.



## 1️⃣ Purchase Domain & Prepare EC2

1. **Buy a domain** from Hostinger, for example: `kkdevopsb4.shop`.

2. **Launch an EC2 instance** on AWS:
   - Select **Ubuntu 22.04 LTS** (free-tier eligible).  
   - Allocate an **Elastic IP** so the IP remains fixed.

3. **Set up security group rules**:
   - Port **80 (HTTP)**: open to all.  
   - Port **443 (HTTPS)**: only if you plan to use SSL.  
   - Port **22 (SSH)**: restricted to your IP.

4. **Connect via SSH**:

   ```bash
   ssh -i my-key.pem ubuntu@<Elastic-IP>
````



## 2️⃣ Install Apache Web Server

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

✅ Test by opening `http://<Elastic-IP>` in a browser — Apache default page should appear.




## 3️⃣ Configure Route 53 Hosted Zone

1. Go to **AWS Console → Route 53 → Hosted Zones → Create Hosted Zone**.
2. Enter your domain: `kkdevopsb4.shop`.
3. Choose **Public Hosted Zone**.
4. Copy the 4 **NS (Nameserver) values**.

---

## 4️⃣ Update Hostinger Nameservers

1. In Hostinger: **Domains → kkdevopsb4.shop → DNS / Nameservers**.
2. Select **Custom nameservers** and paste the Route 53 NS records.
3. Ensure **DNSSEC is turned off** to avoid propagation issues.




## 5️⃣ Add A Records in Route 53

* **Root domain (`kkdevopsb4.shop`)**:

  * Type: `A`
  * Value: `<Elastic-IP>`

* **WWW subdomain (`www.kkdevopsb4.shop`)**:

  * Type: `A`
  * Value: `<Elastic-IP>`

*TTL can remain default (300).*




## 6️⃣ Check DNS Resolution

```bash
nslookup kkdevopsb4.shop 8.8.8.8
nslookup www.kkdevopsb4.shop 8.8.8.8
```

✅ Both should return your Elastic IP.




## 7️⃣ Deploy Your Website

1. Remove Apache’s default page:

```bash
sudo rm /var/www/html/index.html
```

2. Create a custom HTML page:

```bash
sudo vi /var/www/html/index.html
```

### Example HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CarePlus Hospital</title>
  <style>
    body { margin:0; font-family: Arial, sans-serif; background:#0f172a; color:#e2e8f0; }
    header { display:flex; justify-content:space-between; align-items:center; background:#1e293b; padding:15px 40px; }
    header h1 { color:#38bdf8; margin:0; }
    nav a { color:#e2e8f0; text-decoration:none; margin:0 15px; }
    nav a:hover { color:#38bdf8; }
    .hero { text-align:center; padding:80px 20px; background: linear-gradient(145deg,#0f172a,#1e293b); }
    .hero h2 { font-size:2.5rem; margin-bottom:20px; color:#fff; }
    footer { background:#1e293b; color:#cbd5e1; text-align:center; padding:20px; margin-top:30px; }
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

Check your site:

* `http://kkdevopsb4.shop`
* `http://www.kkdevopsb4.shop`



## 8️⃣ Enable HTTPS (Optional)

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache -d kkdevopsb4.shop -d www.kkdevopsb4.shop
```

✅ Free SSL via **Let’s Encrypt** with auto-renewal.




## 9️⃣ Architecture Overview

```
User Browser
     │
     ▼
Hostinger Domain (kkdevopsb4.shop)
     │
(Custom Nameservers)
     ▼
AWS Route 53
     │
(A Records → Elastic IP)
     ▼
EC2 Instance (Apache)
     │
Website Content (HTML / App)
```




## 10️⃣ Quick Troubleshooting

* **DNS issues**: Confirm Hostinger nameservers match Route 53, disable DNSSEC, allow propagation (\~30 mins).
* **Apache not loading**: Check port 80 is open in the Security Group and Apache is running.
* **SSL errors**: Ensure domain resolves correctly before running Certbot.
