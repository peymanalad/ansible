# 📦 Ansible Blocks

> گروه‌بندی task‌ها و مدیریت خطا با block/rescue/always

---

## 🎯 Block چیست؟

Block به شما اجازه می‌دهد چندین task را گروه‌بندی کنید و روی همه آنها شرط، تنظیمات یا مدیریت خطا اعمال کنید.

---

## 📋 سینتکس پایه

```yaml
tasks:
  - name: Block of tasks
    block:
      - name: First task
        command: echo "first"
      
      - name: Second task
        command: echo "second"
```

---

## 🔧 کاربردهای Block

### 1️⃣ اعمال شرط به چند Task:

```yaml
tasks:
  # بدون block - تکراری
  - name: Install nginx
    apt:
      name: nginx
    when: ansible_os_family == "Debian"
  
  - name: Start nginx
    service:
      name: nginx
      state: started
    when: ansible_os_family == "Debian"
  
  # با block - تمیزتر
  - name: Nginx setup
    block:
      - name: Install nginx
        apt:
          name: nginx
          state: present
      
      - name: Start nginx
        service:
          name: nginx
          state: started
    when: ansible_os_family == "Debian"
```

### 2️⃣ اعمال become به چند Task:

```yaml
tasks:
  - name: Privileged tasks
    block:
      - name: Install package
        apt:
          name: nginx
      
      - name: Copy config
        copy:
          src: nginx.conf
          dest: /etc/nginx/nginx.conf
      
      - name: Start service
        service:
          name: nginx
          state: started
    become: yes
    become_user: root
```

### 3️⃣ اعمال tags:

```yaml
tasks:
  - name: Database tasks
    block:
      - name: Install MySQL
        apt:
          name: mysql-server
      
      - name: Configure MySQL
        template:
          src: my.cnf.j2
          dest: /etc/mysql/my.cnf
      
      - name: Start MySQL
        service:
          name: mysql
          state: started
    tags:
      - database
      - mysql
```

---

## 🛡️ Error Handling: block/rescue/always

ساختار try/catch/finally برای Ansible:

```yaml
tasks:
  - name: Handle errors gracefully
    block:
      # این قسمت اجرا می‌شود
      - name: Attempt risky operation
        command: /opt/risky-script.sh
      
      - name: Another task
        command: echo "success"
    
    rescue:
      # اگر block fail شود، این اجرا می‌شود
      - name: Handle failure
        debug:
          msg: "Block failed! Running rescue tasks..."
      
      - name: Rollback
        command: /opt/rollback.sh
    
    always:
      # همیشه اجرا می‌شود (چه موفق، چه fail)
      - name: Cleanup
        file:
          path: /tmp/temp_file
          state: absent
      
      - name: Send notification
        debug:
          msg: "Deployment attempt finished"
```

---

## 📊 جریان اجرا

```
┌─────────────────────────────────────────┐
│               BLOCK                     │
│  ┌─────────────────────────────────┐   │
│  │ Task 1 ──────► Task 2 ──────►   │   │
│  │                     ▼           │   │
│  │              [SUCCESS]          │   │
│  └─────────────────────────────────┘   │
│                   │                     │
│                   ▼                     │
│            ┌──────────────┐            │
│            │   ALWAYS     │            │
│            │  (cleanup)   │            │
│            └──────────────┘            │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│               BLOCK                     │
│  ┌─────────────────────────────────┐   │
│  │ Task 1 ──────► Task 2 ──X       │   │
│  │                  [FAIL]         │   │
│  └─────────────────────────────────┘   │
│                   │                     │
│                   ▼                     │
│            ┌──────────────┐            │
│            │   RESCUE     │            │
│            │ (error handling)          │
│            └──────────────┘            │
│                   │                     │
│                   ▼                     │
│            ┌──────────────┐            │
│            │   ALWAYS     │            │
│            │  (cleanup)   │            │
│            └──────────────┘            │
└─────────────────────────────────────────┘
```

---

## 📝 مثال‌های کاربردی

### Deployment با Rollback:

```yaml
- name: Deploy application
  block:
    - name: Create backup
      command: cp -r /var/www/app /var/www/app.backup
    
    - name: Pull new code
      git:
        repo: https://github.com/company/app.git
        dest: /var/www/app
        version: "{{ app_version }}"
    
    - name: Install dependencies
      command: npm install
      args:
        chdir: /var/www/app
    
    - name: Run migrations
      command: npm run migrate
      args:
        chdir: /var/www/app
    
    - name: Restart application
      service:
        name: myapp
        state: restarted
    
    - name: Health check
      uri:
        url: http://localhost:3000/health
      register: health
      failed_when: health.status != 200
  
  rescue:
    - name: Deployment failed - Rolling back
      debug:
        msg: "Deployment failed! Starting rollback..."
    
    - name: Restore backup
      command: rm -rf /var/www/app && mv /var/www/app.backup /var/www/app
    
    - name: Restart with old version
      service:
        name: myapp
        state: restarted
    
    - name: Notify team
      slack:
        token: "{{ slack_token }}"
        channel: "#deployments"
        msg: "⚠️ Deployment failed on {{ inventory_hostname }}. Rolled back."
      delegate_to: localhost
  
  always:
    - name: Cleanup backup
      file:
        path: /var/www/app.backup
        state: absent
      ignore_errors: yes
```

### سرویس با Maintenance Mode:

```yaml
- name: Update application
  block:
    - name: Enable maintenance mode
      copy:
        content: "Under maintenance"
        dest: /var/www/maintenance.html
    
    - name: Update code
      git:
        repo: "{{ app_repo }}"
        dest: /var/www/app
    
    - name: Build assets
      command: npm run build
      args:
        chdir: /var/www/app
    
    - name: Restart service
      service:
        name: myapp
        state: restarted
  
  always:
    - name: Disable maintenance mode
      file:
        path: /var/www/maintenance.html
        state: absent
```

