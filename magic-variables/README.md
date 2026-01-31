# 🔮 Ansible Magic Variables

> متغیرهای خاص و از پیش تعریف شده Ansible

---

## 🎯 Magic Variables چیست؟

Magic Variables متغیرهای خاصی هستند که Ansible به صورت خودکار تنظیم می‌کند و اطلاعات مفیدی درباره اجرا، سرورها و گروه‌ها ارائه می‌دهند.

---

## 📋 دسته‌بندی Magic Variables

### 1️⃣ اطلاعات Host فعلی

| متغیر | توضیح |
|-------|-------|
| `inventory_hostname` | نام سرور در inventory |
| `inventory_hostname_short` | نام کوتاه (بدون domain) |
| `ansible_host` | آدرس واقعی اتصال |
| `ansible_port` | پورت اتصال |
| `ansible_user` | کاربر اتصال |
| `ansible_connection` | نوع اتصال (ssh, local, ...) |

```yaml
- debug:
    msg: |
      inventory_hostname: {{ inventory_hostname }}
      short: {{ inventory_hostname_short }}
      host: {{ ansible_host | default(inventory_hostname) }}
```

### 2️⃣ اطلاعات گروه‌ها

| متغیر | توضیح |
|-------|-------|
| `groups` | دیکشنری تمام گروه‌ها و اعضایشان |
| `group_names` | لیست گروه‌هایی که سرور فعلی عضو آنهاست |

```yaml
- name: Show all groups
  debug:
    var: groups

- name: Show my groups
  debug:
    var: group_names

- name: List webservers
  debug:
    msg: "{{ groups['webservers'] }}"

- name: Am I a webserver?
  debug:
    msg: "Yes!"
  when: "'webservers' in group_names"

- name: Get first webserver
  debug:
    msg: "{{ groups['webservers'][0] }}"

- name: Get all hosts
  debug:
    msg: "{{ groups['all'] }}"
```

### 3️⃣ دسترسی به متغیرهای سایر Hosts

| متغیر | توضیح |
|-------|-------|
| `hostvars` | دیکشنری متغیرهای تمام سرورها |

```yaml
- name: Get variable from another host
  debug:
    msg: "DB IP is {{ hostvars['db.example.com']['ansible_host'] }}"

- name: Get fact from another host
  debug:
    msg: "Web1 IP: {{ hostvars['web1']['ansible_default_ipv4']['address'] }}"

- name: Loop through group and get IPs
  debug:
    msg: "{{ item }}: {{ hostvars[item]['ansible_default_ipv4']['address'] }}"
  loop: "{{ groups['webservers'] }}"
```

### 4️⃣ اطلاعات Play و Playbook

| متغیر | توضیح |
|-------|-------|
| `playbook_dir` | مسیر دایرکتوری playbook |
| `role_path` | مسیر role فعلی |
| `ansible_play_hosts` | لیست سرورهای در حال اجرا در play |
| `ansible_play_hosts_all` | همه سرورهای play (حتی failed) |
| `ansible_play_batch` | سرورهای batch فعلی (با serial) |
| `play_hosts` | (deprecated) معادل ansible_play_hosts |

```yaml
- debug:
    msg: |
      Playbook directory: {{ playbook_dir }}
      All play hosts: {{ ansible_play_hosts }}
      Current batch: {{ ansible_play_batch }}
```

### 5️⃣ اطلاعات اجرا

| متغیر | توضیح |
|-------|-------|
| `ansible_check_mode` | آیا در حالت --check هستیم؟ |
| `ansible_diff_mode` | آیا --diff فعال است؟ |
| `ansible_version` | نسخه Ansible |
| `ansible_forks` | تعداد forks |
| `ansible_run_tags` | تگ‌های اجرا شده |
| `ansible_skip_tags` | تگ‌های skip شده |

```yaml
- name: Skip in check mode
  command: /opt/destructive-script.sh
  when: not ansible_check_mode

- debug:
    msg: "Running Ansible {{ ansible_version.full }}"
```

### 6️⃣ اطلاعات Inventory

| متغیر | توضیح |
|-------|-------|
| `inventory_dir` | مسیر دایرکتوری inventory |
| `inventory_file` | مسیر فایل inventory |

```yaml
- debug:
    msg: |
      Inventory file: {{ inventory_file }}
      Inventory dir: {{ inventory_dir }}
```

---

## 🔧 کاربردهای رایج

### ساخت فایل hosts داینامیک:

```yaml
- name: Generate /etc/hosts
  template:
    src: hosts.j2
    dest: /etc/hosts
```

```jinja2
# templates/hosts.j2
127.0.0.1 localhost
127.0.1.1 {{ inventory_hostname }}

{% for host in groups['all'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ host }} {{ hostvars[host]['inventory_hostname_short'] }}
{% endfor %}
```

### پیکربندی کلاستر:

```yaml
- name: Configure cluster
  template:
    src: cluster.conf.j2
    dest: /etc/myapp/cluster.conf
```

```jinja2
# templates/cluster.conf.j2
# Cluster Configuration
cluster_name = my_cluster
node_name = {{ inventory_hostname }}

# All nodes
{% for host in groups['cluster'] %}
node.{{ loop.index }} = {{ hostvars[host]['ansible_default_ipv4']['address'] }}:{{ cluster_port }}
{% endfor %}

# Master nodes
{% for host in groups['masters'] %}
master.{{ loop.index }} = {{ hostvars[host]['ansible_default_ipv4']['address'] }}
{% endfor %}
```

