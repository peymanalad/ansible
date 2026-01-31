# ✅ Ansible Best Practices

> اصول و روش‌های توصیه شده برای پروژه‌های Ansible

---

## 📂 ساختار پروژه

### ساختار توصیه شده:

```
ansible-project/
├── ansible.cfg                    # تنظیمات
├── requirements.yml               # وابستگی‌ها
├── .gitignore
├── .vault_pass                    # رمز vault (در git نیست)
│
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all/
│   │   │   │   ├── vars.yml
│   │   │   │   └── vault.yml
│   │   │   ├── webservers.yml
│   │   │   └── dbservers.yml
│   │   └── host_vars/
│   │       └── special-server.yml
│   ├── staging/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   └── development/
│       └── hosts.yml
│
├── playbooks/
│   ├── site.yml                   # master playbook
│   ├── webservers.yml
│   ├── dbservers.yml
│   ├── deploy.yml
│   └── maintenance/
│       ├── backup.yml
│       ├── update.yml
│       └── cleanup.yml
│
├── roles/
│   ├── common/
│   ├── nginx/
│   ├── mysql/
│   ├── app/
│   └── monitoring/
│
├── collections/                   # Ansible Collections
│
├── filter_plugins/                # پلاگین‌های سفارشی
├── callback_plugins/
├── lookup_plugins/
│
├── files/                         # فایل‌های استاتیک مشترک
├── templates/                     # template‌های مشترک
│
└── docs/                          # مستندات
    ├── README.md
    └── runbooks/
```

---

## 🏷️ نام‌گذاری

### ✅ خوب:

```yaml
# نام‌های معنادار و توصیفی
- name: Install nginx web server
- name: Create application user with sudo access
- name: Deploy application configuration file
- name: Ensure MySQL service is running and enabled
```

### ❌ بد:

```yaml
# نام‌های مبهم
- name: Install
- name: Do stuff
- name: Task 1
- name: Fix it
```

### نام‌گذاری متغیرها:

```yaml
# ✅ با prefix نام role
nginx_worker_processes: auto
nginx_listen_port: 80
mysql_root_password: "{{ vault_mysql_root_password }}"
app_deploy_path: /var/www/app

# ❌ بدون prefix (خطر تداخل)
port: 80
password: secret
path: /var/www
```

### نام‌گذاری فایل‌ها:

```
# ✅ خوب
roles/nginx/tasks/main.yml
roles/nginx/tasks/install.yml
roles/nginx/tasks/configure.yml
group_vars/webservers.yml

# ❌ بد
roles/nginx/tasks/1.yml
roles/nginx/tasks/stuff.yml
group_vars/grp1.yml
```

---

## 🔒 امنیت

### Vault برای secrets:

```yaml
# ✅ خوب - جدا کردن secrets
# group_vars/all/vars.yml
db_password: "{{ vault_db_password }}"

# group_vars/all/vault.yml (encrypted)
vault_db_password: "ActualPassword123!"
```

### استفاده از no_log:

```yaml
# ✅ مخفی کردن خروجی حساس
- name: Set database password
  mysql_user:
    name: app
    password: "{{ db_password }}"
  no_log: true

# ✅ شرطی کردن no_log
- name: Debug sensitive task
  command: echo "{{ secret }}"
  no_log: "{{ not debug_mode | default(true) }}"
```

### دسترسی فایل‌ها:

```yaml
- name: Create config with secrets
  template:
    src: config.j2
    dest: /etc/app/config.yml
    owner: app
    group: app
    mode: '0600'  # فقط owner بخواند
```

---

## 🎯 Idempotency

### ✅ Idempotent (قابل اجرای چندباره):

```yaml
# استفاده از state
- name: Ensure package is installed
  apt:
    name: nginx
    state: present  # نه latest

- name: Ensure file exists
  file:
    path: /tmp/myfile
    state: touch
  changed_when: false  # touch همیشه changed است

# استفاده از creates/removes
- name: Run migration once
  command: /app/migrate.sh
  args:
    creates: /app/.migrated
```

### ❌ Non-idempotent:

```yaml
# بدون کنترل
- name: Append to file
  shell: echo "line" >> /etc/config
  # هر بار اجرا، یک خط اضافه می‌کند!

# ✅ اصلاح شده
- name: Ensure line exists in file
  lineinfile:
    path: /etc/config
    line: "line"
    state: present
```

---

## 📝 متغیرها

### استفاده از defaults:

