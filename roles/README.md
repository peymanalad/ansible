# 🎭 Ansible Roles

> سازماندهی و بازاستفاده از کد با Roles

---

## 🎯 Role چیست؟

Role یک روش استاندارد برای سازماندهی task‌ها، متغیرها، فایل‌ها، template‌ها و handlers در یک ساختار مشخص است که امکان بازاستفاده و اشتراک‌گذاری را فراهم می‌کند.

---

## 📂 ساختار Role

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml          # task‌های اصلی
    ├── handlers/
    │   └── main.yml          # handlers
    ├── templates/
    │   └── nginx.conf.j2     # template‌ها
    ├── files/
    │   └── ssl.crt           # فایل‌های استاتیک
    ├── vars/
    │   └── main.yml          # متغیرهای با اولویت بالا
    ├── defaults/
    │   └── main.yml          # متغیرهای پیش‌فرض
    ├── meta/
    │   └── main.yml          # متادیتا و وابستگی‌ها
    ├── library/              # ماژول‌های سفارشی
    ├── module_utils/         # ابزارهای ماژول
    ├── lookup_plugins/       # lookup پلاگین‌ها
    └── README.md             # مستندات
```

---

## 🔧 ایجاد Role

### با دستور:

```bash
# ایجاد ساختار role
ansible-galaxy init nginx

# با namespace
ansible-galaxy init --init-path roles/ company.nginx
```

### دستی:

```bash
mkdir -p roles/nginx/{tasks,handlers,templates,files,vars,defaults,meta}
touch roles/nginx/{tasks,handlers,vars,defaults,meta}/main.yml
```

---

## 📝 فایل‌های اصلی Role

### tasks/main.yml:

```yaml
---
# roles/nginx/tasks/main.yml

- name: Install nginx
  apt:
    name: nginx
    state: present
  notify: Restart nginx

- name: Copy nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Reload nginx

- name: Create sites-available directory
  file:
    path: /etc/nginx/sites-available
    state: directory
    mode: '0755'

- name: Enable default site
  file:
    src: /etc/nginx/sites-available/default
    dest: /etc/nginx/sites-enabled/default
    state: link
  notify: Reload nginx

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

### handlers/main.yml:

```yaml
---
# roles/nginx/handlers/main.yml

- name: Restart nginx
  service:
    name: nginx
    state: restarted

- name: Reload nginx
  service:
    name: nginx
    state: reloaded
```

### defaults/main.yml:

```yaml
---
# roles/nginx/defaults/main.yml
# متغیرهای پیش‌فرض (قابل override)

nginx_worker_processes: auto
nginx_worker_connections: 1024
nginx_keepalive_timeout: 65

nginx_http_port: 80
nginx_https_port: 443

nginx_user: www-data
nginx_group: www-data

nginx_access_log: /var/log/nginx/access.log
nginx_error_log: /var/log/nginx/error.log

nginx_sites: []
```

### vars/main.yml:

```yaml
---
# roles/nginx/vars/main.yml
# متغیرهای با اولویت بالا (سخت override می‌شوند)

nginx_package_name: nginx
nginx_service_name: nginx
nginx_config_path: /etc/nginx
nginx_pid_path: /run/nginx.pid
```

### meta/main.yml:

```yaml
---
# roles/nginx/meta/main.yml

galaxy_info:
  author: Your Name
  description: Install and configure nginx
  company: Your Company
  license: MIT
  min_ansible_version: "2.9"
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: Debian
      versions:
        - bullseye
  galaxy_tags:
    - nginx
    - webserver
    - proxy

dependencies:
  - role: common
  - role: firewall
    vars:
      firewall_allowed_ports:
        - 80
        - 443
```

### templates/nginx.conf.j2:

```jinja2
# {{ ansible_managed }}
# Nginx configuration

user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};
pid {{ nginx_pid_path }};

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
    sendfile on;
    tcp_nopush on;
    keepalive_timeout {{ nginx_keepalive_timeout }};
    
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    access_log {{ nginx_access_log }};
    error_log {{ nginx_error_log }};
    
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## 🚀 استفاده از Role

### روش 1: roles در play:

```yaml
---
- name: Configure webservers
  hosts: webservers
  become: yes
  
  roles:
    - nginx
    - php
    - mysql
