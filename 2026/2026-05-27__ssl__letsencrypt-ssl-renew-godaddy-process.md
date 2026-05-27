# Guide: Renewing Let's Encrypt SSL via PunchSalad & GoDaddy

This document outlines the step-by-step process to manually renew a Let's Encrypt SSL certificate using PunchSalad and apply it to a GoDaddy cPanel hosting account.

---

# 🌐 PunchSalad Website

PunchSalad SSL Certificate Generator:

https://www.punchsalad.com/ssl-certificate-generator/

---

# 📋 Prerequisites

Before starting, ensure you have:

- Access to the GoDaddy account (cPanel and/or DNS Management).
- Access to the domain’s File Manager or hosting files (if using HTTP verification).
- Access to the domain’s DNS settings (if using DNS verification).
- A text editor to safely store the generated certificate files temporarily.

---

# 🛠️ Step 1: Generate the SSL Certificate Using PunchSalad

1. Open the PunchSalad SSL Certificate Generator.

2. Enter your domain name:
   - `example.com`
   - `www.example.com`

3. Enter your email address.
   - This is used for renewal reminders and certificate notifications.

4. Select a verification method.

---

## Option A — HTTP Verification

Use this method if you can upload files to your hosting account.

### Steps

1. PunchSalad will generate a verification file.
2. Download the verification file.
3. Upload it to the following directory on your hosting:

```text
public_html/.well-known/acme-challenge/
```

4. Ensure the file is publicly accessible in the browser.

Example:

```text
http://example.com/.well-known/acme-challenge/filename
```

5. Continue the verification process in PunchSalad.

---

## Option B — DNS Verification

Use this method if you manage DNS through GoDaddy.

### Steps

1. PunchSalad will provide a TXT record.
2. Log in to GoDaddy.
3. Open DNS Management for the domain.
4. Add the provided TXT record exactly as shown.
5. Save the DNS changes.
6. Wait a few minutes for DNS propagation.
7. Continue verification in PunchSalad.

---

# 🔐 Step 2: Save the Generated SSL Files

Once verification is successful, PunchSalad will generate the SSL files.

Save the following:

---

## Required Files

### 1. Certificate (CRT)

```text
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

### 2. Private Key (KEY)

```text
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```

### 3. CA Bundle (Optional)

If provided separately, also save:

```text
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

---

# 🖥️ Step 3: Install the SSL Certificate in GoDaddy cPanel

1. Log in to GoDaddy.
2. Open your hosting product.
3. Click **Manage** to access cPanel.
4. Scroll to the **Security** section.
5. Click **SSL/TLS**.
6. Open:

```text
Install and Manage SSL for your site (HTTPS)
```

7. Click **Manage SSL Sites**.

---

# 🧩 Step 4: Configure the SSL Certificate

## Select the Domain

Choose your domain from the dropdown list.

---

## Paste the Certificate (CRT)

Paste the full certificate text into:

```text
Certificate: (CRT)
```

---

## Paste the Private Key (KEY)

Paste the private key into:

```text
Private Key (KEY)
```

---

## Paste the CA Bundle (CABUNDLE)

If PunchSalad provided a CA Bundle:

- Paste it into:

```text
Certificate Authority Bundle (CABUNDLE)
```

If not:

- Click **Autofill by Domain** to let GoDaddy fetch it automatically.

---

# ✅ Step 5: Install the Certificate

1. Click:

```text
Install Certificate
```

2. Wait for the success confirmation message.

3. Test the website:

```text
https://example.com
```

4. Verify the SSL installation using:
   - Browser padlock icon
   - SSL checker tools

---

# 🔁 Optional: Force HTTPS via .htaccess

To redirect all HTTP traffic to HTTPS:

Edit or create the `.htaccess` file inside `public_html`.

Add:

```apache
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

Save the file.

---

# 🧹 Cleanup

After successful installation:

- Remove temporary HTTP verification files.
- Remove temporary DNS TXT records if no longer needed.

---

# ⚠️ Important Reminders

## ⏳ Certificate Expiry

Let’s Encrypt certificates expire every 90 days.

Recommended practice:

- Renew at least 7 days before expiration.
- Set calendar reminders.

---

# 📌 Recommended Renewal Schedule

| Day | Action |
|------|--------|
| Day 1 | Certificate Issued |
| Day 75 | Start Renewal |
| Day 83 | Final Renewal Reminder |
| Day 90 | Certificate Expires |

---

# 🤖 Optional Automation Alternatives

To avoid manual renewals, consider:

- Certbot with cron jobs
- acme.sh
- Cloudflare DNS API automation
- Hosting providers with AutoSSL support

---

# 🛡️ Best Practices

- Keep private keys secure.
- Never share the KEY publicly.
- Always keep backups of:
  - CRT
  - KEY
  - CABUNDLE
- Test SSL after every renewal.
- Use strong TLS settings when possible.

---

# 📞 Troubleshooting

## SSL Shows as Invalid

Possible causes:
- Incorrect CA Bundle
- Expired certificate
- Domain mismatch
- DNS propagation delay

---

## HTTPS Still Not Working

Check:
- `.htaccess` rules
- Browser cache
- CDN/Cloudflare SSL settings
- Mixed content warnings

---

# ✅ Final Verification Checklist

- [ ] SSL certificate generated
- [ ] Domain verification completed
- [ ] CRT pasted correctly
- [ ] KEY pasted correctly
- [ ] CABUNDLE configured
- [ ] HTTPS working
- [ ] HTTP redirected to HTTPS
- [ ] Cleanup completed
- [ ] Renewal reminder scheduled
