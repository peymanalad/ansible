# ⚙️ Ansible Configuration (ansible.cfg)

> تنظیمات و پیکربندی Ansible

---

## 🎯 ansible.cfg چیست؟

فایل `ansible.cfg` تنظیمات پیش‌فرض Ansible را مشخص می‌کند. این فایل رفتار Ansible را در موارد مختلف کنترل می‌کند.

---

## 📍 اولویت خواندن فایل تنظیمات

Ansible به این ترتیب فایل تنظیمات را جستجو می‌کند:

```
1. ANSIBLE_CONFIG (متغیر محیطی)
2. ./ansible.cfg (دایرکتوری فعلی) ✅ توصیه شده
3. ~/.ansible.cfg (home کاربر)
4. /etc/ansible/ansible.cfg (سیستمی)
```

> ⚠️ **نکته امنیتی**: اگر دایرکتوری فعلی writable by others باشد، Ansible فایل ansible.cfg آن را نادیده می‌گیرد.

---

## 📋 ساختار فایل ansible.cfg

```ini
[defaults]
# تنظیمات عمومی

[inventory]
# تنظیمات inventory

[privilege_escalation]
# تنظیمات sudo/become

[ssh_connection]
# تنظیمات اتصال SSH

[colors]
# رنگ‌های خروجی
```

---

## 🔧 تنظیمات مهم [defaults]

```ini
[defaults]
# مسیر inventory پیش‌فرض
inventory = ./inventory/hosts.yml

# کاربر پیش‌فرض برای اتصال
remote_user = ansible

# غیرفعال کردن چک host key (برای محیط تست)
host_key_checking = False

# تعداد اتصالات همزمان
forks = 10

# مسیر roles
roles_path = ./roles:/etc/ansible/roles

# مسیر collections
collections_path = ./collections

# زمان انتظار برای اتصال (ثانیه)
timeout = 30

# نمایش خلاصه تغییرات در diff
diff_always = True

# فعال کردن callback برای نمایش زمان اجرا
callback_whitelist = timer, profile_tasks

# فرمت خروجی
stdout_callback = yaml

# غیرفعال کردن هشدارهای deprecation
deprecation_warnings = False

# retry file
retry_files_enabled = False

# لاگ فایل
log_path = ./ansible.log

# interpreter python
interpreter_python = auto_silent

# پوشه‌های پلاگین
action_plugins = ./plugins/action
callback_plugins = ./plugins/callback
filter_plugins = ./plugins/filter
```

---

## 🔐 تنظیمات [privilege_escalation]

```ini
[privilege_escalation]
# فعال کردن sudo به صورت پیش‌فرض
become = True

# روش privilege escalation
become_method = sudo

# کاربر هدف sudo
become_user = root

# پرسیدن رمز sudo
become_ask_pass = False
```

---

## 🌐 تنظیمات [ssh_connection]

```ini
[ssh_connection]
# استفاده از pipelining برای سرعت بیشتر
pipelining = True

# آرگومان‌های اضافی SSH
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no

# تعداد تلاش مجدد
retries = 3

# روش انتقال فایل
transfer_method = smart

# فعال کردن SCP اگر SFTP کار نکرد
scp_if_ssh = smart
```

---

## 📦 تنظیمات [inventory]

```ini
[inventory]
# فعال کردن پلاگین‌های inventory
enable_plugins = host_list, script, auto, yaml, ini

# نادیده گرفتن extension‌های خاص
ignore_extensions = .pyc, .pyo, .swp, .bak, ~, .rpm, .md, .txt

# غیرفعال کردن parse شدن خودکار
unparsed_is_failed = True
```

---

## 🎨 تنظیمات [colors]

```ini
[colors]
highlight = white
verbose = blue
warn = bright purple
error = red
debug = dark gray
deprecate = purple
skip = cyan
unreachable = red
ok = green
changed = yellow
diff_add = green
diff_remove = red
diff_lines = cyan
```

---

## 📝 نمونه فایل کامل ansible.cfg

```ini
# ansible.cfg - تنظیمات پروژه
# ================================

[defaults]
inventory = ./inventory/hosts.yml
remote_user = ansible
host_key_checking = False
forks = 20
timeout = 30

# Paths
roles_path = ./roles
collections_path = ./collections

# Output
stdout_callback = yaml
callback_whitelist = timer, profile_tasks
diff_always = True

# Logging
log_path = ./logs/ansible.log

# Misc
retry_files_enabled = False
deprecation_warnings = False
interpreter_python = auto_silent

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=300s -o StrictHostKeyChecking=no
retries = 3

[inventory]
enable_plugins = yaml, ini, auto
```

---

## 🔍 دستورات مفید

```bash
# نمایش تنظیمات فعلی
ansible-config dump

# نمایش تنظیماتی که تغییر کرده‌اند
ansible-config dump --only-changed

# نمایش مسیر فایل تنظیمات
ansible-config view

# لیست تمام تنظیمات ممکن
ansible-config list

# نمایش یک تنظیم خاص
ansible-config dump | grep -i forks
```

---

## 🌍 متغیرهای محیطی

می‌توانید تنظیمات را با متغیرهای محیطی override کنید:

```bash
# تعیین فایل config
export ANSIBLE_CONFIG=/path/to/ansible.cfg

# تعیین inventory
export ANSIBLE_INVENTORY=./inventory/production

# غیرفعال کردن host key checking
export ANSIBLE_HOST_KEY_CHECKING=False

# تعداد forks
export ANSIBLE_FORKS=50

# فعال کردن become
export ANSIBLE_BECOME=True

# کاربر remote
export ANSIBLE_REMOTE_USER=deploy

# verbose mode
export ANSIBLE_VERBOSITY=2
```

---

## ⚡ تنظیمات بهینه‌سازی سرعت

```ini
[defaults]
# افزایش اتصالات همزمان
forks = 50

# جمع‌آوری facts فقط در صورت نیاز
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400

[ssh_connection]
# فعال کردن pipelining (مهم!)
pipelining = True

# نگه داشتن اتصال SSH
ssh_args = -o ControlMaster=auto -o ControlPersist=600s

# transfer سریع‌تر
transfer_method = piped
```

---

## ⚠️ نکات مهم

1. **pipelining**: برای کار کردن، باید `requiretty` در sudoers غیرفعال باشد
2. **host_key_checking**: فقط در محیط تست غیرفعال کنید
3. **forks**: بسته به منابع سیستم تنظیم کنید
4. **fact_caching**: برای پروژه‌های بزرگ بسیار مفید است

---

## 📚 منابع

- [Ansible Configuration Settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
- [ansible-config command](https://docs.ansible.com/ansible/latest/cli/ansible-config.html)