### نمایش خلاصه:

```yaml
- name: Deployment summary
  debug:
    msg: |
      === Deployment Summary ===
      Total hosts: {{ ansible_play_hosts | length }}
      Hosts: {{ ansible_play_hosts | join(', ') }}
      Check mode: {{ ansible_check_mode }}
  run_once: true
```

### اجرای task روی آخرین سرور:

```yaml
- name: Final task on last host
  debug:
    msg: "This is the last host in the batch"
  when: inventory_hostname == ansible_play_hosts[-1]
  # یا
  when: inventory_hostname == ansible_play_batch[-1]
```

---

## 📊 جدول کامل Magic Variables

### Host Variables:

```yaml
inventory_hostname          # "web1.example.com"
inventory_hostname_short    # "web1"
ansible_host                # "192.168.1.10"
ansible_port                # 22
ansible_user                # "ansible"
ansible_become              # true/false
ansible_become_user         # "root"
ansible_connection          # "ssh"
ansible_python_interpreter  # "/usr/bin/python3"
```

### Group Variables:

```yaml
groups                      # {"all": [...], "webservers": [...]}
group_names                 # ["webservers", "production"]
```

### Host Access:

```yaml
hostvars                    # دسترسی به همه متغیرها
hostvars[host]              # متغیرهای یک host
hostvars[host].ansible_host # یک متغیر خاص
```

### Play Variables:

```yaml
ansible_play_hosts          # ["web1", "web2"]
ansible_play_hosts_all      # همه hosts (شامل failed)
ansible_play_batch          # hosts در batch فعلی
ansible_play_name           # نام play
```

### Run Variables:

```yaml
ansible_check_mode          # true/false
ansible_diff_mode           # true/false
ansible_verbosity           # 0-4
ansible_version             # {"full": "2.12.0", ...}
ansible_run_tags            # ["deploy", "config"]
ansible_skip_tags           # ["test"]
```

### Path Variables:

```yaml
playbook_dir                # "/home/user/playbooks"
role_path                   # مسیر role فعلی
inventory_dir               # مسیر inventory
inventory_file              # فایل inventory
```

### Special Variables:

```yaml
omit                        # برای skip کردن پارامتر
ansible_loop                # اطلاعات loop
ansible_loop.index          # شماره فعلی (از 1)
ansible_loop.index0         # شماره فعلی (از 0)
ansible_loop.first          # اولین آیتم؟
ansible_loop.last           # آخرین آیتم؟
ansible_loop.length         # تعداد کل
```

---

## 🔍 omit - Skip کردن پارامتر

```yaml
- name: Create user with optional password
  user:
    name: "{{ item.name }}"
    password: "{{ item.password | default(omit) }}"
    groups: "{{ item.groups | default(omit) }}"
  loop:
    - { name: alice, password: "hash...", groups: admin }
    - { name: bob }  # بدون password و groups
```

---

## 📝 مثال کامل

```yaml
---
- name: Magic Variables Demo
  hosts: all
  
  tasks:
    - name: Display host info
      debug:
        msg: |
          === Host Information ===
          Name: {{ inventory_hostname }}
          Short name: {{ inventory_hostname_short }}
          IP: {{ ansible_host | default('N/A') }}
          Groups: {{ group_names | join(', ') }}
    
    - name: Show all webservers
      debug:
        msg: "Webservers: {{ groups['webservers'] | default([]) | join(', ') }}"
      run_once: true
    
    - name: Get info from first webserver
      debug:
        msg: "First webserver IP: {{ hostvars[groups['webservers'][0]]['ansible_default_ipv4']['address'] }}"
      when: groups['webservers'] | length > 0
      run_once: true
    
    - name: Show play info
      debug:
        msg: |
          === Play Information ===
          Playbook dir: {{ playbook_dir }}
          Total hosts in play: {{ ansible_play_hosts | length }}
          Current batch: {{ ansible_play_batch | length }} hosts
          Check mode: {{ ansible_check_mode }}
          Diff mode: {{ ansible_diff_mode }}
      run_once: true
    
    - name: Generate cluster config
      copy:
        content: |
          # Auto-generated cluster config
          # Generated on {{ inventory_hostname }}
          
          [nodes]
          {% for host in groups['all'] %}
          {{ host }}={{ hostvars[host]['ansible_host'] | default(host) }}
          {% endfor %}
        dest: /tmp/cluster.conf
    
    - name: Final summary (only on last host)
      debug:
        msg: "All {{ ansible_play_hosts | length }} hosts processed!"
      when: inventory_hostname == ansible_play_hosts[-1]
```

---

## ⚠️ نکات مهم

1. **hostvars**: فقط بعد از gather_facts یا setup پر می‌شود
2. **groups**: نام گروه حساس به حروف بزرگ/کوچک است
3. **ansible_play_hosts**: شامل سرورهای failed نمی‌شود
4. **inventory_hostname**: ممکن است با ansible_host متفاوت باشد

---

## 📚 منابع

- [Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)
- [Magic Variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html#information-about-ansible-magic-variables)
