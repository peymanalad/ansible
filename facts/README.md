# 📊 Ansible Facts

> جمع‌آوری و استفاده از اطلاعات سیستم در Ansible

---

## 🎯 Facts چیست؟

Facts اطلاعاتی هستند که Ansible به صورت خودکار از سرورهای هدف جمع‌آوری می‌کند. این اطلاعات شامل مشخصات سخت‌افزار، سیستم‌عامل، شبکه و... است.

---

## 🔍 نمایش Facts

```bash
# نمایش تمام facts یک سرور
ansible web1 -m setup

# با فیلتر
ansible web1 -m setup -a 'filter=ansible_hostname'
ansible web1 -m setup -a 'filter=ansible_distribution*'
ansible web1 -m setup -a 'filter=ansible_memory_mb'
ansible web1 -m setup -a 'filter=ansible_*_mb'

# خروجی به فایل JSON
ansible web1 -m setup --tree /tmp/facts
```

---

## 📋 Facts پرکاربرد

### اطلاعات سیستم:

| Fact | توضیح | مثال |
|------|-------|------|
| `ansible_hostname` | نام سرور | `web1` |
| `ansible_fqdn` | نام کامل | `web1.example.com` |
| `ansible_os_family` | خانواده OS | `Debian`, `RedHat` |
| `ansible_distribution` | توزیع | `Ubuntu`, `CentOS` |
| `ansible_distribution_version` | نسخه توزیع | `20.04` |
| `ansible_distribution_major_version` | نسخه اصلی | `20` |
| `ansible_kernel` | نسخه کرنل | `5.4.0-42-generic` |
| `ansible_architecture` | معماری | `x86_64` |

### اطلاعات سخت‌افزار:

| Fact | توضیح |
|------|-------|
| `ansible_processor` | لیست پردازنده‌ها |
| `ansible_processor_cores` | تعداد هسته |
| `ansible_processor_vcpus` | تعداد CPU مجازی |
| `ansible_memtotal_mb` | کل حافظه (MB) |
| `ansible_memfree_mb` | حافظه آزاد |
| `ansible_swaptotal_mb` | کل swap |

### اطلاعات شبکه:

| Fact | توضیح |
|------|-------|
| `ansible_default_ipv4.address` | IP پیش‌فرض |
| `ansible_default_ipv4.gateway` | Gateway |
| `ansible_all_ipv4_addresses` | تمام IP‌ها |
| `ansible_interfaces` | لیست interface‌ها |
| `ansible_eth0.ipv4.address` | IP یک interface خاص |

### اطلاعات دیسک:

| Fact | توضیح |
|------|-------|
| `ansible_devices` | لیست دیسک‌ها |
| `ansible_mounts` | پارتیشن‌های mount شده |

---

## 📝 استفاده از Facts در Playbook

```yaml
---
- name: Using Facts
  hosts: all
  
  tasks:
    - name: Show hostname
      debug:
        msg: "Hostname is {{ ansible_hostname }}"
    
    - name: Show OS info
      debug:
        msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
    
    - name: Show IP
      debug:
        msg: "IP: {{ ansible_default_ipv4.address }}"
    
    - name: Show memory
      debug:
        msg: "Memory: {{ ansible_memtotal_mb }} MB"
    
    # استفاده در شرط
    - name: Install package on Ubuntu
      apt:
        name: nginx
        state: present
      when: ansible_distribution == "Ubuntu"
    
    - name: Install package on CentOS
      yum:
        name: nginx
        state: present
      when: ansible_distribution == "CentOS"
    
    # استفاده در template
    - name: Copy config
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
```

### در Template (Jinja2):

```jinja2
# app.conf.j2
# Server: {{ ansible_hostname }}
# OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
# IP: {{ ansible_default_ipv4.address }}
# CPUs: {{ ansible_processor_vcpus }}
# Memory: {{ ansible_memtotal_mb }} MB

worker_processes {{ ansible_processor_vcpus }};

{% if ansible_memtotal_mb > 4096 %}
cache_size = 1024m
{% else %}
cache_size = 256m
{% endif %}
```

---

## ⚙️ کنترل جمع‌آوری Facts

### غیرفعال کردن:

```yaml
---
- name: Play without facts
  hosts: all
  gather_facts: no    # سریع‌تر می‌شود
  
  tasks:
    - name: Simple task
      ping:
```

### جمع‌آوری دستی:

```yaml
---
- name: Gather facts manually
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Do something first
      command: echo "hello"
    
    - name: Now gather facts
      setup:
    
    - name: Use facts
      debug:
        msg: "{{ ansible_hostname }}"
```

### جمع‌آوری بخشی از Facts:

```yaml
---
- name: Gather minimal facts
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Gather only network facts
      setup:
        gather_subset:
          - network
    
    - name: Gather hardware and virtual
      setup:
        gather_subset:
          - hardware
          - virtual
```

