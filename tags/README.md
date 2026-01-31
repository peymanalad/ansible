# 🏷️ Ansible Tags

> اجرای انتخابی task‌ها با استفاده از tags

---

## 🎯 Tags چیست؟

Tags برچسب‌هایی هستند که به task‌ها، role‌ها یا play‌ها اضافه می‌شوند و امکان اجرای انتخابی بخش‌هایی از playbook را فراهم می‌کنند.

---

## 📋 سینتکس پایه

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
    tags:
      - packages
      - nginx
  
  # یا در یک خط
  - name: Start nginx
    service:
      name: nginx
      state: started
    tags: [nginx, service]
```

---

## 🔧 اعمال Tags

### روی Task:

```yaml
tasks:
  - name: Install packages
    apt:
      name: "{{ packages }}"
      state: present
    tags:
      - install
      - packages
```

### روی Block:

```yaml
tasks:
  - name: Nginx setup
    block:
      - name: Install nginx
        apt:
          name: nginx
      
      - name: Copy config
        template:
          src: nginx.conf.j2
          dest: /etc/nginx/nginx.conf
      
      - name: Start nginx
        service:
          name: nginx
          state: started
    tags:
      - nginx
      - webserver
```

### روی Role:

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
  
  # یا
  - { role: nginx, tags: ['nginx', 'webserver'] }
```

### روی Include/Import:

```yaml
tasks:
  - name: Include nginx tasks
    include_tasks: nginx.yml
    tags:
      - nginx
  
  - name: Import database tasks
    import_tasks: database.yml
    tags:
      - database
```

### روی Play:

```yaml
---
- name: Configure webservers
  hosts: webservers
  tags:
    - webservers
  tasks:
    - name: Install nginx
      apt:
        name: nginx

- name: Configure databases
  hosts: dbservers
  tags:
    - databases
  tasks:
    - name: Install mysql
      apt:
        name: mysql-server
```

---

## 🚀 اجرا با Tags

```bash
# اجرای فقط task‌های با تگ خاص
ansible-playbook site.yml --tags "nginx"
ansible-playbook site.yml --tags "nginx,packages"
ansible-playbook site.yml -t nginx

# اجرای همه بجز تگ‌های خاص
ansible-playbook site.yml --skip-tags "nginx"
ansible-playbook site.yml --skip-tags "nginx,test"

# لیست تمام تگ‌ها
ansible-playbook site.yml --list-tags

# لیست task‌هایی که اجرا می‌شوند
ansible-playbook site.yml --tags "nginx" --list-tasks
```

---

## 🏷️ تگ‌های خاص (Special Tags)

### always:

```yaml
tasks:
  # همیشه اجرا می‌شود (حتی با --tags)
  - name: Gather facts
    setup:
    tags:
      - always
  
  - name: Check connectivity
    ping:
    tags:
      - always
  
  - name: Normal task
    debug:
      msg: "Hello"
    tags:
      - hello
```

```bash
# این task با تگ always هم اجرا می‌شود
ansible-playbook site.yml --tags "hello"
```

### never:

```yaml
tasks:
  # فقط اگر صراحتاً صدا زده شود
  - name: Dangerous cleanup
    file:
      path: /var/data
      state: absent
    tags:
      - never
      - cleanup
  
  - name: Debug info
    debug:
      var: hostvars
    tags:
      - never
      - debug
```

```bash
# اجرا نمی‌شود چون never دارد
ansible-playbook site.yml

# اجرا می‌شود چون صراحتاً cleanup را خواستیم
ansible-playbook site.yml --tags "cleanup"
```

---

## 📊 جدول Special Tags

| تگ | توضیح |
|----|-------|
| `always` | همیشه اجرا می‌شود (مگر skip شود) |
| `never` | هرگز اجرا نمی‌شود (مگر صراحتاً خواسته شود) |
| `all` | تمام task‌ها (پیش‌فرض) |
| `tagged` | فقط task‌هایی که تگ دارند |
| `untagged` | فقط task‌هایی که تگ ندارند |

```bash
# فقط task‌های بدون تگ
ansible-playbook site.yml --tags "untagged"

# فقط task‌های دارای تگ
ansible-playbook site.yml --tags "tagged"
```

---

## 📝 الگوهای استفاده

### تگ‌بندی بر اساس عملیات:

```yaml
tasks:
  - name: Install packages
    apt:
      name: nginx
    tags:
      - install
      - packages
  
  - name: Copy configuration
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags:
      - configure
      - config
  
  - name: Start service
    service:
      name: nginx
      state: started
    tags:
      - service
      - start
```

```bash
# فقط نصب
ansible-playbook site.yml --tags "install"

# فقط کانفیگ
ansible-playbook site.yml --tags "configure"
```

### تگ‌بندی بر اساس سرویس:

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
    tags:
      - nginx
  
  - name: Install mysql
    apt:
      name: mysql-server
    tags:
      - mysql
  
  - name: Install redis
    apt:
      name: redis
    tags:
      - redis
```

### تگ‌بندی بر اساس محیط:

```yaml
tasks:
  - name: Enable debug mode
    lineinfile:
      path: /etc/app/config
      line: "DEBUG=true"
    tags:
      - development
      - never
  
  - name: Enable production settings
    template:
      src: production.conf.j2
      dest: /etc/app/config
    tags:
      - production