```

### روش 2: با متغیرها:

```yaml
roles:
  - role: nginx
    vars:
      nginx_http_port: 8080
      nginx_worker_processes: 4
  
  - role: nginx
    nginx_http_port: 8080  # shorthand
```

### روش 3: با شرط:

```yaml
roles:
  - role: nginx
    when: "'webserver' in group_names"
  
  - role: mysql
    when: "'database' in group_names"
```

### روش 4: با tags:

```yaml
roles:
  - role: nginx
    tags:
      - nginx
      - webserver
  
  - role: mysql
    tags:
      - mysql
      - database
```

### روش 5: include_role / import_role:

```yaml
tasks:
  - name: Include nginx role
    include_role:
      name: nginx
    vars:
      nginx_http_port: 8080
  
  - name: Import mysql role
    import_role:
      name: mysql
    when: install_mysql | bool
```

---

## 📊 اولویت متغیرها در Role

از پایین به بالا (بالاتر = اولویت بیشتر):

```
1. role defaults (roles/x/defaults/main.yml)
2. inventory vars
3. inventory group_vars
4. inventory host_vars
5. playbook group_vars
6. playbook host_vars
7. host facts
8. play vars
9. play vars_files
10. role vars (roles/x/vars/main.yml)
11. block vars
12. task vars
13. include_vars
14. set_facts
15. extra vars (-e)
```

---

## 🔗 وابستگی‌های Role

### تعریف در meta/main.yml:

```yaml
---
dependencies:
  # ساده
  - common
  
  # با متغیر
  - role: firewall
    vars:
      firewall_allowed_ports:
        - 80
        - 443
  
  # با شرط
  - role: mysql-client
    when: mysql_enabled | default(true)
  
  # با tags
  - role: common
    tags:
      - always
```

### ترتیب اجرا:

```
1. Role dependencies (به ترتیب)
2. Pre-tasks
3. Role tasks
4. Tasks
5. Post-tasks
6. Handlers
```

---

## 📦 Ansible Galaxy

### نصب Role از Galaxy:

```bash
# نصب role
ansible-galaxy install geerlingguy.nginx

# با نام سفارشی
ansible-galaxy install geerlingguy.nginx,nginx

# نصب در مسیر خاص
ansible-galaxy install geerlingguy.nginx -p ./roles

# نصب از فایل requirements
ansible-galaxy install -r requirements.yml
```

### requirements.yml:

```yaml
---
roles:
  # از Galaxy
  - name: geerlingguy.nginx
    version: 4.0.0
  
  # از GitHub
  - name: nginx
    src: https://github.com/user/ansible-nginx
    version: master
  
  # از Git با SSH
  - name: company.custom_role
    src: git@github.com:company/custom_role.git
    version: v1.2.0
  
  # از فایل tar
  - name: local_role
    src: file:///path/to/role.tar.gz

collections:
  - name: community.general
    version: ">=3.0.0"
```

### لیست و مدیریت:

```bash
# لیست role‌های نصب شده
ansible-galaxy list

# حذف role
ansible-galaxy remove geerlingguy.nginx

# جستجو در Galaxy
ansible-galaxy search nginx

# اطلاعات role
ansible-galaxy info geerlingguy.nginx
```

---

## 📂 ساختار پروژه با Roles

```
project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   └── all.yml
│   │   └── host_vars/
│   └── staging/
├── playbooks/
│   ├── site.yml              # اصلی
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   ├── php/
│   └── mysql/
├── galaxy_roles/             # roles از Galaxy
├── requirements.yml
└── group_vars/
    └── all/
        ├── vars.yml
        └── vault.yml
```

### ansible.cfg:

```ini
[defaults]
roles_path = ./roles:./galaxy_roles
```

### site.yml:

```yaml
---
- name: Apply common configuration
  hosts: all
  become: yes
  roles:
    - common

- name: Configure webservers
  hosts: webservers
  become: yes
  roles:
    - nginx
    - php

- name: Configure databases
  hosts: dbservers
  become: yes
  roles:
    - mysql
```

---

## 🔧 تکنیک‌های پیشرفته

### Include tasks بر اساس OS:

```yaml
# roles/nginx/tasks/main.yml
---
- name: Include OS-specific tasks
  include_tasks: "{{ ansible_os_family | lower }}.yml"
```

```yaml
# roles/nginx/tasks/debian.yml
---
- name: Install nginx on Debian
  apt:
    name: nginx
    state: present
