# 🚀 Ansible Reference Guide

> یک مرجع کامل و قابل استفاده برای Ansible - همه چیز در یک جا!

---

## 📚 فهرست مطالب

این repository شامل راهنماهای کامل برای تمام مباحث مهم Ansible است:

### 🔧 پایه و تنظیمات

| موضوع | توضیح |
|-------|-------|
| [📋 Inventories](./inventories/) | مدیریت و تعریف سرورهای هدف |
| [⚙️ Configuration](./configuration/) | تنظیمات ansible.cfg |
| [📜 Playbooks](./playbooks/) | نوشتن و اجرای Playbook‌ها |

### 📦 ماژول‌ها و Facts

| موضوع | توضیح |
|-------|-------|
| [📦 Modules](./modules/) | ماژول‌های Ad-Hoc (setup, file, copy, command, fetch) |
| [📊 Facts](./facts/) | جمع‌آوری و استفاده از اطلاعات سیستم |

### 🧩 Template و Variables

| موضوع | توضیح |
|-------|-------|
| [📦 Variables](./variables/) | group_vars, host_vars, hostvars و اولویت متغیرها |
| [🧩 Jinja2](./jinja2/) | Template engine و فیلترها |
| [🔮 Magic Variables](./magic-variables/) | متغیرهای خاص Ansible |
| [📝 Register](./register/) | ذخیره خروجی task‌ها |

### 🔀 کنترل جریان

| موضوع | توضیح |
|-------|-------|
| [❓ When](./when/) | اجرای شرطی task‌ها |
| [🔄 Looping](./looping/) | حلقه‌ها و تکرار |
| [📦 Blocks](./blocks/) | گروه‌بندی و مدیریت خطا |

### 🎯 ویژگی‌های پیشرفته

| موضوع | توضیح |
|-------|-------|
| [🎯 Delegation](./delegation/) | اجرای task روی سرور دیگر |
| [🏷️ Tags](./tags/) | اجرای انتخابی task‌ها |
| [🔐 Vault](./vault/) | رمزنگاری اطلاعات حساس |

### 🎭 سازماندهی و گسترش

| موضوع | توضیح |
|-------|-------|
| [🎭 Roles](./roles/) | سازماندهی و بازاستفاده از کد |
| [🔌 Plugins](./plugins/) | Lookup، Filter، Callback و... |

### 🔑 SSH Setup

| موضوع | توضیح |
|-------|-------|
| [🔑 SSH Key](./sshkey/) | تنظیم SSH Key برای اتصال |

### 🌐 مثال‌های دنیای واقعی

| موضوع | توضیح |
|-------|-------|
| [📚 Examples](./examples/) | نمونه‌های کامل و آماده استفاده |
| [✅ Best Practices](./best-practices/) | بهترین روش‌ها و استانداردها |

#### فهرست مثال‌ها:
| مثال | توضیح |
|------|-------|
| [LEMP Stack](./examples/lemp-stack/) | Linux + Nginx + MySQL + PHP |
| [Docker Host](./examples/docker-host/) | نصب Docker و دیپلوی با Compose |
| [Kubernetes](./examples/kubernetes/) | راه‌اندازی کلاستر K8s |
| [Monitoring](./examples/monitoring/) | Prometheus + Grafana + Alertmanager |
| [Web App Deploy](./examples/webapp-deploy/) | دیپلوی Rolling بدون Downtime |
| [Backup](./examples/backup/) | بکاپ خودکار با رمزنگاری |
| [Security](./examples/security/) | Hardening سرور لینوکس |
| [Users](./examples/users/) | مدیریت کاربران و SSH Keys |
| [SSL Certificates](./examples/ssl-certificates/) | Let's Encrypt با Certbot |
| [Load Balancer](./examples/load-balancer/) | HAProxy با SSL |

---

## 🚀 شروع سریع

### نصب Ansible:

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# CentOS/RHEL
sudo yum install ansible

# با pip
pip install ansible
```

### اولین دستور:

```bash
# تست اتصال
ansible all -m ping -i inventory.yml

# اجرای playbook
ansible-playbook -i inventory.yml playbook.yml
```

---

## 📂 ساختار پروژه پیشنهادی

```
project/
├── ansible.cfg              # تنظیمات
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── staging/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── mysql/
├── group_vars/
│   └── all/
│       ├── vars.yml
│       └── vault.yml
├── templates/
├── files/
└── filter_plugins/
```

---

## 🎯 Quick Reference

### دستورات پرکاربرد:

```bash
# لیست hosts
ansible all --list-hosts -i inventory.yml

# Dry run
ansible-playbook playbook.yml --check --diff

# با tag خاص
ansible-playbook playbook.yml --tags "deploy"

# محدود به سرور خاص
ansible-playbook playbook.yml --limit "web1"

# با رمز vault
ansible-playbook playbook.yml --ask-vault-pass

# Verbose
ansible-playbook playbook.yml -vvv
```

### ماژول‌های پرکاربرد:

```bash
# Ping
ansible all -m ping

# اجرای command
ansible all -a "hostname"

# نصب پکیج
ansible all -m apt -a "name=nginx state=present" -b

# کپی فایل
ansible all -m copy -a "src=file dest=/tmp/file"

# اطلاعات سیستم
ansible all -m setup -a "filter=ansible_distribution"
```

---

## 📚 منابع مفید

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Ansible GitHub](https://github.com/ansible/ansible)
- [Ansible Blog](https://www.ansible.com/blog)

---

## 🤝 مشارکت

اگر پیشنهاد یا اصلاحی دارید، خوشحال می‌شوم که بشنوم!

---

**نویسنده**: مرجع شخصی Ansible  
**آخرین بروزرسانی**: 2026