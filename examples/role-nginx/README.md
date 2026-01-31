# 🌐 Nginx Role

یک Role کامل و حرفه‌ای برای نصب و کانفیگ Nginx

## ✨ امکانات

- ✅ نصب Nginx
- ✅ کانفیگ بهینه شده
- ✅ پشتیبانی از چندین سایت
- ✅ SSL/TLS با تنظیمات امن
- ✅ Gzip compression
- ✅ Security headers
- ✅ Reverse proxy support
- ✅ PHP-FPM integration
- ✅ پشتیبانی از Ubuntu/Debian و CentOS/RHEL

---

## 📂 ساختار Role

```
roles/nginx/
├── defaults/
│   └── main.yml          # متغیرهای پیش‌فرض
├── handlers/
│   └── main.yml          # handlers
├── meta/
│   └── main.yml          # متادیتا برای Galaxy
├── tasks/
│   ├── main.yml          # نقطه ورود
│   ├── install.yml       # نصب
│   ├── configure.yml     # کانفیگ اصلی
│   ├── sites.yml         # کانفیگ سایت‌ها
│   └── service.yml       # مدیریت سرویس
├── templates/
│   ├── nginx.conf.j2             # کانفیگ اصلی
│   ├── site.conf.j2              # کانفیگ سایت
│   ├── ssl-params.conf.j2        # تنظیمات SSL
│   ├── security-headers.conf.j2  # هدرهای امنیتی
│   └── gzip.conf.j2              # تنظیمات فشرده‌سازی
└── vars/
    ├── Debian.yml        # متغیرهای Debian/Ubuntu
    └── RedHat.yml        # متغیرهای RHEL/CentOS
```

---

## 🚀 استفاده

### استفاده ساده

```yaml
- hosts: webservers
  roles:
    - nginx
```

### با تنظیمات سایت

```yaml
- hosts: webservers
  vars:
    nginx_sites:
      - name: example.com
        server_name: example.com www.example.com
        root: /var/www/example.com
        php: true
        ssl: true
        ssl_cert: /etc/letsencrypt/live/example.com/fullchain.pem
        ssl_key: /etc/letsencrypt/live/example.com/privkey.pem
  roles:
    - nginx
```

### Reverse Proxy

```yaml
nginx_sites:
  - name: api.example.com
    server_name: api.example.com
    proxy_pass: http://localhost:3000
    ssl: true
    ssl_cert: /etc/letsencrypt/live/api.example.com/fullchain.pem
    ssl_key: /etc/letsencrypt/live/api.example.com/privkey.pem
```

---

## ⚙️ متغیرها

### متغیرهای اصلی

| متغیر | پیش‌فرض | توضیح |
|-------|---------|-------|
| `nginx_worker_processes` | `auto` | تعداد worker processes |
| `nginx_worker_connections` | `1024` | حداکثر connection در هر worker |
| `nginx_keepalive_timeout` | `65` | زمان keepalive |
| `nginx_client_max_body_size` | `64m` | حداکثر سایز آپلود |
| `nginx_server_tokens` | `off` | نمایش نسخه Nginx |

### متغیرهای SSL

| متغیر | پیش‌فرض | توضیح |
|-------|---------|-------|
| `nginx_ssl_protocols` | `TLSv1.2 TLSv1.3` | پروتکل‌های مجاز |
| `nginx_ssl_prefer_server_ciphers` | `on` | اولویت cipher سرور |
| `nginx_ssl_stapling` | `on` | OCSP stapling |
| `nginx_generate_dhparam` | `no` | ساخت DH params |

### متغیرهای سایت

هر سایت در `nginx_sites` می‌تواند این آپشن‌ها را داشته باشد:

| آپشن | اجباری | توضیح |
|------|--------|-------|
| `name` | ✅ | نام سایت |
| `server_name` | ✅ | دامنه‌ها |
| `root` | ❌ | مسیر document root |
| `index` | ❌ | فایل‌های index |
| `ssl` | ❌ | فعال‌سازی SSL |
| `ssl_cert` | ❌ | مسیر certificate |
| `ssl_key` | ❌ | مسیر private key |
| `php` | ❌ | فعال‌سازی PHP |
| `proxy_pass` | ❌ | آدرس backend |
| `enabled` | ❌ | فعال/غیرفعال (پیش‌فرض: true) |
| `extra_config` | ❌ | کانفیگ اضافی |

---

## 🏷️ Tags

```bash
# فقط نصب
ansible-playbook site.yml --tags nginx-install

# فقط کانفیگ
ansible-playbook site.yml --tags nginx-config

# فقط سایت‌ها
ansible-playbook site.yml --tags nginx-sites
```

---

## 📋 مثال کامل

```yaml
nginx_worker_processes: auto
nginx_worker_connections: 4096
nginx_client_max_body_size: 100m
nginx_generate_dhparam: yes
nginx_remove_default: yes

nginx_sites:
  # سایت PHP
  - name: blog.example.com
    server_name: blog.example.com
    root: /var/www/blog
    index: index.php index.html
    php: true
    ssl: true
    ssl_cert: /etc/letsencrypt/live/blog.example.com/fullchain.pem
    ssl_key: /etc/letsencrypt/live/blog.example.com/privkey.pem
  
  # API (Reverse Proxy)
  - name: api.example.com
    server_name: api.example.com
    proxy_pass: http://127.0.0.1:3000
    proxy_read_timeout: 120s
    ssl: true
    ssl_cert: /etc/letsencrypt/live/api.example.com/fullchain.pem
    ssl_key: /etc/letsencrypt/live/api.example.com/privkey.pem
  
  # Static site
  - name: static.example.com
    server_name: static.example.com
    root: /var/www/static
    static_expires: 1y
    ssl: false
```
