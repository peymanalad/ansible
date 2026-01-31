# 🔐 Ansible Vault

> رمزنگاری اطلاعات حساس در Ansible

---

## 🎯 Vault چیست؟

Ansible Vault ابزاری برای رمزنگاری فایل‌ها و متغیرهای حساس است. با استفاده از Vault می‌توانید رمزها، کلیدها و سایر اطلاعات محرمانه را به صورت امن در repository ذخیره کنید.

---

## 📋 دستورات اصلی

```bash
# ایجاد فایل رمزنگاری شده جدید
ansible-vault create secrets.yml

# ویرایش فایل رمزنگاری شده
ansible-vault edit secrets.yml

# مشاهده محتوای فایل رمزنگاری شده
ansible-vault view secrets.yml

# رمزنگاری فایل موجود
ansible-vault encrypt vars.yml

# رمزگشایی فایل
ansible-vault decrypt secrets.yml

# تغییر رمز فایل
ansible-vault rekey secrets.yml

# رمزنگاری یک رشته
ansible-vault encrypt_string 'my_secret_password' --name 'db_password'
```

---

## 🔧 روش‌های استفاده

### 1️⃣ رمزنگاری کل فایل

```bash
# ایجاد فایل جدید
ansible-vault create group_vars/production/vault.yml
```

```yaml
# محتوای vault.yml (قبل از رمزنگاری)
---
vault_db_password: "SuperSecretPassword123!"
vault_api_key: "sk-xxxxxxxxxxxxxxxxxxxxx"
vault_ssl_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvQIBADANBgkqhkiG9w0BAQE...
  -----END PRIVATE KEY-----
```

فایل رمزنگاری شده:
```
$ANSIBLE_VAULT;1.1;AES256
36323436313538386336323663313563613730636562313836303861653865363065373738663930
6231383466623964323938646337616166623336653436610a653664626434386264323962666336
...
```

### 2️⃣ رمزنگاری متغیر تکی (Inline)

```bash
# رمزنگاری یک رشته
ansible-vault encrypt_string 'MySecretPassword' --name 'db_password'

# خروجی:
# db_password: !vault |
#   $ANSIBLE_VAULT;1.1;AES256
#   3431373...
```

استفاده در vars:
```yaml
# vars/main.yml
---
db_user: myapp
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  34313739623364373135393232353531323932393261636138356134313264
  6466353762666334653031363732323237376362386231620a31383133653937
  3534...
db_host: db.example.com
```

---

## 🔑 مدیریت رمز Vault

### روش 1: پرسیدن رمز

```bash
ansible-playbook site.yml --ask-vault-pass
```

### روش 2: فایل رمز

```bash
# ذخیره رمز در فایل
echo "MyVaultPassword" > .vault_pass
chmod 600 .vault_pass

# استفاده
ansible-playbook site.yml --vault-password-file .vault_pass

# یا در ansible.cfg
[defaults]
vault_password_file = .vault_pass
```

> ⚠️ **مهم**: فایل `.vault_pass` را به `.gitignore` اضافه کنید!

### روش 3: اسکریپت رمز

```bash
#!/bin/bash
# .vault_pass.sh
# گرفتن رمز از password manager

pass show ansible/vault-password
# یا
security find-generic-password -a ansible -s vault -w
# یا
vault read -field=password secret/ansible/vault
```

```bash
chmod +x .vault_pass.sh
ansible-playbook site.yml --vault-password-file .vault_pass.sh
```

### روش 4: متغیر محیطی

```bash
export ANSIBLE_VAULT_PASSWORD_FILE=.vault_pass
ansible-playbook site.yml
```

---

## 🏷️ Vault IDs (چند رمز)

برای استفاده از چند رمز مختلف:

```bash
# رمزنگاری با vault-id
ansible-vault encrypt --vault-id prod@prompt secrets_prod.yml
ansible-vault encrypt --vault-id dev@.vault_pass_dev secrets_dev.yml

# اجرا با چند vault-id
ansible-playbook site.yml \
  --vault-id prod@prompt \
  --vault-id dev@.vault_pass_dev
```

```yaml
# فایل با vault-id خاص
db_password: !vault |
  $ANSIBLE_VAULT;1.2;AES256;prod
  ...
```

---

## 📂 ساختار پیشنهادی

```
project/
├── ansible.cfg
├── .gitignore              # شامل .vault_pass
├── .vault_pass             # فایل رمز (در git نیست)
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all.yml           # متغیرهای عادی
│   │   │   └── all/
│   │   │       ├── vars.yml      # متغیرهای عادی
│   │   │       └── vault.yml     # متغیرهای رمزنگاری شده
│   │   └── host_vars/
│   └── staging/
├── playbooks/
└── roles/
```

### الگوی نام‌گذاری متغیرها:

```yaml
# group_vars/all/vars.yml
---
db_password: "{{ vault_db_password }}"
api_key: "{{ vault_api_key }}"
ssl_certificate: "{{ vault_ssl_certificate }}"

# group_vars/all/vault.yml (encrypted)
---
vault_db_password: "ActualSecretPassword"
vault_api_key: "sk-xxxxxxxxxxxxxxxx"
vault_ssl_certificate: |
  -----BEGIN CERTIFICATE-----
  ...
  -----END CERTIFICATE-----
```

---

## 📝 استفاده در Playbook

