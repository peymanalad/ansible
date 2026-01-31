# 📦 Ansible Variables

> همه چیز درباره متغیرها در Ansible

---

## 🎯 انواع متغیرها

Ansible چندین روش برای تعریف و استفاده از متغیرها دارد. درک اولویت آنها بسیار مهم است.

---

## 📂 group_vars

متغیرهایی که به یک گروه از سرورها اعمال می‌شوند.

### ساختار:

```
inventory/
├── hosts.yml
├── group_vars/
│   ├── all.yml              # برای همه سرورها
│   ├── all/                 # یا به صورت فولدر
│   │   ├── vars.yml
│   │   └── vault.yml        # متغیرهای رمزنگاری شده
│   ├── webservers.yml       # برای گروه webservers
│   ├── webservers/
│   │   ├── vars.yml
│   │   └── vault.yml
│   └── dbservers.yml        # برای گروه dbservers
```

### مثال group_vars/all.yml:

```yaml
---
# متغیرهای مشترک همه سرورها
ntp_server: time.google.com
dns_servers:
  - 8.8.8.8
  - 8.8.4.4

ansible_user: deploy
ansible_become: yes

timezone: Asia/Tehran
```

### مثال group_vars/webservers.yml:

```yaml
---
# متغیرهای مخصوص وب‌سرورها
http_port: 80
https_port: 443
document_root: /var/www/html

nginx_worker_processes: auto
nginx_worker_connections: 1024
```

### مثال group_vars/dbservers.yml:

```yaml
---
# متغیرهای مخصوص دیتابیس‌ها
mysql_port: 3306
mysql_bind_address: 0.0.0.0
mysql_max_connections: 500

mysql_databases:
  - name: app_production
    encoding: utf8mb4
  - name: app_staging
```

---

## 🖥️ host_vars

متغیرهایی که به یک سرور خاص اعمال می‌شوند.

### ساختار:

```
inventory/
├── hosts.yml
├── group_vars/
└── host_vars/
    ├── web1.example.com.yml     # برای web1
    ├── web1.example.com/        # یا به صورت فولدر
    │   ├── vars.yml
    │   └── vault.yml
    ├── web2.example.com.yml     # برای web2
    └── db1.example.com.yml      # برای db1
```

### مثال host_vars/web1.example.com.yml:

```yaml
---
# متغیرهای مخصوص این سرور
ansible_host: 192.168.1.10
ansible_port: 22

# override کردن متغیر گروه
http_port: 8080

# متغیرهای خاص این سرور
ssl_certificate: /etc/ssl/certs/web1.crt
ssl_key: /etc/ssl/private/web1.key
is_primary: true
```

---

## 🔄 hostvars (Magic Variable)

دسترسی به متغیرهای **همه سرورها** از هر جایی.

### استفاده:

```yaml
tasks:
  # متغیر سرور فعلی
  - debug:
      msg: "My IP: {{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] }}"
  
  # متغیر سرور دیگر
  - debug:
      msg: "DB IP: {{ hostvars['db1.example.com']['ansible_host'] }}"
  
  # لوپ روی گروه و گرفتن IP
  - debug:
      msg: "{{ item }}: {{ hostvars[item]['ansible_default_ipv4']['address'] }}"
    loop: "{{ groups['webservers'] }}"
```

### در Template:

```jinja2
# ایجاد فایل hosts
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }}  {{ host }}
{% endfor %}

# کانفیگ کلاستر
{% for host in groups['cluster'] %}
node.{{ loop.index }}={{ hostvars[host]['ansible_host'] }}:{{ hostvars[host]['cluster_port'] | default(7000) }}
{% endfor %}
```

### نکته مهم:

```yaml
# hostvars فقط بعد از gather_facts پر می‌شود!
- hosts: all
  gather_facts: yes  # باید yes باشد
  
  tasks:
    - debug:
        var: hostvars['web1']['ansible_memtotal_mb']
```

---

## 📋 روش‌های تعریف متغیر

### 1️⃣ در Inventory:

```yaml
# inventory.yml
all:
  hosts:
    web1:
      ansible_host: 192.168.1.10
      http_port: 80          # متغیر host
  children:
    webservers:
      hosts:
        web1:
      vars:
        nginx_user: www-data  # متغیر گروه
```

### 2️⃣ در Playbook (vars):

```yaml
- hosts: webservers
  vars:
    http_port: 80
    packages:
      - nginx
      - php
  tasks:
    - apt:
        name: "{{ packages }}"
```

### 3️⃣ در Playbook (vars_files):

```yaml
- hosts: webservers
  vars_files:
    - vars/common.yml
    - vars/{{ env }}.yml
    - vars/secrets.yml  # encrypted
  tasks:
    - debug:
        var: db_password
```

### 4️⃣ با vars_prompt:

```yaml
- hosts: webservers
  vars_prompt:
    - name: username
      prompt: "Enter username"
      private: no
    
    - name: password
      prompt: "Enter password"
      private: yes
      confirm: yes
  
  tasks:
    - user:
        name: "{{ username }}"
        password: "{{ password | password_hash('sha512') }}"
```

### 5️⃣ با include_vars:

```yaml
tasks:
  - name: Load OS-specific vars
    include_vars: "{{ ansible_os_family }}.yml"
  
  - name: Load from directory
    include_vars:
      dir: vars/
      extensions:
        - yml
        - yaml
```

### 6️⃣ با set_fact:

