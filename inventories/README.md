# 📋 Ansible Inventories

> مدیریت و تعریف سرورهای هدف در Ansible

---

## 🎯 Inventory چیست؟

Inventory فایلی است که لیست سرورهای هدف (managed nodes) را مشخص می‌کند. Ansible از این فایل برای دانستن اینکه روی چه سرورهایی باید کار کند استفاده می‌کند.

---

## 📁 انواع Inventory

### 1️⃣ Static Inventory (فایل ثابت)

#### فرمت INI (ساده‌ترین):

```ini
# /etc/ansible/hosts یا inventory.ini

# سرورهای تکی
192.168.1.10
server1.example.com

# گروه‌بندی سرورها
[webservers]
web1.example.com
web2.example.com
192.168.1.20

[dbservers]
db1.example.com
db2.example.com

# گروه تو در تو (children)
[production:children]
webservers
dbservers

# متغیرهای گروه
[webservers:vars]
http_port=80
ansible_user=deploy
```

#### فرمت YAML (خواناتر):

```yaml
# inventory.yml
all:
  hosts:
    server1.example.com:
    192.168.1.10:
  
  children:
    webservers:
      hosts:
        web1.example.com:
          http_port: 80
        web2.example.com:
          http_port: 8080
      vars:
        ansible_user: deploy
    
    dbservers:
      hosts:
        db1.example.com:
          mysql_port: 3306
        db2.example.com:
    
    production:
      children:
        webservers:
        dbservers:
```

### 2️⃣ Dynamic Inventory (پویا)

برای محیط‌های Cloud که سرورها دائماً تغییر می‌کنند:

```bash
# اسکریپت Python یا هر زبان دیگر
./dynamic_inventory.py --list

# استفاده از پلاگین‌های آماده
ansible-inventory -i aws_ec2.yml --list
```

---

## 🔧 پارامترهای اتصال (Connection Variables)

```yaml
[webservers]
web1 ansible_host=192.168.1.10 ansible_port=22 ansible_user=admin
web2 ansible_host=192.168.1.11 ansible_ssh_private_key_file=~/.ssh/web_key

[windows]
win1 ansible_host=192.168.1.50 ansible_connection=winrm ansible_user=Administrator
```

### جدول پارامترهای مهم:

| پارامتر | توضیح | مثال |
|---------|-------|------|
| `ansible_host` | آدرس IP یا hostname واقعی | `192.168.1.10` |
| `ansible_port` | پورت SSH | `22` |
| `ansible_user` | نام کاربری برای اتصال | `root` |
| `ansible_password` | رمز عبور (توصیه نمی‌شود) | `secret` |
| `ansible_ssh_private_key_file` | مسیر کلید SSH | `~/.ssh/id_rsa` |
| `ansible_connection` | نوع اتصال | `ssh`, `winrm`, `local` |
| `ansible_become` | استفاده از sudo | `yes` |
| `ansible_become_user` | کاربر sudo | `root` |
| `ansible_python_interpreter` | مسیر Python | `/usr/bin/python3` |

---

## 📂 ساختار Inventory پیشرفته

```
inventory/
├── production/
│   ├── hosts.yml           # لیست سرورها
│   ├── group_vars/
│   │   ├── all.yml         # متغیرهای همه
│   │   ├── webservers.yml  # متغیرهای وب‌سرورها
│   │   └── dbservers.yml   # متغیرهای دیتابیس
│   └── host_vars/
│       ├── web1.yml        # متغیرهای سرور خاص
│       └── db1.yml
├── staging/
│   ├── hosts.yml
│   ├── group_vars/
│   └── host_vars/
└── development/
    └── hosts.yml
```

### group_vars/webservers.yml:

```yaml
---
http_port: 80
max_clients: 200
ansible_user: deploy
```

### host_vars/web1.yml:

```yaml
---
http_port: 8080  # override برای این سرور خاص
ssl_enabled: true
```

---

## 🎯 الگوهای انتخاب (Patterns)

```bash
# همه سرورها
ansible all -m ping

# یک گروه
ansible webservers -m ping

# چند گروه
ansible webservers:dbservers -m ping

# اشتراک دو گروه (AND)
ansible 'webservers:&production' -m ping

# استثنا (NOT)
ansible 'webservers:!web1' -m ping

# با Regex
ansible '~web[0-9]+' -m ping

# یک سرور خاص
ansible web1.example.com -m ping

# محدود کردن با limit
ansible all -m ping --limit webservers
ansible all -m ping --limit 'web1,web2'
```

---

## 🔍 دستورات مفید

```bash
# نمایش لیست سرورها
ansible-inventory --list -y

# نمایش گراف inventory
ansible-inventory --graph

# نمایش متغیرهای یک سرور
ansible-inventory --host web1

# تست اتصال به همه
ansible all -m ping

# تست با inventory خاص
ansible -i inventory/production/hosts.yml all -m ping

# نمایش سرورهای یک گروه
ansible webservers --list-hosts
```

---

## 📝 مثال کامل

### inventory.yml:

```yaml
---
all:
  vars:
    ansible_user: ansible
    ansible_ssh_private_key_file: ~/.ssh/ansible_key
  
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
      vars:
        http_port: 80
        document_root: /var/www/html
    
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
          mysql_port: 3306
        db2:
          ansible_host: 192.168.1.21
      vars:
        mysql_max_connections: 500
    
    loadbalancers:
      hosts:
        lb1:
          ansible_host: 192.168.1.5
    
    # گروه‌بندی محیط‌ها
    production:
      children:
        webservers:
        dbservers:
        loadbalancers:
```

---

## ⚠️ نکات مهم

1. **اولویت متغیرها**: `host_vars` > `group_vars` > `inventory vars`
2. **امنیت**: رمزها را در inventory نگذارید، از `ansible-vault` استفاده کنید
3. **نام‌گذاری**: از نام‌های معنادار برای گروه‌ها استفاده کنید
4. **تست**: همیشه با `--list-hosts` قبل از اجرا چک کنید

---

## 📚 منابع

- [Ansible Inventory Docs](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
- [Dynamic Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html)
