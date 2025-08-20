# 🌐 LAB: Hostinger Domain → AWS Route 53 → EC2 Hosting (Apache Web Server)

This lab demonstrates how to host a website using a **domain from Hostinger**, **AWS Route 53 DNS**, and an **EC2 instance running Apache**.

---

## 1️⃣ Domain Purchase & EC2 Setup

1. **Buy a domain** via Hostinger  
   Example: `kkdevopsb4.shop`

2. **Launch an EC2 instance** on AWS:  
   - OS: Ubuntu 22.04 LTS (free-tier eligible)  
   - Allocate an **Elastic IP** for a static public IP  

3. **Security Group Configuration**:  
   - Open port **80 (HTTP)** for all  
   - Open port **443 (HTTPS)** if you plan SSL  
   - Restrict port **22 (SSH)** to your own IP  

4. **SSH into your instance**:  
   ```bash
   ssh -i my-key.pem ubuntu@<Elastic-IP>
````

---

## 2️⃣ Install Apache Web Server

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

✅ Open `http://<Elastic-IP>` in a browser — Apache default page should appear.

---

## 3️⃣ Create Hosted Zone in Route 53

1. Navigate to **AWS Console → Route 53 → Hosted zones → Create Hosted Zone**
2. Domain: `kkdevopsb4.shop`
3. Type: **Public hosted zone**
4. Copy the **4 NS (nameserver) records** provided

---

## 4️⃣ Update Nameservers in Hostinger

1. In Hostinger: **Domains → kkdevopsb4.shop → DNS / Nameservers**
2. Select **Use custom nameservers** and paste the 4 NS records from Route 53
3. Make sure **DNSSEC is turned off** (otherwise DNS may fail to propagate)

---

## 5️⃣ Add A Records in Route 53

* **Root domain** (`kkdevopsb4.shop`)

  * Type: `A`
  * Value: `<Elastic-IP>`

* **www subdomain** (`www.kkdevopsb4.shop`)

  * Type: `A`
  * Value: `<Elastic-IP>`

TTL can remain **300**.

---

## 6️⃣ Verify DNS Setup

Run these commands to confirm DNS is pointing correctly:

```bash
nslookup kkdevopsb4.shop 8.8.8.8
nslookup www.kkdevopsb4.shop 8.8.8.8
```

✅ Both should resolve to your **Elastic IP**.

---

## 7️⃣ Deploy Your Website

1. Remove Apache default page:

```bash
sudo rm /var/www/html/index.html
```

2. Create your website page:

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
    body { margin:0; font-family:Arial,sans-serif; background:#0f172a; color:#e2e8f0; }
    header { display:flex; justify-content:space-between; align-items:center; background:#1e293b; padding:15px 40px; }
    header h1 { color:#38bdf8; margin:0; }
    nav a { margin:0 15px; color:#e2e8f0; text-decoration:none; }
    nav a:hover { color:#38bdf8; }
    .hero { text-align:center; padding:80px 20px; background:linear-gradient(145deg,#0f172a,#1e293b); }
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

✅ Check in browser:

* `http://kkdevopsb4.shop`
* `http://www.kkdevopsb4.shop`

---

## 8️⃣ Enable HTTPS (Optional but Recommended)

```bash
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache -d kkdevopsb4.shop -d www.kkdevopsb4.shop
```

✅ Let’s Encrypt will automatically install and renew the SSL certificate.

---

## 9️⃣ Architecture Overview

```
User Browser
     │
     ▼
Domain: kkdevopsb4.shop (Hostinger)
     │
(Custom Nameservers)
     ▼
AWS Route 53 (Hosted Zone)
     │
(A Records → Elastic IP)
     ▼
AWS EC2 Instance (Apache Web Server)
     │
Website Files (HTML / CSS / JS)
```

---

## 10️⃣ Troubleshooting

* **DNS not resolving:** Confirm nameservers are correct in Hostinger and DNSSEC is off. Wait 5–30 min.
* **Apache not showing site:** Make sure port 80 is open in Security Group and Apache is running:

  ```bash
  sudo systemctl status apache2
  ```
* **SSL fails:** Ensure domain resolves correctly before running Certbot.

```