```yaml
# ✅ همیشه default بگذارید
http_port: "{{ custom_port | default(80) }}"
enabled: "{{ feature_enabled | default(true) | bool }}"
users: "{{ custom_users | default([]) }}"

# در template
{{ optional_var | default('fallback') }}
```

### اولویت متغیرها:

```yaml
# roles/nginx/defaults/main.yml
# ← اولویت پایین، قابل override
nginx_port: 80
nginx_user: www-data

# roles/nginx/vars/main.yml
# ← اولویت بالا، ثابت
nginx_config_path: /etc/nginx
nginx_service_name: nginx
```

---

## 🔄 Handlers

### یک handler برای هر action:

```yaml
# ✅ خوب
handlers:
  - name: Reload nginx
    service:
      name: nginx
      state: reloaded
  
  - name: Restart nginx
    service:
      name: nginx
      state: restarted

tasks:
  - name: Update config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Reload nginx  # reload کافی است
```

### ترتیب handlers:

```yaml
handlers:
  # ترتیب مهم است!
  - name: Reload systemd
    systemd:
      daemon_reload: yes
  
  - name: Restart app
    service:
      name: myapp
      state: restarted
    # بعد از reload systemd اجرا می‌شود
```

---

## 📦 Roles

### ساختار role:

```yaml
roles/nginx/
├── defaults/main.yml      # متغیرهای قابل override
├── vars/main.yml          # متغیرهای ثابت
├── tasks/
│   ├── main.yml          # entry point
│   ├── install.yml
│   ├── configure.yml
│   └── service.yml
├── handlers/main.yml
├── templates/
├── files/
├── meta/main.yml          # وابستگی‌ها
└── README.md              # مستندات!
```

### جداسازی tasks:

```yaml
# roles/nginx/tasks/main.yml
---
- name: Include install tasks
  include_tasks: install.yml
  tags: [install]

- name: Include configure tasks
  include_tasks: configure.yml
  tags: [configure]

- name: Include service tasks
  include_tasks: service.yml
  tags: [service]
```

---

## 🏃 Performance

### محدود کردن facts:

```yaml
# فقط facts مورد نیاز
- hosts: all
  gather_facts: no
  
  tasks:
    - name: Gather minimal facts
      setup:
        gather_subset:
          - min
          - network
```

### استفاده از free strategy:

```yaml
# برای task‌های مستقل
- hosts: all
  strategy: free
  tasks:
    - name: Independent task
      command: /long-running-script.sh
```

### Pipelining:

```ini
# ansible.cfg
[ssh_connection]
pipelining = True
```

### Package list:

```yaml
# ✅ سریع - یکبار
- name: Install packages
  apt:
    name:
      - nginx
      - php
      - mysql-client
    state: present

# ❌ کند - چندبار
- name: Install packages
  apt:
    name: "{{ item }}"
  loop:
    - nginx
    - php
    - mysql-client
```

---

## 📋 Tags

### استفاده استاندارد:

```yaml
# تگ‌های استاندارد
- name: Install nginx
  apt:
    name: nginx
  tags:
    - install
    - packages
    - nginx

- name: Configure nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  tags:
    - configure
    - nginx

# تگ‌های خاص
- name: Debug info
  debug:
    var: hostvars
  tags:
    - never
    - debug
```

---

## 🧪 Testing

### Syntax check:

```bash
ansible-playbook site.yml --syntax-check
```

### Dry run:

```bash
ansible-playbook site.yml --check --diff
```

### Step by step:

```bash
ansible-playbook site.yml --step
```

### Limit:

```bash
ansible-playbook site.yml --limit "web1,web2"
```

---

## 📄 مستندات

### README برای هر role:

```markdown
# Nginx Role

## Description
Installs and configures nginx web server.

## Requirements
- Ubuntu 20.04+
- Ansible 2.9+

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| nginx_port | 80 | HTTP port |
| nginx_user | www-data | Nginx user |

## Dependencies
- common

## Example Playbook

\```yaml
- hosts: webservers
  roles:
    - role: nginx
      nginx_port: 8080
\```

## License
MIT
```

---

## 📚 چک‌لیست نهایی

- [ ] ساختار فولدربندی استاندارد
- [ ] نام‌گذاری معنادار
- [ ] Secrets در Vault
- [ ] no_log برای task‌های حساس
- [ ] همه task‌ها idempotent
- [ ] Default برای متغیرهای اختیاری
- [ ] Prefix برای متغیرهای role
- [ ] Handlers برای restart/reload
- [ ] Tags برای اجرای انتخابی
- [ ] README برای هر role
- [ ] تست با --check قبل از اجرا