**Subset های موجود:**
- `all` - همه (پیش‌فرض)
- `min` - حداقل
- `hardware` - سخت‌افزار
- `network` - شبکه
- `virtual` - مجازی‌سازی
- `ohai` - facts از Ohai
- `facter` - facts از Facter

---

## 🏭 Custom Facts (Local Facts)

می‌توانید facts سفارشی روی سرورها ایجاد کنید.

### روی سرور هدف:

فایل‌ها در `/etc/ansible/facts.d/` با پسوند `.fact`:

```bash
# /etc/ansible/facts.d/custom.fact
[application]
name = myapp
version = 1.2.3
env = production

[database]
host = db.example.com
port = 5432
```

یا به صورت JSON:

```json
# /etc/ansible/facts.d/app.fact
{
  "app_name": "myapp",
  "app_version": "1.2.3",
  "features": ["api", "web", "worker"]
}
```

یا اسکریپت اجرایی:

```bash
#!/bin/bash
# /etc/ansible/facts.d/dynamic.fact
# باید خروجی JSON بدهد

cat << EOF
{
  "uptime_days": $(awk '{print int($1/86400)}' /proc/uptime),
  "disk_usage": "$(df -h / | tail -1 | awk '{print $5}')"
}
EOF
```

### استفاده در Playbook:

```yaml
---
- name: Use custom facts
  hosts: all
  
  tasks:
    - name: Show custom fact
      debug:
        msg: "App: {{ ansible_local.custom.application.name }}"
    
    - name: Show all local facts
      debug:
        var: ansible_local
```

---

## 💾 Facts Caching

برای پروژه‌های بزرگ، cache کردن facts سرعت را افزایش می‌دهد.

### در ansible.cfg:

```ini
[defaults]
# فعال کردن smart gathering
gathering = smart

# نوع cache
fact_caching = jsonfile

# مسیر cache
fact_caching_connection = /tmp/ansible_facts_cache

# مدت اعتبار (ثانیه) - 24 ساعت
fact_caching_timeout = 86400
```

### انواع Cache:

```ini
# فایل JSON
fact_caching = jsonfile
fact_caching_connection = /path/to/cache

# Redis
fact_caching = redis
fact_caching_connection = localhost:6379:0

# Memcached
fact_caching = memcached
fact_caching_connection = localhost:11211

# YAML
fact_caching = yaml
fact_caching_connection = /path/to/cache
```

---

## 🔧 Set_fact برای ایجاد Facts موقت

```yaml
---
- name: Using set_fact
  hosts: all
  
  tasks:
    - name: Set a fact
      set_fact:
        my_custom_var: "Hello World"
        server_type: "{{ 'web' if 'web' in inventory_hostname else 'db' }}"
    
    - name: Calculate and set
      set_fact:
        total_memory_gb: "{{ (ansible_memtotal_mb / 1024) | round(2) }}"
    
    - name: Set complex fact
      set_fact:
        server_info:
          hostname: "{{ ansible_hostname }}"
          ip: "{{ ansible_default_ipv4.address }}"
          os: "{{ ansible_distribution }}"
    
    - name: Use the facts
      debug:
        msg: "{{ my_custom_var }} on {{ server_info.hostname }}"
    
    # Fact که در سایر play‌ها هم موجود باشد
    - name: Set cacheable fact
      set_fact:
        permanent_fact: "This will be cached"
        cacheable: yes
```

---

## 📊 مقایسه روش‌های تعریف متغیر

| روش | Scope | Persist |
|-----|-------|---------|
| `vars:` در playbook | همان play | خیر |
| `set_fact` | همان host در همه play‌ها | خیر (مگر cacheable) |
| Custom facts (`/etc/ansible/facts.d/`) | دائمی روی سرور | بله |
| Facts cache | بین اجراها | بله (تا timeout) |

---

## 📝 مثال کامل

```yaml
---
- name: Facts Demo
  hosts: all
  
  tasks:
    - name: Gather all facts
      setup:
    
    - name: Display system summary
      debug:
        msg: |
          === System Info ===
          Hostname: {{ ansible_hostname }}
          FQDN: {{ ansible_fqdn }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Kernel: {{ ansible_kernel }}
          Architecture: {{ ansible_architecture }}
          
          === Hardware ===
          CPUs: {{ ansible_processor_vcpus }}
          Memory: {{ ansible_memtotal_mb }} MB
          
          === Network ===
          IP: {{ ansible_default_ipv4.address }}
          Gateway: {{ ansible_default_ipv4.gateway | default('N/A') }}
    
    - name: Set derived facts
      set_fact:
        is_low_memory: "{{ ansible_memtotal_mb < 2048 }}"
        is_ubuntu: "{{ ansible_distribution == 'Ubuntu' }}"
    
    - name: Conditional task
      debug:
        msg: "This server has low memory!"
      when: is_low_memory | bool
```

---

## 📚 منابع

- [Ansible Facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)
- [setup module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html)
