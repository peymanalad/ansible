# 📝 Ansible Register

> ذخیره و استفاده از خروجی task‌ها با register

---

## 🎯 Register چیست؟

`register` به شما اجازه می‌دهد خروجی یک task را در یک متغیر ذخیره کنید و در task‌های بعدی از آن استفاده کنید.

---

## 📋 سینتکس پایه

```yaml
tasks:
  - name: Run a command
    command: hostname
    register: result
  
  - name: Show the result
    debug:
      var: result
```

---

## 📊 ساختار متغیر Register شده

هر ماژول خروجی متفاوتی دارد، اما فیلدهای مشترک:

```yaml
{
  "changed": true,           # آیا تغییری ایجاد شد؟
  "failed": false,           # آیا خطا داشت؟
  "rc": 0,                   # return code (برای command/shell)
  "stdout": "output",        # خروجی استاندارد
  "stdout_lines": ["line1"], # خروجی به صورت لیست
  "stderr": "",              # خروجی خطا
  "stderr_lines": [],        # خروجی خطا به صورت لیست
  "cmd": ["hostname"],       # دستور اجرا شده
  "start": "2024-01-01...",  # زمان شروع
  "end": "2024-01-01...",    # زمان پایان
  "delta": "0:00:00.123"     # مدت زمان اجرا
}
```

---

## 🔧 استفاده‌های رایج

### 1. دسترسی به خروجی:

```yaml
tasks:
  - name: Get hostname
    command: hostname
    register: hostname_result
  
  - name: Show stdout
    debug:
      msg: "Hostname is {{ hostname_result.stdout }}"
  
  - name: Show as list
    debug:
      msg: "Lines: {{ hostname_result.stdout_lines }}"
```

### 2. بررسی موفقیت:

```yaml
tasks:
  - name: Check service
    command: systemctl is-active nginx
    register: nginx_status
    ignore_errors: yes
  
  - name: Start if not running
    service:
      name: nginx
      state: started
    when: nginx_status.rc != 0
  
  - name: Report status
    debug:
      msg: "Nginx is {{ 'running' if nginx_status.rc == 0 else 'not running' }}"
```

### 3. استفاده در شرط:

```yaml
tasks:
  - name: Check if file exists
    stat:
      path: /etc/myapp/config.yml
    register: config_stat
  
  - name: Create config if not exists
    template:
      src: config.yml.j2
      dest: /etc/myapp/config.yml
    when: not config_stat.stat.exists
  
  - name: Backup if exists and is file
    copy:
      src: /etc/myapp/config.yml
      dest: /etc/myapp/config.yml.bak
      remote_src: yes
    when: 
      - config_stat.stat.exists
      - config_stat.stat.isreg  # is regular file
```

### 4. Loop با نتایج:

```yaml
tasks:
  - name: Get list of files
    find:
      paths: /tmp
      patterns: "*.log"
    register: log_files
  
  - name: Show each file
    debug:
      msg: "Found: {{ item.path }}"
    loop: "{{ log_files.files }}"
  
  - name: Delete old logs
    file:
      path: "{{ item.path }}"
      state: absent
    loop: "{{ log_files.files }}"
    when: item.size > 1048576  # بزرگتر از 1MB
```

---

## 📦 Register با ماژول‌های مختلف

### Command/Shell:

```yaml
- name: Run shell command
  shell: cat /etc/passwd | grep root
  register: grep_result
  
- debug:
    msg: |
      stdout: {{ grep_result.stdout }}
      stderr: {{ grep_result.stderr }}
      return code: {{ grep_result.rc }}
```

### Stat Module:

```yaml
- name: Get file info
  stat:
    path: /etc/nginx/nginx.conf
  register: nginx_conf
  
- debug:
    msg: |
      exists: {{ nginx_conf.stat.exists }}
      size: {{ nginx_conf.stat.size | default(0) }} bytes
      mode: {{ nginx_conf.stat.mode | default('N/A') }}
      owner: {{ nginx_conf.stat.pw_name | default('N/A') }}
      is_dir: {{ nginx_conf.stat.isdir | default(false) }}
      checksum: {{ nginx_conf.stat.checksum | default('N/A') }}
```

### URI Module:

```yaml
- name: Call API
  uri:
    url: https://api.example.com/health
    method: GET
    return_content: yes
  register: api_response
  
- debug:
    msg: |
      Status: {{ api_response.status }}
      Body: {{ api_response.content }}
      JSON: {{ api_response.json }}
```

### Find Module:

```yaml
- name: Find files
  find:
    paths: /var/log
    patterns: "*.log"
    age: 7d
    size: 1m
  register: old_logs
  
- debug:
    msg: "Found {{ old_logs.matched }} files"
  
- name: List files
  debug:
    msg: "{{ item.path }} - {{ item.size }} bytes"
  loop: "{{ old_logs.files }}"
```

### Package Module:

```yaml
- name: Install package
  apt:
    name: nginx
    state: present
  register: install_result
  
- debug:
    msg: "Package was {{ 'installed' if install_result.changed else 'already present' }}"
```