```yaml
tasks:
  - name: Set fact
    set_fact:
      my_var: "hello"
      calculated: "{{ ansible_memtotal_mb / 1024 }}"
  
  - name: Use it
    debug:
      msg: "{{ my_var }}"
```

### 7️⃣ با register:

```yaml
tasks:
  - name: Run command
    command: hostname
    register: hostname_result
  
  - name: Use result
    debug:
      msg: "Hostname is {{ hostname_result.stdout }}"
```

### 8️⃣ از Command Line (-e):

```bash
ansible-playbook site.yml -e "env=production"
ansible-playbook site.yml -e '{"users": ["ali", "reza"]}'
ansible-playbook site.yml -e "@vars/extra.yml"
```

---

## 📊 اولویت متغیرها (از پایین به بالا)

```
1.  command line values (for example, -u my_user)
2.  role defaults (roles/x/defaults/main.yml)
3.  inventory file or script group vars
4.  inventory group_vars/all
5.  playbook group_vars/all
6.  inventory group_vars/*
7.  playbook group_vars/*
8.  inventory file or script host vars
9.  inventory host_vars/*
10. playbook host_vars/*
11. host facts / cached set_facts
12. play vars
13. play vars_prompt
14. play vars_files
15. role vars (roles/x/vars/main.yml)
16. block vars (only for tasks in block)
17. task vars (only for the task)
18. include_vars
19. set_facts / registered vars
20. role parameters
21. include parameters
22. extra vars (-e) ← بالاترین اولویت!
```

### خلاصه مهم:

| اولویت | منبع |
|--------|------|
| **پایین‌ترین** | role defaults |
| ⬆️ | inventory vars |
| ⬆️ | group_vars |
| ⬆️ | host_vars |
| ⬆️ | play vars |
| ⬆️ | role vars |
| ⬆️ | task vars |
| ⬆️ | set_fact |
| **بالاترین** | extra vars (-e) |

---

## 📂 ساختار کامل پروژه

```
project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all/
│   │   │   │   ├── vars.yml       # متغیرهای عمومی
│   │   │   │   └── vault.yml      # رمزنگاری شده
│   │   │   ├── webservers.yml
│   │   │   └── dbservers.yml
│   │   └── host_vars/
│   │       ├── web1.yml
│   │       └── db1.yml
│   └── staging/
│       ├── hosts.yml
│       ├── group_vars/
│       └── host_vars/
├── playbooks/
│   └── site.yml
├── roles/
│   └── nginx/
│       ├── defaults/main.yml      # پیش‌فرض‌ها (اولویت پایین)
│       └── vars/main.yml          # ثابت‌ها (اولویت بالا)
└── vars/
    ├── common.yml
    └── secrets.yml
```

---

## 🔧 الگوهای استفاده

### الگوی جداسازی secrets:

```yaml
# group_vars/all/vars.yml (unencrypted)
db_host: db.example.com
db_port: 5432
db_name: myapp
db_user: myapp
db_password: "{{ vault_db_password }}"  # reference

# group_vars/all/vault.yml (encrypted)
vault_db_password: "SuperSecret123!"
```

### الگوی override بر اساس محیط:

```yaml
# group_vars/all.yml
env: development
debug: true
log_level: debug

# inventory/production/group_vars/all.yml
env: production
debug: false
log_level: warning
```

### الگوی OS-specific:

```yaml
# vars/Debian.yml
package_name: nginx
service_name: nginx
config_path: /etc/nginx

# vars/RedHat.yml
package_name: nginx
service_name: nginx
config_path: /etc/nginx

# در playbook
- include_vars: "vars/{{ ansible_os_family }}.yml"
```

---

## 🔍 Debug متغیرها

```yaml
tasks:
  # نمایش یک متغیر
  - debug:
      var: my_variable
  
  # نمایش با پیام
  - debug:
      msg: "Value is {{ my_variable }}"
  
  # نمایش همه متغیرهای host
  - debug:
      var: hostvars[inventory_hostname]
  
  # نمایش همه facts
  - debug:
      var: ansible_facts
```

```bash
# از command line
ansible web1 -m debug -a "var=hostvars[inventory_hostname]"
```

---

## 📝 مثال کامل

### inventory/production/hosts.yml:

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
```

### inventory/production/group_vars/all.yml:

```yaml
---
# مشترک همه
ansible_user: deploy
ansible_become: yes
ntp_server: time.google.com
env: production
```

### inventory/production/group_vars/webservers.yml:

```yaml
---
http_port: 80
document_root: /var/www/html
```

### inventory/production/host_vars/web1.yml:

```yaml
---
is_primary: true
ssl_enabled: true
```

### playbook.yml:

```yaml
---
- hosts: webservers
  vars:
    app_version: "2.0.0"
  
  tasks:
    - name: Show all relevant vars
      debug:
        msg: |
          Environment: {{ env }}
          HTTP Port: {{ http_port }}
          Is Primary: {{ is_primary | default(false) }}
          App Version: {{ app_version }}
          
          Other server IP: {{ hostvars['web2']['ansible_host'] }}
          DB Server IP: {{ hostvars['db1']['ansible_host'] }}
```

---

## ⚠️ نکات مهم

1. **نام‌گذاری**: از prefix برای جلوگیری از تداخل استفاده کنید
2. **Vault**: اطلاعات حساس را رمزنگاری کنید
3. **Default**: همیشه از `| default()` استفاده کنید
4. **Debug**: قبل از اجرا، متغیرها را چک کنید

---

## 📚 منابع

- [Using Variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
- [Variable Precedence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)
- [Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)
