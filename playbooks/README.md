# 📜 Ansible Playbooks

> نوشتن و اجرای Playbook‌ها در Ansible

---

## 🎯 Playbook چیست؟

Playbook فایل YAML است که مجموعه‌ای از task‌ها را برای اجرا روی سرورها تعریف می‌کند. برخلاف دستورات ad-hoc، playbook‌ها قابل ذخیره، بازاستفاده و version control هستند.

---

## 📋 ساختار پایه Playbook

```yaml
---
# playbook.yml
- name: نام Play (توضیح کلی)
  hosts: webservers          # روی کدام سرورها
  become: yes                # با sudo اجرا شود
  vars:                      # متغیرها
    http_port: 80
  
  tasks:                     # لیست task‌ها
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

---

## 🔧 اجزای Playbook

### 1️⃣ Play

هر playbook شامل یک یا چند play است:

```yaml
---
# چند play در یک playbook
- name: Configure webservers
  hosts: webservers
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

- name: Configure databases
  hosts: dbservers
  tasks:
    - name: Install mysql
      apt:
        name: mysql-server
        state: present
```

### 2️⃣ Tasks

هر task یک action است:

```yaml
tasks:
  # فرمت ساده
  - name: Install package
    apt:
      name: nginx
      state: present
  
  # فرمت یک خطی (برای task‌های ساده)
  - name: Create file
    file: path=/tmp/test state=touch
  
  # با متغیر
  - name: Install package
    apt:
      name: "{{ package_name }}"
      state: present
```

### 3️⃣ Handlers

task‌هایی که فقط وقتی تغییری رخ دهد اجرا می‌شوند:

```yaml
tasks:
  - name: Copy nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx    # صدا زدن handler

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

---

## 📝 متغیرها در Playbook

```yaml
---
- name: Using variables
  hosts: all
  
  # تعریف متغیر در playbook
  vars:
    username: deploy
    packages:
      - nginx
      - vim
      - htop
  
  # خواندن از فایل
  vars_files:
    - vars/common.yml
    - vars/{{ env }}.yml
  
  # پرسیدن از کاربر
  vars_prompt:
    - name: admin_password
      prompt: "Enter admin password"
      private: yes
  
  tasks:
    - name: Create user
      user:
        name: "{{ username }}"
        state: present
    
    - name: Install packages
      apt:
        name: "{{ packages }}"
        state: present
```

---

## 🎯 ساختار فولدربندی توصیه شده

```
project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   └── hosts.yml
│   └── staging/
│       └── hosts.yml
├── playbooks/
│   ├── site.yml              # playbook اصلی
│   ├── webservers.yml
│   └── dbservers.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── mysql/
├── group_vars/
│   ├── all.yml
│   └── webservers.yml
├── host_vars/
│   └── web1.yml
├── files/
│   └── nginx.conf
├── templates/
│   └── nginx.conf.j2
└── vars/
    ├── common.yml
    └── secrets.yml
```

---

## 🚀 اجرای Playbook

```bash
# اجرای ساده
ansible-playbook playbook.yml

# با inventory خاص
ansible-playbook -i inventory/production playbook.yml

# با محدود کردن به سرورهای خاص
ansible-playbook playbook.yml --limit webservers
ansible-playbook playbook.yml --limit web1,web2

# با متغیر از command line
ansible-playbook playbook.yml -e "env=production"
ansible-playbook playbook.yml -e '{"users": ["ali", "reza"]}'

# Dry run (چک بدون اجرا)
ansible-playbook playbook.yml --check

# نمایش تغییرات (diff)
ansible-playbook playbook.yml --check --diff

# با verbose
ansible-playbook playbook.yml -v    # یا -vv، -vvv، -vvvv

# شروع از task خاص
ansible-playbook playbook.yml --start-at-task="Install nginx"

# اجرای step by step
ansible-playbook playbook.yml --step

# لیست task‌ها
ansible-playbook playbook.yml --list-tasks

# لیست hosts
ansible-playbook playbook.yml --list-hosts

# لیست tags
ansible-playbook playbook.yml --list-tags
```

---

## 📦 Import و Include

### Import (استاتیک - در زمان parse):

```yaml
---
- name: Main playbook
  hosts: all
  
  tasks:
    - name: Common tasks
      import_tasks: tasks/common.yml
    
    - name: Include role
      import_role:
        name: nginx

# برای import کردن playbook دیگر
- import_playbook: webservers.yml
- import_playbook: dbservers.yml
```

### Include (داینامیک - در زمان اجرا):

```yaml
tasks:
  - name: Include tasks based on OS
    include_tasks: "{{ ansible_os_family }}.yml"
  
  - name: Include role with variables
    include_role:
      name: nginx
    vars:
      nginx_port: 8080
```

---

## 🔄 Pre/Post Tasks

```yaml
---
- name: Deploy application
  hosts: webservers
  
  pre_tasks:
    - name: Disable monitoring
      uri:
        url: "http://monitor/disable/{{ inventory_hostname }}"
        method: POST
  
  roles:
    - nginx
    - app
  
  tasks:
    - name: Deploy code
      git:
        repo: https://github.com/user/app.git
        dest: /var/www/app
  
  post_tasks:
    - name: Enable monitoring
      uri:
        url: "http://monitor/enable/{{ inventory_hostname }}"
        method: POST
```

---

## 📊 نمونه Playbook کامل

```yaml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  gather_facts: yes
  
  vars:
    http_port: 80
    document_root: /var/www/html
    packages:
      - nginx
      - php-fpm
      - php-mysql
  
  vars_files:
    - vars/secrets.yml
  
  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
  
  tasks:
    - name: Install packages
      apt:
        name: "{{ packages }}"
        state: present
    
    - name: Create document root
      file:
        path: "{{ document_root }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
    
    - name: Copy nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Reload nginx
    
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link
    
    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
  
  post_tasks:
    - name: Check website
      uri:
        url: "http://{{ inventory_hostname }}:{{ http_port }}"
        status_code: 200
      delegate_to: localhost
```

---

## ⚠️ نکات مهم

1. **Idempotency**: task‌ها باید چندبار قابل اجرا باشند بدون تغییر نتیجه
2. **نام‌گذاری**: همیشه name معنادار بنویسید
3. **--check**: قبل از اجرای واقعی، حتماً dry run کنید
4. **handlers**: برای restart سرویس‌ها از handler استفاده کنید

---

## 📚 منابع

- [Ansible Playbook Guide](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)
- [Playbook Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
