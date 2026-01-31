# 🔄 Ansible Loops

> تکرار task‌ها با استفاده از loop

---

## 🎯 Loop چیست؟

Loop به شما اجازه می‌دهد یک task را چندین بار با مقادیر مختلف اجرا کنید، بدون اینکه task را تکرار کنید.

---

## 📋 سینتکس‌های Loop

### روش جدید (توصیه شده): `loop`

```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - vim
    - htop
```

### روش قدیمی: `with_*`

```yaml
# معادل loop
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - nginx
    - vim
    - htop
```

---

## 🔧 انواع Loop

### 1️⃣ Loop ساده (لیست):

```yaml
tasks:
  - name: Create users
    user:
      name: "{{ item }}"
      state: present
    loop:
      - alice
      - bob
      - charlie
  
  # با متغیر
  - name: Install packages
    apt:
      name: "{{ item }}"
      state: present
    loop: "{{ packages }}"
```

### 2️⃣ Loop با Dictionary:

```yaml
tasks:
  - name: Create users with groups
    user:
      name: "{{ item.name }}"
      groups: "{{ item.groups }}"
      shell: "{{ item.shell | default('/bin/bash') }}"
    loop:
      - { name: 'alice', groups: 'admin', shell: '/bin/zsh' }
      - { name: 'bob', groups: 'developers' }
      - { name: 'charlie', groups: 'users' }
  
  # یا با YAML بهتر
  - name: Create users
    user:
      name: "{{ item.name }}"
      groups: "{{ item.groups }}"
    loop:
      - name: alice
        groups: admin
      - name: bob
        groups: developers
```

### 3️⃣ Loop روی Dictionary (dict2items):

```yaml
vars:
  users:
    alice: admin
    bob: developer
    charlie: user

tasks:
  - name: Show users and roles
    debug:
      msg: "{{ item.key }} is {{ item.value }}"
    loop: "{{ users | dict2items }}"
  
  # خروجی:
  # alice is admin
  # bob is developer
  # charlie is user
```

### 4️⃣ Loop تو در تو (Nested):

```yaml
tasks:
  - name: Create directories for each user
    file:
      path: "/home/{{ item.0 }}/{{ item.1 }}"
      state: directory
    loop: "{{ users | product(directories) | list }}"
    vars:
      users:
        - alice
        - bob
      directories:
        - documents
        - downloads
        - pictures
```

### 5️⃣ Loop با Index:

```yaml
tasks:
  - name: Show index
    debug:
      msg: "{{ index }}: {{ item }}"
    loop:
      - apple
      - banana
      - cherry
    loop_control:
      index_var: index
  
  # با شروع از 1
  - name: Show numbered list
    debug:
      msg: "{{ index + 1 }}. {{ item }}"
    loop:
      - apple
      - banana
    loop_control:
      index_var: index
```

---

## 🎮 loop_control

کنترل رفتار loop:

```yaml
tasks:
  - name: Install packages with custom label
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - mysql-server
      - php-fpm
    loop_control:
      label: "Installing {{ item }}"  # نمایش در خروجی
      pause: 2                         # وقفه بین هر آیتم (ثانیه)
      index_var: my_index              # متغیر index
      loop_var: package                # تغییر نام item
  
  # با loop_var
  - name: Create users
    user:
      name: "{{ user.name }}"
    loop: "{{ users }}"
    loop_control:
      loop_var: user   # به جای item از user استفاده کن
```

### مخفی کردن خروجی طولانی:

```yaml
- name: Create many files
  copy:
    content: "{{ item.content }}"
    dest: "{{ item.path }}"
  loop: "{{ large_data_list }}"
  loop_control:
    label: "{{ item.path }}"  # فقط path را نشان بده، نه کل محتوا
```

---

## 📊 with_* (روش‌های قدیمی)

### with_items:

```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
  with_items:
    - nginx
    - vim
```

### with_dict:

```yaml
- name: Set sysctl
  sysctl:
    name: "{{ item.key }}"
    value: "{{ item.value }}"
  with_dict:
    net.ipv4.ip_forward: 1
    net.core.somaxconn: 65535
```

### with_file:

```yaml
- name: Copy files
  copy:
    content: "{{ item }}"
    dest: /tmp/file
  with_file:
    - /path/to/file1.txt
    - /path/to/file2.txt
```

### with_fileglob:

```yaml
- name: Copy all configs
  copy:
    src: "{{ item }}"
    dest: /etc/myapp/
  with_fileglob:
    - files/*.conf
```

### with_lines:

```yaml
- name: Process each line
  debug:
    msg: "{{ item }}"
  with_lines: cat /etc/passwd
```

### with_sequence:

```yaml
- name: Create numbered files
  file:
    path: "/tmp/file{{ item }}"
    state: touch
  with_sequence: start=1 end=5

- name: With format
  debug:
    msg: "server{{ item }}"
  with_sequence: start=1 end=3 format=%02d
  # server01, server02, server03
```

### with_random_choice:

```yaml
- name: Random server
  debug:
    msg: "Selected: {{ item }}"
  with_random_choice:
    - server1
    - server2
    - server3
```

### with_together (zip):

