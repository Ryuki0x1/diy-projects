🚀 **My Journey Setting Up a Secure Personal Cloud with Nextcloud + Tailscale on MX Linux** 🌥️

---

🧠 **Objective:**
Create a private cloud server using Nextcloud, securely accessible over the internet **without port forwarding**, and accessible to a friend — but with restricted folder access.

---

## 🔧 Step-by-Step Setup Process

### 1️⃣ **Installed Apache, PHP, MySQL, and Nextcloud**

Commands used:

```bash
sudo apt install apache2 mariadb-server libapache2-mod-php php php-mysql
```

* ✅ Apache is the web server
* ✅ MariaDB (a drop-in replacement for MySQL)
* ✅ PHP and modules to run Nextcloud

Then, Nextcloud was extracted to:

```bash
/var/www/html/nextcloud
```

Permissions were fixed:

```bash
sudo chown -R www-data:www-data /var/www/html/nextcloud
```

---

### 2️⃣ **Set up Nextcloud database**

```bash
sudo mysql -u root -p
```

```sql
CREATE DATABASE nextcloud;
CREATE USER 'ncuser'@'localhost' IDENTIFIED BY 'your-password-here';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'ncuser'@'localhost';
FLUSH PRIVILEGES;
```

📄 Then edited `config.php` in `/var/www/html/nextcloud/config/`:

* Added `trusted_domains`
* Set `overwrite.cli.url` to match Tailscale URL

---

### 3️⃣ **Installed & Configured Tailscale**

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

🎉 Tailscale provided a secure HTTPS domain:

```bash
https://yourserver.tailXXXX.ts.net
```

Then exposed the Nextcloud server:

```bash
sudo tailscale serve --bg --https=443 --set-path / http://localhost:8080
```

> ✅ This runs in the background, allowing HTTPS access over Tailscale

---

### 🧪 **Debugging Issues & Their Solutions**

#### 🧨 `tailscale serve status` shows "No serve config"

**Problem:** Running `tailscale serve` without `--bg` only runs temporarily

**Fix:**

```bash
sudo tailscale serve --bg --https=443 --set-path / http://localhost:8080
```

✅ Now it persists in background until reboot.

---

#### 🧨 HTTP Error 502

**Problem:** Apache or Nextcloud server might not be reachable internally

**Debugging steps:**

* ✅ Checked Apache:

```bash
sudo systemctl status apache2
```

* ✅ Verified it's listening:

```bash
sudo ss -tuln | grep ':80'
```

* ✅ Tested with:

```bash
curl -I http://localhost
```

**Fix:** Ensure Apache and Nextcloud are running properly. Check `index.php` loads.

---

### 4️⃣ **Making Nextcloud Work for Friend**

#### ✅ Created a new user in Nextcloud

* Login as admin
* Go to **Users**
* Add new user (e.g., `frienduser`)

#### ✅ Gave folder access:

* Created folder via admin
* Shared to `frienduser`
* Used share permissions to **restrict access**

> 🔒 Friend can only access their own folder!

#### ✅ Friend joins Tailnet

* Installed Tailscale app
* Joined same Tailnet
* Opened `https://yourserver.tailXXXX.ts.net` in browser

---

### 5️⃣ **Systemd & Boot Config**

#### ✅ Make services run at boot:

```bash
sudo systemctl enable apache2
sudo systemctl enable tailscaled
```

#### ✅ Check default boot OS (for dual-booters):

```bash
sudo grep "^menuentry" /boot/grub/grub.cfg
```

Make sure MX Linux is selected as default in GRUB.

---

## 🧠 Local Data Storage (for recovery)

Your data is stored here:

```bash
/var/www/html/nextcloud/data/
```

Back this up to another drive or cloud location if needed.

---

## 🔍 Tips and Common Mistakes

| Issue                        | Reason                 | Fix                                                          |
| ---------------------------- | ---------------------- | ------------------------------------------------------------ |
| ❌ Tailscale link not opening | `serve` not persistent | Use `--bg` flag                                              |
| ❌ Friend can’t access        | Not in Tailnet         | Invite friend or let them log in with shared Tailnet account |
| ❌ 502 error                  | Apache/Nextcloud down  | Restart services, verify config                              |
| ❌ Accessing full server      | Friend not isolated    | Use **Nextcloud user permissions**                           |

---

## ✅ Final Checklist

* [x] Apache running
* [x] Nextcloud reachable on `localhost`
* [x] Tailscale HTTPS tunnel working
* [x] Friend has limited access
* [x] Data stored locally & backed up
* [x] Auto-start enabled on boot

---

🌟 **Now you have a secure, portable, and easy-to-manage cloud!**
Accessible from anywhere via Tailscale, with user-level access control — and no need to open ports. 🔐💾☁️

---

📦 **Optional Future Ideas:**

* Enable 2FA for Nextcloud
* Setup automatic backups of `data/`
* Add more friends with isolated folders
* Use a domain name with Tailscale Funnel (if public sharing is needed)