```yaml
---
- name: Deploy with secrets
  hosts: webservers
  vars_files:
    - vars/common.yml
    - vars/secrets.yml  # فایل رمزنگاری شده
  
  tasks:
    - name: Configure database
      template:
        src: database.yml.j2
        dest: /etc/myapp/database.yml
        mode: '0600'
      # template از متغیرهای رمزگشایی شده استفاده می‌کند
    
    - name: Set API key
      lineinfile:
        path: /etc/myapp/config
        line: "API_KEY={{ api_key }}"
        mode: '0600'
```

### در Template:

```jinja2
# templates/database.yml.j2
database:
  host: {{ db_host }}
  port: {{ db_port }}
  username: {{ db_user }}
  password: {{ db_password }}  # از vault می‌آید
```

---

## 🔄 عملیات Vault

### رمزنگاری چند فایل:

```bash
ansible-vault encrypt file1.yml file2.yml file3.yml
```

### رمزگشایی موقت:

```bash
# رمزگشایی، ویرایش دستی، رمزنگاری مجدد
ansible-vault decrypt secrets.yml
vim secrets.yml
ansible-vault encrypt secrets.yml
```

### تغییر رمز:

```bash
# با پرسیدن رمز قدیم و جدید
ansible-vault rekey secrets.yml

# با فایل رمز
ansible-vault rekey secrets.yml \
  --vault-password-file old_pass \
  --new-vault-password-file new_pass
```

### مشاهده بدون ویرایش:

```bash
ansible-vault view secrets.yml --vault-password-file .vault_pass
```

---

## 🛡️ Best Practices

### 1. جداسازی متغیرها:

```yaml
# vars.yml (unencrypted)
---
app_name: myapp
app_port: 8080
db_host: db.example.com
db_user: myapp
db_password: "{{ vault_db_password }}"

# vault.yml (encrypted)
---
vault_db_password: "SuperSecret123!"
```

### 2. استفاده از no_log:

```yaml
- name: Configure with secret
  template:
    src: config.j2
    dest: /etc/app/config
  vars:
    secret_key: "{{ vault_secret_key }}"
  no_log: true  # خروجی را مخفی کن
```

### 3. رمزنگاری فقط موارد حساس:

```bash
# فقط رشته حساس را رمز کن
ansible-vault encrypt_string 'secret_value' --name 'my_secret' >> vars.yml
```

---

## 🔍 اشکال‌زدایی

### بررسی رمزگشایی:

```bash
# نمایش متغیر رمزگشایی شده
ansible localhost -m debug -a "var=vault_db_password" \
  -e "@secrets.yml" \
  --vault-password-file .vault_pass
```

### بررسی سینتکس:

```bash
ansible-playbook site.yml --syntax-check --vault-password-file .vault_pass
```

---

## 📊 جدول دستورات

| دستور | توضیح |
|-------|-------|
| `create` | ایجاد فایل رمزنگاری شده جدید |
| `edit` | ویرایش فایل رمزنگاری شده |
| `view` | مشاهده محتوا |
| `encrypt` | رمزنگاری فایل موجود |
| `decrypt` | رمزگشایی فایل |
| `rekey` | تغییر رمز |
| `encrypt_string` | رمزنگاری یک رشته |

---

## 📝 مثال کامل

### ساختار پروژه:

```
project/
├── ansible.cfg
├── .vault_pass
├── .gitignore
├── inventory/
│   └── production/
│       └── group_vars/
│           └── all/
│               ├── vars.yml
│               └── vault.yml
├── playbooks/
│   └── deploy.yml
└── templates/
    └── app.conf.j2
```

### ansible.cfg:

```ini
[defaults]
inventory = inventory/production
vault_password_file = .vault_pass
```

### .gitignore:

```
.vault_pass
*.retry
```

### vars.yml:

```yaml
---
app_name: myapp
app_port: 8080

db_host: db.example.com
db_port: 5432
db_name: myapp_production
db_user: myapp
db_password: "{{ vault_db_password }}"

api_url: https://api.example.com
api_key: "{{ vault_api_key }}"
```

### vault.yml (encrypted):

```yaml
---
vault_db_password: "ProductionDBPassword123!"
vault_api_key: "sk-prod-xxxxxxxxxxxxxxxx"
vault_ssl_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvQIBADANBgkqhkiG9w0BAQEFAASC...
  -----END PRIVATE KEY-----
```

### deploy.yml:

```yaml
---
- name: Deploy Application
  hosts: webservers
  become: yes
  
  tasks:
    - name: Create config file
      template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf
        owner: myapp
        group: myapp
        mode: '0600'
      no_log: true
      notify: Restart app
    
    - name: Set database password in environment
      lineinfile:
        path: /etc/myapp/.env
        line: "DB_PASSWORD={{ db_password }}"
        mode: '0600'
        create: yes
      no_log: true
  
  handlers:
    - name: Restart app
      service:
        name: myapp
        state: restarted
```

### app.conf.j2:

```jinja2
# Application Configuration
# Generated by Ansible

[database]
host = {{ db_host }}
port = {{ db_port }}
name = {{ db_name }}
user = {{ db_user }}
password = {{ db_password }}

[api]
url = {{ api_url }}
key = {{ api_key }}
```

### اجرا:

```bash
ansible-playbook playbooks/deploy.yml
# رمز از .vault_pass خوانده می‌شود
```

---

## ⚠️ نکات امنیتی

1. **فایل رمز**: هرگز در git commit نکنید
2. **no_log**: برای task‌های حساس استفاده کنید
3. **دسترسی فایل**: `chmod 600` برای فایل‌های حساس
4. **Vault ID**: برای محیط‌های مختلف رمزهای مختلف
5. **Password Manager**: رمز Vault را در password manager نگهداری کنید

---

## 📚 منابع

- [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#best-practices-for-variables-and-vaults)
