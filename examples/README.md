# 🌐 Real-World Ansible Examples

> نمونه‌های واقعی، کامل و آماده استفاده در محیط Production

---

## 📂 فهرست مثال‌ها

### 🌐 وب و اپلیکیشن
| پوشه | توضیح | سطح |
|------|-------|-----|
| [lemp-stack](./lemp-stack/) | نصب کامل Linux, Nginx, MySQL, PHP | ⭐⭐ متوسط |
| [webapp-deploy](./webapp-deploy/) | دیپلوی Rolling با Zero Downtime | ⭐⭐⭐ پیشرفته |
| [load-balancer](./load-balancer/) | HAProxy Load Balancer با SSL | ⭐⭐⭐ پیشرفته |
| [ssl-certificates](./ssl-certificates/) | Let's Encrypt با Certbot + تنظیم Nginx | ⭐⭐ متوسط |

### 🐳 کانتینر و ارکستریشن
| پوشه | توضیح | سطح |
|------|-------|-----|
| [docker-host](./docker-host/) | راه‌اندازی Docker و Docker Compose + دیپلوی | ⭐⭐ متوسط |
| [kubernetes](./kubernetes/) | راه‌اندازی کامل کلاستر K8s با kubeadm | ⭐⭐⭐ پیشرفته |

### 🗄️ دیتابیس‌ها
| پوشه | توضیح | سطح |
|------|-------|-----|
| [postgresql](./postgresql/) | PostgreSQL با Streaming Replication | ⭐⭐⭐ پیشرفته |
| [mssql-server](./mssql-server/) | Microsoft SQL Server 2022 روی Linux | ⭐⭐ متوسط |
| [oracle-database](./oracle-database/) | Oracle Database 19c Silent Installation | ⭐⭐⭐ پیشرفته |
| [mongodb-replicaset](./mongodb-replicaset/) | MongoDB Replica Set با 3 گره | ⭐⭐⭐ پیشرفته |
| [redis-cluster](./redis-cluster/) | Redis Cluster با 3 Master و 3 Replica | ⭐⭐⭐ پیشرفته |

### 📊 مانیتورینگ و لاگ
| پوشه | توضیح | سطح |
|------|-------|-----|
| [monitoring](./monitoring/) | Prometheus + Grafana + Alertmanager + Node Exporter | ⭐⭐⭐ پیشرفته |
| [elk-stack](./elk-stack/) | Elasticsearch + Logstash + Kibana Cluster | ⭐⭐⭐ پیشرفته |
| [loki-stack](./loki-stack/) | Grafana + Loki + Promtail (جایگزین سبک ELK) | ⭐⭐⭐ پیشرفته |

### 📨 پیام‌رسانی
| پوشه | توضیح | سطح |
|------|-------|-----|
| [rabbitmq-cluster](./rabbitmq-cluster/) | RabbitMQ Cluster با HA Policies | ⭐⭐⭐ پیشرفته |

### 🛠️ DevOps Tools
| پوشه | توضیح | سطح |
|------|-------|-----|
| [gitlab-cicd](./gitlab-cicd/) | GitLab CE + Container Registry + Runners | ⭐⭐⭐ پیشرفته |
| [mattermost](./mattermost/) | Mattermost Team Chat با PostgreSQL | ⭐⭐ متوسط |

### 🔒 امنیت و مدیریت
| پوشه | توضیح | سطح |
|------|-------|-----|
| [security](./security/) | Hardening کامل سرور لینوکس (14 بخش) | ⭐⭐ متوسط |
| [users](./users/) | مدیریت کاربران، گروه‌ها و SSH Keys | ⭐ مبتدی |
| [backup](./backup/) | بکاپ خودکار دیتابیس و فایل‌ها با رمزنگاری | ⭐⭐ متوسط |

### 📦 نمونه Role
| پوشه | توضیح | سطح |
|------|-------|-----|
| [role-nginx](./role-nginx/) | نمونه Role کامل و حرفه‌ای برای Nginx | ⭐⭐⭐ پیشرفته |

---

## 📋 جزئیات هر مثال

