# 🔌 Ansible Plugins

> توسعه و استفاده از پلاگین‌ها در Ansible

---

## 🎯 Plugin چیست؟

پلاگین‌ها کدهایی هستند که قابلیت‌های هسته Ansible را گسترش می‌دهند. برخلاف ماژول‌ها که روی سرور هدف اجرا می‌شوند، پلاگین‌ها روی سرور کنترلر اجرا می‌شوند.

---

## 📋 انواع پلاگین‌ها

| نوع | توضیح | مثال |
|-----|-------|------|
| **Action** | کنترل اجرای ماژول | `normal`, `debug` |
| **Become** | روش‌های privilege escalation | `sudo`, `su`, `doas` |
| **Cache** | ذخیره facts | `jsonfile`, `redis`, `memory` |
| **Callback** | هوک برای رویدادها | `timer`, `profile_tasks`, `json` |
| **Connection** | روش‌های اتصال | `ssh`, `local`, `docker` |
| **Filter** | فیلترهای Jinja2 | `to_json`, `regex_search` |
| **Inventory** | منابع inventory | `yaml`, `script`, `aws_ec2` |
| **Lookup** | دسترسی به داده‌های خارجی | `file`, `env`, `password` |
| **Strategy** | استراتژی اجرا | `linear`, `free`, `debug` |
| **Test** | تست‌های Jinja2 | `defined`, `file`, `match` |
| **Vars** | منابع متغیرها | `host_group_vars` |

---

## 🔍 Lookup Plugins

دسترسی به داده‌های خارجی از کنترلر.

### استفاده:

```yaml
vars:
  # خواندن فایل
  file_content: "{{ lookup('file', '/etc/passwd') }}"
  
  # متغیر محیطی
  home_dir: "{{ lookup('env', 'HOME') }}"
  
  # تولید رمز
  random_pass: "{{ lookup('password', '/tmp/pass length=16') }}"
  
  # خواندن از pipe
  git_hash: "{{ lookup('pipe', 'git rev-parse HEAD') }}"
  
  # Template
  rendered: "{{ lookup('template', 'config.j2') }}"
  
  # URL
  api_response: "{{ lookup('url', 'https://api.example.com/data') }}"
  
  # اول موجود
  config: "{{ lookup('first_found', ['prod.yml', 'default.yml']) }}"

tasks:
  # با query (همیشه لیست برمی‌گرداند)
  - name: Read multiple files
    debug:
      msg: "{{ query('file', 'file1.txt', 'file2.txt') }}"
  
  # با loop
  - name: Process files
    debug:
      msg: "{{ item }}"
    loop: "{{ query('fileglob', 'files/*.conf') }}"
```

### Lookup‌های پرکاربرد:

```yaml
# file - خواندن فایل
content: "{{ lookup('file', 'myfile.txt') }}"

# env - متغیر محیطی
user: "{{ lookup('env', 'USER') }}"

# password - تولید/خواندن رمز
pass: "{{ lookup('password', '/tmp/pass length=20 chars=ascii_letters,digits') }}"

# pipe - اجرای command
date: "{{ lookup('pipe', 'date +%Y-%m-%d') }}"

# template - رندر template
cfg: "{{ lookup('template', 'config.j2') }}"

# fileglob - لیست فایل‌ها
files: "{{ lookup('fileglob', 'files/*.txt', wantlist=True) }}"

# csvfile - خواندن CSV
value: "{{ lookup('csvfile', 'key file=data.csv delimiter=,') }}"

# ini - خواندن INI
db_host: "{{ lookup('ini', 'host section=database file=config.ini') }}"

# dig - DNS lookup
ip: "{{ lookup('dig', 'example.com') }}"

# aws_ssm - AWS Parameter Store
secret: "{{ lookup('aws_ssm', '/myapp/db_password') }}"

# hashivault - HashiCorp Vault
secret: "{{ lookup('hashi_vault', 'secret/data/myapp:password') }}"
```

---

## 🎨 Filter Plugins

تبدیل و پردازش داده‌ها در Jinja2.

### فیلترهای Ansible:

```yaml
vars:
  # JSON/YAML
  json_str: "{{ data | to_json }}"
  nice_json: "{{ data | to_nice_json(indent=2) }}"
  yaml_str: "{{ data | to_yaml }}"
  from_json: "{{ json_string | from_json }}"
  
  # رمزنگاری
  hashed: "{{ 'password' | hash('sha256') }}"
  pass_hash: "{{ 'password' | password_hash('sha512') }}"
  b64: "{{ 'hello' | b64encode }}"
  decoded: "{{ b64_string | b64decode }}"
  
  # لیست
  first: "{{ mylist | first }}"
  last: "{{ mylist | last }}"
  unique: "{{ mylist | unique }}"
  sorted: "{{ mylist | sort }}"
  flat: "{{ nested_list | flatten }}"
  combined: "{{ list1 | union(list2) }}"
  
  # Dictionary
  dict_items: "{{ mydict | dict2items }}"
  back_to_dict: "{{ items | items2dict }}"
  combined_dict: "{{ dict1 | combine(dict2) }}"
  
  # Regex
  matched: "{{ text | regex_search('pattern') }}"
  replaced: "{{ text | regex_replace('old', 'new') }}"
  
  # شبکه
  is_ip: "{{ '192.168.1.1' | ipaddr }}"
  network: "{{ '192.168.1.0/24' | ipaddr('network') }}"
  
  # مسیر فایل
  basename: "{{ '/path/to/file.txt' | basename }}"
  dirname: "{{ '/path/to/file.txt' | dirname }}"
  realpath: "{{ path | realpath }}"
  
  # پیش‌فرض
  with_default: "{{ undefined_var | default('fallback') }}"
  mandatory: "{{ must_exist | mandatory }}"
  
  # Type conversion
  int_val: "{{ '42' | int }}"
  float_val: "{{ '3.14' | float }}"
  bool_val: "{{ 'yes' | bool }}"
```

### ایجاد Filter سفارشی:

```python
# filter_plugins/custom_filters.py

def reverse_string(value):
    return value[::-1]

def add_prefix(value, prefix):
    return f"{prefix}{value}"

class FilterModule:
    def filters(self):
        return {
            'reverse_string': reverse_string,
            'add_prefix': add_prefix,
        }
```

استفاده:
```yaml
vars:
  reversed: "{{ 'hello' | reverse_string }}"  # olleh
  prefixed: "{{ 'world' | add_prefix('hello_') }}"  # hello_world
```

---

## 📞 Callback Plugins

هوک برای رویدادهای مختلف اجرا.

### فعال‌سازی در ansible.cfg:

```ini
[defaults]
# فعال کردن callback‌ها
callback_whitelist = timer, profile_tasks, profile_roles

# callback پیش‌فرض برای stdout
stdout_callback = yaml
# یا: json, debug, dense, minimal, oneline
```

### Callback‌های پرکاربرد:

| Callback | توضیح |
|----------|-------|
| `timer` | نمایش زمان کل اجرا |
| `profile_tasks` | زمان هر task |
| `profile_roles` | زمان هر role |
| `json` | خروجی JSON |
| `yaml` | خروجی YAML |
| `debug` | اطلاعات debug |
| `slack` | ارسال به Slack |
| `mail` | ارسال ایمیل |
| `log_plays` | لاگ به فایل |

### ایجاد Callback سفارشی:

```python
# callback_plugins/my_callback.py

from ansible.plugins.callback import CallbackBase

class CallbackModule(CallbackBase):
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'my_callback'
    
    def v2_playbook_on_start(self, playbook):
        self._display.display(f"Starting playbook: {playbook._file_name}")
    
    def v2_runner_on_ok(self, result):
        host = result._host.get_name()
        task = result._task.get_name()
        self._display.display(f"OK: {host} - {task}")
    
    def v2_runner_on_failed(self, result, ignore_errors=False):
        host = result._host.get_name()
        task = result._task.get_name()
        self._display.display(f"FAILED: {host} - {task}")
    
    def v2_playbook_on_stats(self, stats):
        self._display.display("Playbook finished!")
```

---

## 🌐 Connection Plugins

روش‌های اتصال به سرور هدف.

### انواع Connection:

```yaml
# SSH (پیش‌فرض)
- hosts: servers
  connection: ssh

# Local (روی کنترلر)
- hosts: localhost
  connection: local

# Docker
- hosts: containers
  connection: docker
  vars:
    ansible_docker_extra_args: "-H tcp://localhost:2375"

# WinRM (Windows)
- hosts: windows
  connection: winrm
  vars:
    ansible_winrm_transport: ntlm

# Network devices
- hosts: switches
  connection: network_cli
  vars:
    ansible_network_os: ios
```

---

## 📦 Inventory Plugins

منابع مختلف برای inventory.

### فعال‌سازی:

```ini
# ansible.cfg
[inventory]
enable_plugins = host_list, script, auto, yaml, ini, aws_ec2
```

### AWS EC2:

```yaml
# inventory/aws_ec2.yml
plugin: aws_ec2
regions:
  - us-east-1
  - us-west-2

keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: instance_type
    prefix: type

filters:
  instance-state-name: running

compose:
  ansible_host: public_ip_address
```

### Azure:

```yaml
# inventory/azure_rm.yml
plugin: azure.azcollection.azure_rm
include_vm_resource_groups:
  - myResourceGroup
auth_source: auto

keyed_groups:
  - key: tags.application
```

### GCP:

```yaml
# inventory/gcp.yml
plugin: google.cloud.gcp_compute
projects:
  - my-project
regions:
  - us-central1
auth_kind: serviceaccount
service_account_file: /path/to/sa.json

keyed_groups:
  - key: labels.environment
```

---

## 🔄 Strategy Plugins