### Database Migration:

```yaml
- name: Database migration
  block:
    - name: Create database backup
      mysql_db:
        name: myapp
        state: dump
        target: /tmp/myapp_backup.sql
    
    - name: Run migrations
      command: python manage.py migrate
      args:
        chdir: /var/www/app
      register: migration_result
  
  rescue:
    - name: Migration failed - restoring database
      mysql_db:
        name: myapp
        state: import
        target: /tmp/myapp_backup.sql
    
    - name: Fail the play
      fail:
        msg: "Database migration failed and was rolled back"
  
  always:
    - name: Remove backup file
      file:
        path: /tmp/myapp_backup.sql
        state: absent
```

---

## 🔄 Block در Loop

```yaml
- name: Process each server
  block:
    - name: Stop service
      service:
        name: myapp
        state: stopped
    
    - name: Update
      command: /opt/update.sh
    
    - name: Start service
      service:
        name: myapp
        state: started
  rescue:
    - name: Handle failure for {{ item }}
      debug:
        msg: "Failed to update {{ item }}"
  loop: "{{ groups['webservers'] }}"
  delegate_to: "{{ item }}"
```

---

## 🔍 متغیرهای خاص در rescue

```yaml
- name: Error handling example
  block:
    - name: This will fail
      command: /bin/false
  rescue:
    - name: Show error info
      debug:
        msg: |
          Task '{{ ansible_failed_task.name }}' failed.
          Result: {{ ansible_failed_result }}
```

| متغیر | توضیح |
|-------|-------|
| `ansible_failed_task` | اطلاعات task که fail شد |
| `ansible_failed_result` | نتیجه task که fail شد |

---

## 🎯 Block تو در تو

```yaml
- name: Outer block
  block:
    - name: Inner block
      block:
        - name: Risky task
          command: /opt/risky.sh
      rescue:
        - name: Inner rescue
          debug:
            msg: "Inner block failed"
    
    - name: Another task
      debug:
        msg: "This runs if inner block succeeded or was rescued"
  rescue:
    - name: Outer rescue
      debug:
        msg: "Outer block failed"
```

---

## ⚡ ترکیب با handlers

```yaml
- name: Configuration with error handling
  block:
    - name: Update config
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
      notify: Restart app
    
    - name: Force handlers now
      meta: flush_handlers
    
    - name: Verify service
      uri:
        url: http://localhost:8080/health
  rescue:
    - name: Restore config
      copy:
        src: /etc/app/app.conf.backup
        dest: /etc/app/app.conf
        remote_src: yes
      notify: Restart app

handlers:
  - name: Restart app
    service:
      name: myapp
      state: restarted
```

---

## 📝 مثال کامل

```yaml
---
- name: Safe Deployment Playbook
  hosts: webservers
  serial: 1
  
  tasks:
    - name: Deployment with full error handling
      block:
        - name: Pre-deployment checks
          block:
            - name: Check disk space
              assert:
                that: ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first > 1073741824
                fail_msg: "Not enough disk space"
            
            - name: Check service is running
              command: systemctl is-active myapp
              changed_when: false
          tags: [checks]
        
        - name: Deployment
          block:
            - name: Remove from load balancer
              uri:
                url: "http://lb/api/remove/{{ inventory_hostname }}"
                method: POST
              delegate_to: localhost
            
            - name: Wait for connections to drain
              pause:
                seconds: 30
            
            - name: Create backup
              archive:
                path: /var/www/app
                dest: /var/backups/app_{{ ansible_date_time.epoch }}.tar.gz
            
            - name: Deploy new version
              unarchive:
                src: "{{ app_package }}"
                dest: /var/www/
                remote_src: no
            
            - name: Restart service
              service:
                name: myapp
                state: restarted
            
            - name: Health check
              uri:
                url: http://localhost:8080/health
              register: health
              until: health.status == 200
              retries: 10
              delay: 3
          tags: [deploy]
      
      rescue:
        - name: Deployment failed
          debug:
            msg: "Deployment failed on {{ inventory_hostname }}"
        
        - name: Restore from backup
          unarchive:
            src: "{{ latest_backup }}"
            dest: /var/www/
            remote_src: yes
          vars:
            latest_backup: "{{ lookup('fileglob', '/var/backups/app_*.tar.gz') | sort | last }}"
        
        - name: Restart with old version
          service:
            name: myapp
            state: restarted
        
        - name: Send failure alert
          slack:
            token: "{{ slack_token }}"
            channel: "#alerts"
            msg: "❌ Deployment failed on {{ inventory_hostname }}"
          delegate_to: localhost
      
      always:
        - name: Add back to load balancer
          uri:
            url: "http://lb/api/add/{{ inventory_hostname }}"
            method: POST
          delegate_to: localhost
        
        - name: Log deployment attempt
          lineinfile:
            path: /var/log/deployments.log
            line: "{{ ansible_date_time.iso8601 }} - {{ inventory_hostname }} - {{ 'SUCCESS' if ansible_failed_task is undefined else 'FAILED' }}"
          delegate_to: localhost
```

---

## ⚠️ نکات مهم

1. **rescue فقط برای خطا**: اگر همه task‌های block موفق باشند، rescue اجرا نمی‌شود
2. **always همیشه**: حتی اگر rescue هم fail شود، always اجرا می‌شود
3. **تگ‌ها روی block**: به همه task‌های داخل اعمال می‌شود
4. **handlers**: بعد از block اجرا می‌شوند، نه داخل آن

---

## 📚 منابع

- [Ansible Blocks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_blocks.html)
- [Error Handling](https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html)