```yaml
- name: Create users with passwords
  user:
    name: "{{ item.0 }}"
    password: "{{ item.1 | password_hash('sha512') }}"
  with_together:
    - ['alice', 'bob', 'charlie']
    - ['pass1', 'pass2', 'pass3']
```

### with_subelements:

```yaml
vars:
  users:
    - name: alice
      keys:
        - ssh-rsa AAA...
        - ssh-rsa BBB...
    - name: bob
      keys:
        - ssh-rsa CCC...

tasks:
  - name: Add SSH keys
    authorized_key:
      user: "{{ item.0.name }}"
      key: "{{ item.1 }}"
    with_subelements:
      - "{{ users }}"
      - keys
```

---

## 🔄 تبدیل with_* به loop

| with_* | معادل loop |
|--------|-----------|
| `with_items` | `loop` |
| `with_list` | `loop` |
| `with_dict` | `loop: "{{ dict \| dict2items }}"` |
| `with_sequence` | `loop: "{{ range(1, 6) \| list }}"` |
| `with_together` | `loop: "{{ list1 \| zip(list2) \| list }}"` |
| `with_subelements` | `loop: "{{ users \| subelements('keys') }}"` |
| `with_nested` | `loop: "{{ list1 \| product(list2) \| list }}"` |
| `with_fileglob` | `loop: "{{ query('fileglob', 'files/*.conf') }}"` |

---

## 🔍 Lookup Plugins با Loop

```yaml
tasks:
  - name: Read files
    debug:
      msg: "{{ item }}"
    loop: "{{ lookup('fileglob', 'files/*.txt', wantlist=True) }}"
  
  - name: Read from file lines
    debug:
      msg: "{{ item }}"
    loop: "{{ lookup('file', '/etc/hosts').splitlines() }}"
  
  - name: Use query (always returns list)
    debug:
      msg: "{{ item }}"
    loop: "{{ query('file', 'file1.txt', 'file2.txt') }}"
```

---

## 📝 Register با Loop

```yaml
tasks:
  - name: Check services
    command: systemctl is-active {{ item }}
    loop:
      - nginx
      - mysql
      - redis
    register: service_status
    ignore_errors: yes
  
  - name: Show results
    debug:
      msg: "{{ item.item }}: {{ 'running' if item.rc == 0 else 'stopped' }}"
    loop: "{{ service_status.results }}"
  
  # فیلتر کردن نتایج
  - name: Show failed services
    debug:
      msg: "{{ item.item }} is not running"
    loop: "{{ service_status.results }}"
    when: item.rc != 0
```

---

## ⚡ بهینه‌سازی Loop

### استفاده از پارامتر لیست ماژول:

```yaml
# ❌ کند - یک بار برای هر پکیج
- name: Install packages (slow)
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - vim
    - htop

# ✅ سریع - یکبار برای همه
- name: Install packages (fast)
  apt:
    name:
      - nginx
      - vim
      - htop
    state: present

# یا با متغیر
- name: Install packages
  apt:
    name: "{{ packages }}"
    state: present
```

---

## 📝 مثال‌های کاربردی

### ایجاد کاربران با SSH Key:

```yaml
vars:
  users:
    - name: alice
      groups: ['admin', 'docker']
      ssh_key: "ssh-rsa AAAA..."
    - name: bob
      groups: ['developers']
      ssh_key: "ssh-rsa BBBB..."

tasks:
  - name: Create users
    user:
      name: "{{ item.name }}"
      groups: "{{ item.groups }}"
      append: yes
    loop: "{{ users }}"
  
  - name: Add SSH keys
    authorized_key:
      user: "{{ item.name }}"
      key: "{{ item.ssh_key }}"
    loop: "{{ users }}"
```

### کپی چند فایل با تنظیمات مختلف:

```yaml
- name: Copy config files
  copy:
    src: "{{ item.src }}"
    dest: "{{ item.dest }}"
    mode: "{{ item.mode | default('0644') }}"
    owner: "{{ item.owner | default('root') }}"
  loop:
    - src: nginx.conf
      dest: /etc/nginx/nginx.conf
    - src: app.conf
      dest: /etc/myapp/app.conf
      mode: '0600'
      owner: app
```

### ایجاد vhosts:

```yaml
- name: Create vhosts
  template:
    src: vhost.conf.j2
    dest: "/etc/nginx/sites-available/{{ item.name }}"
  loop: "{{ vhosts }}"
  notify: Reload nginx

- name: Enable vhosts
  file:
    src: "/etc/nginx/sites-available/{{ item.name }}"
    dest: "/etc/nginx/sites-enabled/{{ item.name }}"
    state: link
  loop: "{{ vhosts }}"
  notify: Reload nginx
```

---

## ⚠️ نکات مهم

1. **loop_var**: در loop‌های تو در تو از نام‌های مختلف استفاده کنید
2. **label**: برای داده‌های بزرگ، label را تنظیم کنید
3. **بهینه‌سازی**: اگر ماژول لیست قبول می‌کند، از loop استفاده نکنید
4. **register**: نتایج در `.results` ذخیره می‌شوند

---

## 📚 منابع

- [Ansible Loops](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)
- [Migrating from with_X to loop](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#migrating-from-with-x-to-loop)