کنترل نحوه اجرای task‌ها.

### استراتژی‌ها:

```yaml
# Linear (پیش‌فرض) - همه hostها task را تمام کنند، بعد برو task بعدی
- hosts: all
  strategy: linear
  tasks: ...

# Free - هر host مستقل پیش برود
- hosts: all
  strategy: free
  tasks: ...

# Debug - برای troubleshooting
- hosts: all
  strategy: debug
  tasks: ...

# Host pinned - task ها در یک host اجرا شود بعد host بعدی
- hosts: all
  strategy: host_pinned
  tasks: ...
```

---

## 🧪 Test Plugins

تست‌های Jinja2 برای استفاده با `is`.

### تست‌های داخلی:

```yaml
tasks:
  # بررسی تعریف شدن
  - debug:
      msg: "Defined"
    when: my_var is defined
  
  # بررسی نوع
  - debug:
      msg: "Is a string"
    when: my_var is string
  
  - debug:
      msg: "Is a number"
    when: my_var is number
  
  - debug:
      msg: "Is iterable"
    when: my_var is iterable
  
  # بررسی فایل
  - debug:
      msg: "File exists"
    when: "'/etc/passwd' is file"
  
  - debug:
      msg: "Is directory"
    when: "'/etc' is directory"
  
  # بررسی نسخه
  - debug:
      msg: "Version OK"
    when: my_version is version('2.0', '>=')
  
  # Regex
  - debug:
      msg: "Matches pattern"
    when: hostname is match("web.*")
  
  - debug:
      msg: "Contains substring"
    when: text is search("error")
```

### ایجاد Test سفارشی:

```python
# test_plugins/custom_tests.py

def is_even(value):
    return value % 2 == 0

def is_valid_email(value):
    import re
    pattern = r'^[\w\.-]+@[\w\.-]+\.\w+$'
    return bool(re.match(pattern, value))

class TestModule:
    def tests(self):
        return {
            'even': is_even,
            'valid_email': is_valid_email,
        }
```

استفاده:
```yaml
- debug:
    msg: "Even number"
  when: count is even

- debug:
    msg: "Valid email"
  when: email is valid_email
```

---

## 📂 مسیر پلاگین‌ها

### در پروژه:

```
project/
├── ansible.cfg
├── filter_plugins/
│   └── custom_filters.py
├── lookup_plugins/
│   └── custom_lookup.py
├── callback_plugins/
│   └── custom_callback.py
├── test_plugins/
│   └── custom_tests.py
└── playbooks/
```

### در ansible.cfg:

```ini
[defaults]
filter_plugins = ./filter_plugins:/usr/share/ansible/plugins/filter
lookup_plugins = ./lookup_plugins:/usr/share/ansible/plugins/lookup
callback_plugins = ./callback_plugins
test_plugins = ./test_plugins
```

---

## 📝 مثال Lookup Plugin کامل

```python
# lookup_plugins/vault_secret.py

from ansible.errors import AnsibleError
from ansible.plugins.lookup import LookupBase
import hvac

DOCUMENTATION = """
    lookup: vault_secret
    author: Your Name
    description:
      - Read secrets from HashiCorp Vault
    options:
      _terms:
        description: Path to secret
        required: True
      field:
        description: Field to extract
        default: value
"""

class LookupModule(LookupBase):
    def run(self, terms, variables=None, **kwargs):
        field = kwargs.get('field', 'value')
        
        # Get Vault settings
        vault_url = variables.get('vault_url', 'http://localhost:8200')
        vault_token = variables.get('vault_token')
        
        if not vault_token:
            raise AnsibleError("vault_token is required")
        
        client = hvac.Client(url=vault_url, token=vault_token)
        
        results = []
        for term in terms:
            try:
                secret = client.secrets.kv.read_secret_version(path=term)
                value = secret['data']['data'].get(field)
                results.append(value)
            except Exception as e:
                raise AnsibleError(f"Error reading {term}: {e}")
        
        return results
```

استفاده:
```yaml
vars:
  vault_url: "https://vault.example.com"
  vault_token: "{{ lookup('env', 'VAULT_TOKEN') }}"
  
  db_password: "{{ lookup('vault_secret', 'database/creds', field='password') }}"
```

---

## ⚠️ نکات مهم

1. **مسیر پلاگین‌ها**: در ansible.cfg یا متغیر محیطی تنظیم کنید
2. **کش**: برخی lookup‌ها کش می‌شوند
3. **امنیت**: lookup‌ها روی کنترلر اجرا می‌شوند
4. **Python**: پلاگین‌ها باید با Python نوشته شوند

---

## 📚 منابع

- [Ansible Plugins](https://docs.ansible.com/ansible/latest/plugins/plugins.html)
- [Developing Plugins](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html)
- [Lookup Plugins](https://docs.ansible.com/ansible/latest/plugins/lookup.html)
- [Filter Plugins](https://docs.ansible.com/ansible/latest/plugins/filter.html)
