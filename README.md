# 🎵 Sputify – Self-Hosted Koel Music Streaming Server

Welcome to **Sputify**, a private, self-hosted music streaming service powered by [Koel](https://koel.dev). This guide walks through setting it up from scratch on a VPS with a custom domain, HTTPS, and multi-user support.

---

![Benjamin Bannekat](https://raw.githubusercontent.com/rubentotterman/sputify/refs/heads/main/theme-dawn.t2QwyJDN.webp)

## 📦 Tech Stack

- **Koel** – Laravel-based music streaming server  
- **Ubuntu 22.04** – VPS OS (DigitalOcean droplet)  
- **SQLite** – Lightweight music metadata storage  
- **Nginx** – Web server  
- **PHP 8.3 FPM** – PHP runtime  
- **Node.js + Vite** – Frontend asset builder  
- **Let's Encrypt (Certbot)** – Free HTTPS  
- **Cloudflare** – DNS + Domain (`sputify.org`)  
- **FileZilla** – Easy drag-and-drop audio uploads  

---

## 🚀 Installation Steps

### 1. 🖥️ Provision VPS

Use DigitalOcean or any provider to launch a VPS (recommend 1GB+ RAM).

```bash
apt update && apt upgrade -y
apt install nginx php8.3 php8.3-fpm php8.3-sqlite3 \
    unzip curl git composer nodejs npm -y
```

---

### 2. 📁 Clone Koel and Set Up

```bash
cd /var/www
git clone https://github.com/koel/koel.git
cd koel
composer install
cp .env.example .env
php artisan key:generate
```

Set up SQLite in `.env`:

```env
DB_CONNECTION=sqlite
DB_DATABASE=/var/www/koel/database.sqlite
```

```bash
touch database.sqlite
php artisan koel:init
```

---

### 3. 🛠️ Fix Vite/Node Memory (512MB RAM workaround)

```bash
NODE_OPTIONS="--max-old-space-size=2048" npm install --legacy-peer-deps
npx vite build
```

---

## 🌐 Configure Nginx

### Disable Apache (if active):

```bash
systemctl stop apache2
systemctl disable apache2
```

### Create Koel site config:

```nginx
# /etc/nginx/sites-available/koel
server {
    listen 80;
    server_name sputify.org www.sputify.org;

    root /var/www/koel/public;
    index index.php index.html;

    access_log /var/log/nginx/koel_access.log;
    error_log /var/log/nginx/koel_error.log;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Enable and test:

```bash
ln -s /etc/nginx/sites-available/koel /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

---

## 🔒 Enable HTTPS with Certbot

1. Point your domain (`sputify.org`) to your VPS IP via Cloudflare DNS.
2. Temporarily disable proxy (gray cloud icon).
3. Run Certbot:

```bash
apt install certbot python3-certbot-nginx -y
certbot --nginx
```

Choose your domain and enable HTTP to HTTPS redirect.

---

## 🎵 Upload Music with FileZilla

1. **Install FileZilla Client** (not Server).
2. **Open FileZilla** and connect to your server:
   - **Host**: `sftp://your.server.ip`
   - **Username**: `root`
   - **Password**: (your VPS root password)
   - **Port**: `22`
3. **Navigate to server folder**: `/var/www/koel/music`
4. **Upload** `.mp3`, `.flac`, or `.wav` files via drag and drop.
5. SSH into your server and sync:

```bash
php artisan koel:sync
```

This will scan your media folder and update Koel’s library.

If the folder doesn't exist:

```bash
mkdir /var/www/koel/music
chown -R www-data:www-data /var/www/koel/music
```

Update your `.env` file:

```env
MEDIA_PATH=/var/www/koel/music
```

Then re-run:

```bash
php artisan koel:sync
```

---

## 👥 Add Users

1. Log in as admin.
2. Go to Settings → Users.
3. Add more users (like your partner) so you can both stream simultaneously.

---

## 🛡️ Background Process

No need to run `php artisan serve`. Koel runs behind Nginx and PHP-FPM.

To make sure it's running:

```bash
systemctl status nginx
systemctl status php8.3-fpm
```

---

## 🔧 Troubleshooting

- Check logs:
```bash
tail -f /var/log/nginx/error.log
tail -f /var/www/koel/storage/logs/laravel.log
```

- Restart services:
```bash
systemctl restart nginx
systemctl restart php8.3-fpm
```

---

## ✅ Final Result

Visit your Koel server:

```
https://your-domain
```

Log in, stream music, manage playlists, and invite others to enjoy your cloud music experience.

---

## 🙏 Credits

- [Koel](https://koel.dev)
- [Laravel](https://laravel.com)
- [DigitalOcean](https://digitalocean.com)
- [Let's Encrypt](https://letsencrypt.org)
- [Cloudflare](https://cloudflare.com)
- [FileZilla](https://filezilla-project.org)

---

Enjoy your music — privately, securely, and from anywhere 🎧