### 1. 🌐 LEMP Stack
```
lemp-stack/
├── inventory.yml      # تعریف webservers و dbservers
├── site.yml           # Playbook اصلی
└── templates/
    ├── nginx.conf.j2          # کانفیگ اصلی Nginx
    ├── default-site.conf.j2   # کانفیگ سایت
    ├── php-fpm-pool.conf.j2   # تنظیمات PHP-FPM
    ├── my.cnf.j2              # کانفیگ MySQL
    └── db-test.php.j2         # صفحه تست
```

### 2. 🐳 Docker Host
```
docker-host/
├── inventory.yml              # سرورهای Docker
├── site.yml                   # نصب Docker
├── deploy-compose-app.yml     # دیپلوی با Compose
└── templates/
    ├── docker-compose.yml.j2  # قالب Compose
    └── env.j2                 # متغیرهای محیطی
```

### 3. ☸️ Kubernetes
```
kubernetes/
├── inventory.yml      # master و worker nodes
└── site.yml           # راه‌اندازی کامل کلاستر
                       # شامل: containerd, kubeadm, CNI, addons
```

### 4. 📊 Monitoring (Prometheus Stack)
```
monitoring/
├── inventory.yml
├── site.yml           # Prometheus + Grafana + Alertmanager
└── templates/
    ├── prometheus.yml.j2      # کانفیگ Prometheus
    ├── alert_rules.yml.j2     # قوانین Alert
    └── alertmanager.yml.j2    # کانفیگ Alertmanager
```

### 5. 📊 ELK Stack
```
elk-stack/
├── inventory.yml      # Elasticsearch + Logstash + Kibana
├── site.yml
└── templates/
    ├── elasticsearch.yml.j2     # کانفیگ Elasticsearch
    ├── kibana.yml.j2            # کانفیگ Kibana
    ├── logstash.yml.j2          # کانفیگ Logstash
    ├── jvm.options.j2           # تنظیمات JVM
    ├── pipeline-syslog.conf.j2  # پایپلاین Syslog
    ├── pipeline-nginx.conf.j2   # پایپلاین Nginx
    └── pipeline-beats.conf.j2   # پایپلاین Beats
```

### 6. 📊 Loki Stack (جایگزین سبک ELK)
```
loki-stack/
├── inventory.yml      # Loki + Grafana + Promtail
├── site.yml
└── templates/
    ├── loki-config.yml.j2      # کانفیگ Loki
    ├── promtail-config.yml.j2  # کانفیگ Promtail
    └── grafana.ini.j2          # کانفیگ Grafana
```

### 7. 🗄️ PostgreSQL (Primary-Replica)
```
postgresql/
├── inventory.yml      # Primary و Replica nodes
└── site.yml           # نصب + Streaming Replication
```

### 8. 🗄️ Microsoft SQL Server
```
mssql-server/
├── inventory.yml
└── site.yml           # نصب MSSQL 2022 روی Linux
```

### 9. 🗄️ Oracle Database
```
oracle-database/
├── inventory.yml
├── site.yml
└── templates/
    ├── db_install.rsp.j2   # Response File نصب
    ├── dbca.rsp.j2         # Response File ساخت DB
    ├── netca.rsp.j2        # Response File Listener
    └── oracle.service.j2   # Systemd Service
```

### 10. 🗄️ MongoDB Replica Set
```
mongodb-replicaset/
├── inventory.yml      # 3 گره MongoDB
├── site.yml
└── templates/
    └── mongod.conf.j2     # کانفیگ MongoDB
```

### 11. 🔴 Redis Cluster
```
redis-cluster/
├── inventory.yml      # 3 Master + 3 Replica
├── site.yml
└── templates/
    └── redis.conf.j2      # کانفیگ Redis
```

### 12. 🐰 RabbitMQ Cluster
```
rabbitmq-cluster/
├── inventory.yml      # 3 گره RabbitMQ
├── site.yml
└── templates/
    ├── rabbitmq.conf.j2       # کانفیگ RabbitMQ
    └── rabbitmq-env.conf.j2   # Environment
```

### 13. 🦊 GitLab + CI/CD
```
gitlab-cicd/
├── inventory.yml      # GitLab + Runners
├── site.yml
└── templates/
    └── gitlab.rb.j2       # کانفیگ GitLab
```

### 14. 💬 Mattermost
```
mattermost/
├── inventory.yml
├── site.yml
└── templates/
    ├── config.json.j2              # کانفیگ Mattermost
    └── nginx-mattermost.conf.j2    # Nginx Reverse Proxy
```

