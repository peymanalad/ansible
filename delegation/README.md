# 🎯 Task Delegation in Ansible

> اجرای task روی سرور دیگر با delegate_to

---

## 🎯 Delegation چیست؟

Delegation به شما اجازه می‌دهد یک task را روی سرور دیگری (نه سرور هدف فعلی) اجرا کنید، در حالی که متغیرها و facts همچنان مربوط به سرور اصلی هستند.

---

## 📋 سینتکس پایه

```yaml
tasks:
  - name: Run on different host
    command: echo "Hello from {{ inventory_hostname }}"
    delegate_to: other_server
```

---

## 🔧 موارد استفاده رایج

### 1️⃣ اجرا روی localhost:

```yaml
tasks:
  # دانلود فایل به کنترلر
  - name: Download file to controller
    fetch:
      src: /var/log/app.log
      dest: /tmp/logs/
    # یا با delegate_to
  
  - name: Run locally
    command: aws s3 cp s3://bucket/file /tmp/
    delegate_to: localhost
  
  # استفاده از local_action (معادل delegate_to: localhost)
  - name: Check from controller
    local_action:
      module: uri
      url: "http://{{ ansible_host }}/health"
```

### 2️⃣ ثبت در Load Balancer:

```yaml
tasks:
  # حذف از LB قبل از maintenance
  - name: Remove from load balancer
    uri:
      url: "http://lb.example.com/api/remove"
      method: POST
      body:
        server: "{{ inventory_hostname }}"
      body_format: json
    delegate_to: localhost
  
  # انجام کارها روی سرور
  - name: Upgrade application
    apt:
      name: myapp
      state: latest
  
  # اضافه به LB بعد از کار
  - name: Add back to load balancer
    uri:
      url: "http://lb.example.com/api/add"
      method: POST
      body:
        server: "{{ inventory_hostname }}"
      body_format: json
    delegate_to: localhost
```

### 3️⃣ اجرا روی DB Server:

```yaml
tasks:
  - name: Create database for this app server
    mysql_db:
      name: "app_{{ inventory_hostname | replace('.', '_') }}"
      state: present
    delegate_to: db.example.com
```

### 4️⃣ کپی بین سرورها:

```yaml
tasks:
  - name: Sync files from master
    synchronize:
      src: /var/www/html/
      dest: /var/www/html/
    delegate_to: master.example.com
```

---

## 🔄 delegate_facts

ذخیره facts در سرور delegate شده:

```yaml
tasks:
  - name: Gather facts from load balancer
    setup:
    delegate_to: lb.example.com
    delegate_facts: true
  
  # حالا facts مربوط به lb در hostvars ذخیره شده
  - name: Show LB memory
    debug:
      msg: "LB has {{ hostvars['lb.example.com']['ansible_memtotal_mb'] }} MB RAM"
```

---

## 🏃 run_once

اجرای فقط یکبار (معمولاً با delegate_to):

```yaml
tasks:
  # فقط یکبار اجرا می‌شود، نه برای هر سرور
  - name: Update DNS
    route53:
      zone: example.com
      record: app.example.com
      type: A
      value: "{{ groups['webservers'] | map('extract', hostvars, 'ansible_host') | join(',') }}"
    delegate_to: localhost
    run_once: true
  
  # ایجاد دیتابیس فقط یکبار
  - name: Create shared database
    mysql_db:
      name: shared_db
      state: present
    delegate_to: db.example.com
    run_once: true
```

---

## 🔧 connection: local

جایگزین delegate_to: localhost:

```yaml
# برای یک task
- name: Local task
  command: whoami
  delegate_to: localhost
  connection: local

# برای کل play
- name: Run everything locally
  hosts: webservers
  connection: local
  
  tasks:
    - name: Generate config locally
      template:
        src: config.j2
        dest: "/tmp/configs/{{ inventory_hostname }}.conf"
```

---

## 📊 Serial و Delegation

ترکیب با rolling updates:

```yaml
---
- name: Rolling update
  hosts: webservers
  serial: 1  # یکی یکی
  
  tasks:
    - name: Remove from LB
      uri:
        url: "http://lb/api/disable/{{ inventory_hostname }}"
      delegate_to: localhost
    
    - name: Wait for connections to drain
      wait_for:
        timeout: 30
      delegate_to: localhost
    
    - name: Update application
      apt:
        name: myapp
        state: latest
    
    - name: Restart service
      service:
        name: myapp
        state: restarted
    
    - name: Health check
      uri:
        url: "http://{{ ansible_host }}/health"
      delegate_to: localhost
      register: health
      until: health.status == 200
      retries: 10
      delay: 5
    
    - name: Add back to LB
      uri:
        url: "http://lb/api/enable/{{ inventory_hostname }}"
      delegate_to: localhost
```