### User Module:

```yaml
- name: Create user
  user:
    name: deploy
    generate_ssh_key: yes
  register: user_result
  
- debug:
    msg: "SSH key: {{ user_result.ssh_public_key }}"
  when: user_result.ssh_public_key is defined
```

---

## 🔄 Register در Loop

```yaml
tasks:
  - name: Check multiple services
    command: systemctl is-active {{ item }}
    loop:
      - nginx
      - mysql
      - redis
    register: services_status
    ignore_errors: yes
  
  - name: Show results
    debug:
      msg: "{{ item.item }}: {{ 'running' if item.rc == 0 else 'stopped' }}"
    loop: "{{ services_status.results }}"
  
  - name: Start stopped services
    service:
      name: "{{ item.item }}"
      state: started
    loop: "{{ services_status.results }}"
    when: item.rc != 0
```

### ساختار results در loop:

```yaml
services_status:
  results:
    - item: nginx      # آیتم اصلی loop
      rc: 0
      stdout: active
      changed: false
    - item: mysql
      rc: 3
      stdout: inactive
      changed: false
```

---

## ⚡ changed_when و failed_when

### کنترل changed:

```yaml
tasks:
  # این command همیشه changed=true می‌شود
  - name: Check something
    command: cat /etc/hostname
    register: result
    changed_when: false  # هیچوقت changed نشو
  
  # changed بر اساس خروجی
  - name: Update config
    command: /opt/app/update-config.sh
    register: update_result
    changed_when: "'Updated' in update_result.stdout"
```

### کنترل failed:

```yaml
tasks:
  - name: Check user exists
    command: id deploy
    register: user_check
    failed_when: false  # هیچوقت fail نکن
  
  - name: Create user if not exists
    user:
      name: deploy
    when: user_check.rc != 0
  
  # fail بر اساس خروجی
  - name: Run critical script
    command: /opt/app/critical.sh
    register: script_result
    failed_when: 
      - script_result.rc != 0
      - "'SKIP' not in script_result.stdout"
```

---

## 🎯 الگوهای کاربردی

### Check and Create Pattern:

```yaml
- name: Check if app is installed
  stat:
    path: /opt/myapp/bin/app
  register: app_binary

- name: Download and install app
  block:
    - name: Download
      get_url:
        url: https://example.com/app.tar.gz
        dest: /tmp/app.tar.gz
    
    - name: Extract
      unarchive:
        src: /tmp/app.tar.gz
        dest: /opt/myapp
        remote_src: yes
  when: not app_binary.stat.exists
```

### Retry Pattern:

```yaml
- name: Wait for service to be ready
  uri:
    url: http://localhost:8080/health
  register: health_check
  until: health_check.status == 200
  retries: 30
  delay: 10
```

### Capture and Use:

```yaml
- name: Get current version
  command: /opt/app/bin/app --version
  register: current_version
  changed_when: false

- name: Upgrade if needed
  include_tasks: upgrade.yml
  when: current_version.stdout is version(required_version, '<')
```

---

## 📝 مثال کامل

```yaml
---
- name: Register Examples
  hosts: all
  
  tasks:
    # 1. بررسی پیش‌نیازها
    - name: Check OS
      command: cat /etc/os-release
      register: os_info
      changed_when: false
    
    - name: Parse OS info
      set_fact:
        os_name: "{{ os_info.stdout | regex_search('ID=(.+)', '\\1') | first }}"
    
    # 2. بررسی سرویس
    - name: Check nginx status
      command: systemctl is-active nginx
      register: nginx_status
      ignore_errors: yes
      changed_when: false
    
    # 3. نصب در صورت نیاز
    - name: Install nginx
      apt:
        name: nginx
        state: present
      register: nginx_install
      when: nginx_status.rc != 0
    
    # 4. گرفتن تنظیمات
    - name: Get nginx config
      command: nginx -T
      register: nginx_config
      changed_when: false
      when: nginx_status.rc == 0 or nginx_install is changed
    
    # 5. بررسی پورت
    - name: Check if port 80 is open
      wait_for:
        port: 80
        timeout: 5
      register: port_check
      ignore_errors: yes
    
    # 6. گزارش نهایی
    - name: Summary
      debug:
        msg: |
          OS: {{ os_name }}
          Nginx was: {{ 'installed' if nginx_install is changed else 'already present' }}
          Nginx status: {{ 'running' if nginx_status.rc == 0 else 'not running' }}
          Port 80: {{ 'open' if port_check is success else 'closed' }}
```

---

## ⚠️ نکات مهم

1. **همیشه `ignore_errors`**: اگر ممکن است task fail شود و می‌خواهید ادامه دهید
2. **`changed_when: false`**: برای command‌های فقط‌خواندنی
3. **دسترسی امن**: از `| default()` برای فیلدهایی که ممکن است نباشند استفاده کنید
4. **در loop**: نتایج در `.results` ذخیره می‌شوند

---

## 📚 منابع

- [Ansible Register](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#registering-variables)
- [Return Values](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html)
