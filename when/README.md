# ❓ Ansible Conditionals (when)

> اجرای شرطی task‌ها با استفاده از when

---

## 🎯 when چیست؟

`when` یک directive در Ansible است که به شما اجازه می‌دهد task‌ها را فقط در صورت برقرار بودن شرط خاصی اجرا کنید.

---

## 📋 سینتکس پایه

```yaml
tasks:
  - name: Task name
    module:
      param: value
    when: condition
```

> ⚠️ **نکته مهم**: در `when` نیازی به `{{ }}` نیست! Ansible خودش متغیرها را پردازش می‌کند.

---

## 🔧 شرط‌های ساده

### مقایسه مقادیر:

```yaml
tasks:
  - name: Only on Ubuntu
    apt:
      name: nginx
      state: present
    when: ansible_distribution == "Ubuntu"
  
  - name: Only on CentOS 7+
    yum:
      name: nginx
      state: present
    when: ansible_distribution == "CentOS" and ansible_distribution_major_version | int >= 7
  
  - name: Not on production
    debug:
      msg: "Development mode"
    when: env != "production"
```

### عملگرهای مقایسه:

| عملگر | توضیح | مثال |
|-------|-------|------|
| `==` | برابر | `var == "value"` |
| `!=` | نابرابر | `var != "value"` |
| `>` | بزرگتر | `count > 10` |
| `<` | کوچکتر | `count < 5` |
| `>=` | بزرگتر مساوی | `version >= 2` |
| `<=` | کوچکتر مساوی | `port <= 1024` |

---

## 🔀 شرط‌های ترکیبی

### AND:

```yaml
- name: Install on Ubuntu 20.04
  apt:
    name: nginx
  when: ansible_distribution == "Ubuntu" and ansible_distribution_version == "20.04"

# یا با لیست (implicit AND)
- name: Same as above
  apt:
    name: nginx
  when:
    - ansible_distribution == "Ubuntu"
    - ansible_distribution_version == "20.04"
```

### OR:

```yaml
- name: Install on Debian-based
  apt:
    name: nginx
  when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian"

# یا
- name: Same with in
  apt:
    name: nginx
  when: ansible_distribution in ["Ubuntu", "Debian"]
```

### NOT:

```yaml
- name: Skip on Windows
  debug:
    msg: "Linux task"
  when: not ansible_os_family == "Windows"

# یا
- name: Same
  debug:
    msg: "Linux task"
  when: ansible_os_family != "Windows"
```

### ترکیب پیچیده:

```yaml
- name: Complex condition
  debug:
    msg: "Matched!"
  when: >
    (ansible_distribution == "Ubuntu" and ansible_distribution_version >= "20.04")
    or
    (ansible_distribution == "CentOS" and ansible_distribution_major_version | int >= 8)
```

---

## ✅ بررسی متغیرها

### تعریف شده/نشده:

```yaml
- name: Only if variable is defined
  debug:
    msg: "{{ my_var }}"
  when: my_var is defined

- name: Only if variable is NOT defined
  debug:
    msg: "Using default"
  when: my_var is undefined
  
- name: Check if not none
  debug:
    msg: "{{ my_var }}"
  when: my_var is not none
```

### خالی/پر بودن:

```yaml
- name: If list is not empty
  debug:
    msg: "Has items"
  when: my_list | length > 0

# یا ساده‌تر
- name: If list has items
  debug:
    msg: "Has items"
  when: my_list  # True اگر خالی نباشد

- name: If string is not empty
  debug:
    msg: "{{ my_string }}"
  when: my_string | length > 0
```

---

## 📊 استفاده با Facts

```yaml
tasks:
  # بر اساس سیستم‌عامل
  - name: Debian family
    apt:
      name: nginx
    when: ansible_os_family == "Debian"
  
  - name: RedHat family
    yum:
      name: nginx
    when: ansible_os_family == "RedHat"
  
  # بر اساس حافظه
  - name: High memory config
    template:
      src: high-memory.conf.j2
      dest: /etc/app/config.conf
    when: ansible_memtotal_mb > 4096
  
  # بر اساس معماری
  - name: 64-bit only
    debug:
      msg: "64-bit system"
    when: ansible_architecture == "x86_64"
  
  # بر اساس مجازی بودن
  - name: Physical servers only
    debug:
      msg: "Physical machine"
    when: ansible_virtualization_type == "NA"
  
  - name: Virtual machines
    debug:
      msg: "VM: {{ ansible_virtualization_type }}"
    when: ansible_virtualization_role == "guest"
```

---

## 📝 استفاده با Register

```yaml
tasks:
  - name: Check if file exists
    stat:
      path: /etc/myapp/config.yml
    register: config_file
  
  - name: Create config if not exists
    template:
      src: config.yml.j2
      dest: /etc/myapp/config.yml
    when: not config_file.stat.exists
  
  - name: Read file content
    command: cat /etc/myapp/version
    register: version_output
    changed_when: false
  
  - name: Upgrade if old version
    include_tasks: upgrade.yml
    when: version_output.stdout | version('2.0', '<')
  
  - name: Check service status
    command: systemctl is-active myservice
    register: service_status
    ignore_errors: yes
    changed_when: false
  
  - name: Start if not running
    service:
      name: myservice
      state: started
    when: service_status.rc != 0
```