```

```yaml
# roles/nginx/tasks/redhat.yml
---
- name: Install nginx on RedHat
  yum:
    name: nginx
    state: present
```

### چند instance از یک role:

```yaml
roles:
  - role: nginx
    vars:
      nginx_vhost_name: site1
      nginx_port: 8080
  
  - role: nginx
    vars:
      nginx_vhost_name: site2
      nginx_port: 8081
```

### Conditional role files:

```yaml
# roles/app/tasks/main.yml
---
- name: Include environment-specific vars
  include_vars: "{{ env }}.yml"

- name: Include deployment tasks
  include_tasks: "deploy_{{ deployment_method }}.yml"
```

---

## 📝 مثال Role کامل

### ساختار:

```
roles/myapp/
├── defaults/main.yml
├── files/
│   └── myapp.service
├── handlers/main.yml
├── meta/main.yml
├── tasks/
│   ├── main.yml
│   ├── install.yml
│   ├── configure.yml
│   └── service.yml
├── templates/
│   └── config.yml.j2
└── vars/main.yml
```

### defaults/main.yml:

```yaml
---
myapp_version: "1.0.0"
myapp_port: 8080
myapp_user: myapp
myapp_group: myapp
myapp_home: /opt/myapp
myapp_config_path: /etc/myapp
myapp_log_path: /var/log/myapp

myapp_database:
  host: localhost
  port: 5432
  name: myapp
  user: myapp
```

### tasks/main.yml:

```yaml
---
- name: Include install tasks
  include_tasks: install.yml
  tags:
    - install

- name: Include configure tasks
  include_tasks: configure.yml
  tags:
    - configure

- name: Include service tasks
  include_tasks: service.yml
  tags:
    - service
```

### tasks/install.yml:

```yaml
---
- name: Create app group
  group:
    name: "{{ myapp_group }}"
    state: present

- name: Create app user
  user:
    name: "{{ myapp_user }}"
    group: "{{ myapp_group }}"
    home: "{{ myapp_home }}"
    shell: /bin/false
    system: yes

- name: Create directories
  file:
    path: "{{ item }}"
    state: directory
    owner: "{{ myapp_user }}"
    group: "{{ myapp_group }}"
    mode: '0755'
  loop:
    - "{{ myapp_home }}"
    - "{{ myapp_config_path }}"
    - "{{ myapp_log_path }}"

- name: Download application
  get_url:
    url: "https://releases.myapp.com/{{ myapp_version }}/myapp.tar.gz"
    dest: /tmp/myapp.tar.gz

- name: Extract application
  unarchive:
    src: /tmp/myapp.tar.gz
    dest: "{{ myapp_home }}"
    remote_src: yes
    owner: "{{ myapp_user }}"
    group: "{{ myapp_group }}"
```

### tasks/configure.yml:

```yaml
---
- name: Copy configuration
  template:
    src: config.yml.j2
    dest: "{{ myapp_config_path }}/config.yml"
    owner: "{{ myapp_user }}"
    group: "{{ myapp_group }}"
    mode: '0600'
  notify: Restart myapp

- name: Copy systemd service
  copy:
    src: myapp.service
    dest: /etc/systemd/system/myapp.service
  notify:
    - Reload systemd
    - Restart myapp
```

### handlers/main.yml:

```yaml
---
- name: Reload systemd
  systemd:
    daemon_reload: yes

- name: Restart myapp
  service:
    name: myapp
    state: restarted

- name: Reload myapp
  service:
    name: myapp
    state: reloaded
```

### استفاده:

```yaml
---
- name: Deploy MyApp
  hosts: appservers
  become: yes
  
  roles:
    - role: myapp
      vars:
        myapp_version: "2.0.0"
        myapp_port: 9090
        myapp_database:
          host: db.example.com
          port: 5432
          name: myapp_prod
          user: myapp_prod
```

---

## ⚠️ Best Practices

1. **defaults برای قابلیت override**: همه متغیرهای قابل تغییر در defaults
2. **vars برای ثابت‌ها**: مقادیر ثابت در vars
3. **مستندسازی**: README.md با مثال‌ها
4. **تست**: از Molecule برای تست استفاده کنید
5. **نسخه‌بندی**: از Git tags استفاده کنید
6. **نام‌گذاری**: prefix با نام role برای متغیرها

---

## 📚 منابع

- [Ansible Roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Role Directory Structure](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-directory-structure)