```

---

## 🔄 ارث‌بری Tags

### در include_tasks:

```yaml
# main.yml
- name: Include nginx tasks
  include_tasks: nginx.yml
  tags:
    - nginx
  # تگ nginx به task‌های داخل nginx.yml هم اعمال می‌شود
```

### در import_tasks:

```yaml
# import_tasks تگ‌ها را به task‌های داخلی اعمال می‌کند
- name: Import nginx tasks
  import_tasks: nginx.yml
  tags:
    - nginx
```

> ⚠️ **تفاوت مهم**: در `include_tasks` تگ فقط روی خود include اعمال می‌شود، در `import_tasks` روی همه task‌های داخل فایل.

### برای apply کردن تگ به include_tasks:

```yaml
- name: Include with tags
  include_tasks:
    file: nginx.yml
    apply:
      tags:
        - nginx
  tags:
    - always  # خود include همیشه اجرا شود
```

---

## 📦 Tags در Roles

### در playbook:

```yaml
roles:
  - role: common
    tags:
      - common
      - always
  
  - role: nginx
    tags:
      - nginx
      - webserver
  
  - role: mysql
    tags:
      - mysql
      - database
```

### در role (meta/main.yml):

```yaml
# roles/nginx/meta/main.yml
---
dependencies:
  - role: common
    tags:
      - nginx
```

### در role tasks:

```yaml
# roles/nginx/tasks/main.yml
---
- name: Install nginx
  apt:
    name: nginx
  tags:
    - install

- name: Configure nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  tags:
    - configure
```

```bash
# همه task‌های role nginx
ansible-playbook site.yml --tags "nginx"

# فقط install در role nginx
ansible-playbook site.yml --tags "nginx,install"
```

---

## 🎯 مثال‌های کاربردی

### Deployment با مراحل:

```yaml
---
- name: Deploy Application
  hosts: webservers
  
  tasks:
    - name: Gather facts
      setup:
      tags:
        - always
    
    - name: Pre-deployment checks
      block:
        - name: Check disk space
          assert:
            that: ansible_mounts[0].size_available > 1073741824
        
        - name: Check connectivity
          wait_for:
            host: db.example.com
            port: 5432
      tags:
        - checks
        - pre
    
    - name: Backup current version
      archive:
        path: /var/www/app
        dest: /var/backups/app_{{ ansible_date_time.epoch }}.tar.gz
      tags:
        - backup
    
    - name: Deploy new version
      block:
        - name: Stop service
          service:
            name: myapp
            state: stopped
        
        - name: Update code
          git:
            repo: https://github.com/company/app.git
            dest: /var/www/app
        
        - name: Install dependencies
          command: npm install
          args:
            chdir: /var/www/app
        
        - name: Start service
          service:
            name: myapp
            state: started
      tags:
        - deploy
    
    - name: Post-deployment verification
      uri:
        url: http://localhost:8080/health
      register: health
      failed_when: health.status != 200
      tags:
        - verify
        - post
    
    - name: Cleanup old backups
      find:
        paths: /var/backups
        patterns: "app_*.tar.gz"
        age: 7d
      register: old_backups
      tags:
        - cleanup
        - never
    
    - name: Remove old backups
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ old_backups.files }}"
      tags:
        - cleanup
        - never
```

```bash
# فقط چک‌ها
ansible-playbook deploy.yml --tags "checks"

# دیپلوی کامل
ansible-playbook deploy.yml --tags "deploy,verify"

# همه چیز بجز cleanup
ansible-playbook deploy.yml --skip-tags "cleanup"

# شامل cleanup
ansible-playbook deploy.yml --tags "all,cleanup"
```

### تست و Debug:

```yaml
tasks:
  - name: Show all variables (debug only)
    debug:
      var: hostvars[inventory_hostname]
    tags:
      - never
      - debug
  
  - name: Test connectivity
    ping:
    tags:
      - never
      - test
  
  - name: Dry run deployment
    debug:
      msg: "Would deploy version {{ app_version }}"
    tags:
      - never
      - dryrun
```

---

## 📝 Best Practices

### 1. نام‌گذاری یکپارچه:

```yaml
# خوب
tags: [nginx, install]
tags: [nginx, configure]
tags: [nginx, service]

# بد
tags: [install-nginx]
tags: [nginx_config]
tags: [start_nginx_service]
```

### 2. استفاده از always برای پیش‌نیازها:

```yaml
- name: Gather facts
  setup:
  tags: [always]

- name: Update apt cache
  apt:
    update_cache: yes
    cache_valid_time: 3600
  tags: [always]
```

### 3. استفاده از never برای عملیات خطرناک:

```yaml
- name: Delete all data
  file:
    path: /var/data
    state: absent
  tags: [never, dangerous, cleanup]
```

### 4. مستندسازی تگ‌ها:

```yaml
# Available tags:
# - install: Install packages
# - configure: Copy configuration files
# - service: Manage services
# - deploy: Full deployment
# - rollback: Rollback to previous version (never)
# - cleanup: Remove old files (never)
```

---

## ⚠️ نکات مهم

1. **import vs include**: رفتار تگ‌ها متفاوت است
2. **never + always**: never اولویت دارد
3. **handlers**: تگ‌های handlers جداگانه هستند
4. **--list-tags**: همیشه قبل از اجرا چک کنید

---

## 📚 منابع

- [Ansible Tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html)
- [Special Tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html#special-tags)