---

## 🔄 استفاده در Loop

```yaml
tasks:
  - name: Install only enabled packages
    apt:
      name: "{{ item.name }}"
      state: present
    loop:
      - { name: 'nginx', enabled: true }
      - { name: 'apache2', enabled: false }
      - { name: 'php', enabled: true }
    when: item.enabled
  
  - name: Create users with shell
    user:
      name: "{{ item.name }}"
      shell: "{{ item.shell }}"
    loop: "{{ users }}"
    when: 
      - item.shell is defined
      - item.active | default(true)
```

---

## 🧪 تست‌ها (Tests)

```yaml
tasks:
  # تست نوع داده
  - name: If is string
    debug:
      msg: "It's a string"
    when: my_var is string
  
  - name: If is number
    debug:
      msg: "It's a number"
    when: my_var is number
  
  - name: If is mapping (dict)
    debug:
      msg: "It's a dictionary"
    when: my_var is mapping
  
  - name: If is iterable
    debug:
      msg: "It's iterable"
    when: my_var is iterable
  
  # تست فایل
  - name: If file exists
    debug:
      msg: "File exists"
    when: "'/etc/passwd' is file"
  
  - name: If is directory
    debug:
      msg: "Is directory"
    when: "'/etc' is directory"
  
  # تست نسخه
  - name: Version check
    debug:
      msg: "Version is new enough"
    when: ansible_python_version is version('3.6', '>=')
  
  # تست regex
  - name: Hostname matches pattern
    debug:
      msg: "This is a web server"
    when: inventory_hostname is match("web.*")
  
  - name: Contains substring
    debug:
      msg: "Contains 'prod'"
    when: inventory_hostname is search("prod")
```

---

## 📦 Block با when

```yaml
- name: Setup webserver
  block:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Start nginx
      service:
        name: nginx
        state: started
    
    - name: Copy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
  when: "'webserver' in group_names"
```

---

## 🎯 استفاده با include/import

```yaml
tasks:
  # Include با شرط
  - name: Include Ubuntu tasks
    include_tasks: ubuntu.yml
    when: ansible_distribution == "Ubuntu"
  
  - name: Include CentOS tasks
    include_tasks: centos.yml
    when: ansible_distribution == "CentOS"
  
  # یا داینامیک
  - name: Include OS-specific tasks
    include_tasks: "{{ ansible_distribution | lower }}.yml"
    when: ansible_distribution in ['Ubuntu', 'CentOS', 'Debian']
```

---

## ⚠️ موارد خاص

### Boolean در when:

```yaml
tasks:
  # صحیح
  - name: If enabled
    debug:
      msg: "Enabled"
    when: feature_enabled | bool
  
  # یا
  - name: If enabled
    debug:
      msg: "Enabled"
    when: feature_enabled | default(false) | bool
  
  # غلط (string "false" هم true است!)
  - name: Wrong way
    debug:
      msg: "This might run unexpectedly"
    when: feature_enabled  # اگر "false" (string) باشد، اجرا می‌شود!
```

### Skip شدن در vs Failed:

```yaml
tasks:
  - name: This will be skipped
    debug:
      msg: "Hello"
    when: false
    # نتیجه: skipped (نه failed)
  
  - name: Force fail if condition
    fail:
      msg: "Condition not met!"
    when: required_var is undefined
```

---

## 📝 مثال کامل

```yaml
---
- name: Conditional Tasks Demo
  hosts: all
  vars:
    deploy_env: production
    enable_ssl: true
    min_memory_mb: 2048
  
  tasks:
    - name: Fail if not enough memory
      fail:
        msg: "Server needs at least {{ min_memory_mb }}MB RAM"
      when: ansible_memtotal_mb < min_memory_mb
    
    - name: Install on Debian-based systems
      apt:
        name: "{{ item }}"
        state: present
        update_cache: yes
      loop:
        - nginx
        - python3
      when: ansible_os_family == "Debian"
    
    - name: Install on RedHat-based systems
      yum:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - python3
      when: ansible_os_family == "RedHat"
    
    - name: Production-only tasks
      block:
        - name: Enable firewall
          ufw:
            state: enabled
        
        - name: Setup monitoring
          include_tasks: monitoring.yml
      when: deploy_env == "production"
    
    - name: SSL configuration
      block:
        - name: Install certbot
          apt:
            name: certbot
            state: present
        
        - name: Get certificate
          command: certbot --nginx -d {{ domain }}
          args:
            creates: /etc/letsencrypt/live/{{ domain }}
      when:
        - enable_ssl | bool
        - domain is defined
```

---

## 📚 منابع

- [Ansible Conditionals](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)
- [Ansible Tests](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html)