---

## 📝 مثال‌های کاربردی

### Monitoring Integration:

```yaml
tasks:
  - name: Disable monitoring alerts
    uri:
      url: "https://monitoring.example.com/api/silence"
      method: POST
      body:
        host: "{{ inventory_hostname }}"
        duration: 3600
      body_format: json
      headers:
        Authorization: "Bearer {{ monitoring_token }}"
    delegate_to: localhost
  
  - name: Perform maintenance
    include_tasks: maintenance.yml
  
  - name: Re-enable monitoring
    uri:
      url: "https://monitoring.example.com/api/unsilence"
      method: POST
      body:
        host: "{{ inventory_hostname }}"
      body_format: json
    delegate_to: localhost
```

### Cross-datacenter Operations:

```yaml
tasks:
  - name: Sync config to DR site
    copy:
      src: /etc/myapp/config.yml
      dest: /etc/myapp/config.yml
    delegate_to: "{{ item }}"
    loop: "{{ groups['dr_servers'] }}"
```

### Cluster Operations:

```yaml
tasks:
  - name: Notify cluster master
    command: /usr/bin/cluster-ctl notify-update {{ inventory_hostname }}
    delegate_to: "{{ groups['cluster_masters'][0] }}"
    run_once: true
```

---

## ⚡ wait_for_connection

صبر برای آماده شدن سرور (مثلاً بعد از reboot):

```yaml
tasks:
  - name: Reboot server
    reboot:
      reboot_timeout: 600
  
  # یا به صورت دستی
  - name: Reboot
    command: /sbin/shutdown -r now
    async: 1
    poll: 0
  
  - name: Wait for server to come back
    wait_for_connection:
      delay: 30
      timeout: 300
    delegate_to: localhost
```

---

## 📊 جدول مقایسه

| روش | کاربرد |
|-----|--------|
| `delegate_to: localhost` | اجرای task روی کنترلر |
| `delegate_to: other_host` | اجرا روی سرور دیگر |
| `local_action` | معادل delegate_to: localhost |
| `connection: local` | تغییر connection برای کل play |
| `run_once: true` | فقط یکبار اجرا شود |
| `delegate_facts: true` | facts در سرور delegate ذخیره شود |

---

## 📝 مثال کامل

```yaml
---
- name: Zero-downtime Deployment
  hosts: webservers
  serial: "25%"  # 25% سرورها همزمان
  
  vars:
    lb_api: "http://lb.example.com/api"
    health_endpoint: "/health"
  
  pre_tasks:
    - name: Disable server in load balancer
      uri:
        url: "{{ lb_api }}/servers/{{ inventory_hostname }}/disable"
        method: POST
      delegate_to: localhost
    
    - name: Wait for connections to drain
      pause:
        seconds: 30
  
  tasks:
    - name: Pull latest code
      git:
        repo: https://github.com/company/app.git
        dest: /var/www/app
        version: "{{ app_version }}"
    
    - name: Install dependencies
      command: pip install -r requirements.txt
      args:
        chdir: /var/www/app
    
    - name: Restart application
      service:
        name: myapp
        state: restarted
    
    - name: Wait for application to start
      uri:
        url: "http://localhost{{ health_endpoint }}"
      register: health_check
      until: health_check.status == 200
      retries: 30
      delay: 2
  
  post_tasks:
    - name: Enable server in load balancer
      uri:
        url: "{{ lb_api }}/servers/{{ inventory_hostname }}/enable"
        method: POST
      delegate_to: localhost
    
    - name: Verify external health
      uri:
        url: "http://{{ ansible_host }}{{ health_endpoint }}"
      delegate_to: localhost
      register: external_health
      until: external_health.status == 200
      retries: 10
      delay: 5
    
    - name: Notify deployment success
      slack:
        token: "{{ slack_token }}"
        channel: "#deployments"
        msg: "{{ inventory_hostname }} updated successfully"
      delegate_to: localhost
      run_once: true
      when: ansible_play_hosts | last == inventory_hostname
```

---

## ⚠️ نکات مهم

1. **متغیرها**: در delegation، متغیرها مربوط به سرور اصلی هستند
2. **hostvars**: برای دسترسی به متغیرهای سرور delegate از `hostvars` استفاده کنید
3. **become**: ممکن است نیاز به تنظیم جداگانه داشته باشد
4. **connection**: برای localhost معمولاً `connection: local` بهتر است

---

## 📚 منابع

- [Delegation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html)
- [Rolling Updates](https://docs.ansible.com/ansible/latest/user_guide/playbooks_strategies.html)