### 15. 🚀 Web App Deploy
```
webapp-deploy/
├── inventory.yml
└── site.yml           # Rolling deployment با rollback
```

### 16. 💾 Backup
```
backup/
├── inventory.yml
├── site.yml
└── templates/
    ├── mysql_backup.sh.j2     # بکاپ MySQL
    ├── postgres_backup.sh.j2  # بکاپ PostgreSQL
    └── file_backup.sh.j2      # بکاپ فایل‌ها
```

### 17. 🔒 Security Hardening
```
security/
├── inventory.yml
├── site.yml           # 14 بخش Hardening
└── templates/
    └── jail.local.j2  # کانفیگ Fail2ban
```

### 18. 👥 Users Management
```
users/
├── inventory.yml
└── site.yml           # مدیریت کاربران، گروه‌ها، SSH
```

### 19. 🔐 SSL Certificates
```
ssl-certificates/
├── inventory.yml
├── site.yml
└── templates/
    └── nginx-ssl-site.conf.j2
```

### 20. ⚖️ Load Balancer (HAProxy)
```
load-balancer/
├── inventory.yml
├── site.yml
└── templates/
    └── haproxy.cfg.j2
```

### 21. 📦 Role Structure (Nginx Example)
```
role-nginx/
├── defaults/main.yml     # مقادیر پیش‌فرض
├── handlers/main.yml     # Handlers
├── meta/main.yml         # متادیتای Role
├── tasks/main.yml        # Task‌های اصلی
├── templates/            # قالب‌ها
└── vars/main.yml         # متغیرهای ثابت
```

---

## 🚀 نحوه استفاده

### 1️⃣ کلون کردن Repository
```bash
git clone <repo-url>
cd ansible
```

### 2️⃣ ویرایش Inventory
```bash
cd examples/<example-name>
vim inventory.yml
# IP ها و متغیرها را تنظیم کنید
```

### 3️⃣ اجرای Playbook
```bash
# اجرای کامل
ansible-playbook -i inventory.yml site.yml

# Dry Run (بدون تغییر واقعی)
ansible-playbook -i inventory.yml site.yml --check

# با Tag خاص
ansible-playbook -i inventory.yml site.yml --tags install

# Verbose mode
ansible-playbook -i inventory.yml site.yml -vvv
```

---

## 📖 فرمت توضیحات Playbook

هر Playbook با یک بلوک توضیحات شروع می‌شود:

```yaml
# ═══════════════════════════════════════════════════════════════════════════════
# 📦 [نام Playbook]
# ═══════════════════════════════════════════════════════════════════════════════
#
# 🎯 هدف:
#    [توضیح هدف اصلی]
#
# ✅ کارهایی که انجام می‌شود:
#    1. [کار اول]
#    2. [کار دوم]
#    ...
#
# 📋 نحوه اجرا:
#    ansible-playbook -i inventory.yml site.yml
#
# 📊 Tags موجود:
#    - tag1: توضیح
#    - tag2: توضیح
#
# ═══════════════════════════════════════════════════════════════════════════════
```

---

## 🎯 Best Practices استفاده شده

- ✅ استفاده از `become` فقط در جایی که لازم است
- ✅ نام‌گذاری واضح برای همه Task ها
- ✅ استفاده از `tags` برای اجرای انتخابی
- ✅ `handlers` برای restart سرویس‌ها
- ✅ متغیرهای قابل تنظیم در `inventory.yml`
- ✅ استفاده از `templates` برای کانفیگ‌ها
- ✅ `wait_for` برای اطمینان از آماده بودن سرویس
- ✅ `block` و `rescue` برای Error Handling
- ✅ توضیحات فارسی برای درک بهتر
- ✅ پشتیبانی از چند سیستم‌عامل (Debian/RedHat)

---

## 🔧 سفارشی‌سازی

### تغییر متغیرها
همه متغیرهای قابل تنظیم در `inventory.yml` تعریف شده‌اند:

```yaml
all:
  vars:
    # تنظیمات عمومی
    variable_name: value
```

### اضافه کردن سرورها
```yaml
  hosts:
    new-server:
      ansible_host: 192.168.1.XXX
```

---

## 📚 منابع بیشتر

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
