# 🧩 Jinja2 Templates in Ansible

> استفاده از template engine Jinja2 در Ansible

---

## 🎯 Jinja2 چیست؟

Jinja2 یک template engine برای Python است که Ansible از آن برای پردازش متغیرها، شرط‌ها، حلقه‌ها و فیلترها استفاده می‌کند.

---

## 📋 سینتکس پایه

```jinja2
{# این یک کامنت است - نمایش داده نمی‌شود #}

{{ variable }}              {# نمایش متغیر #}
{{ variable | filter }}     {# اعمال فیلتر #}

{% if condition %}          {# شرط #}
{% endif %}

{% for item in list %}      {# حلقه #}
{% endfor %}
```

---

## 🔤 متغیرها

```jinja2
{# متغیر ساده #}
Hello {{ username }}!

{# متغیر با مقدار پیش‌فرض #}
Port: {{ http_port | default(80) }}

{# دسترسی به dictionary #}
{{ user.name }}
{{ user['name'] }}

{# دسترسی به لیست #}
{{ servers[0] }}
{{ servers[-1] }}   {# آخرین آیتم #}
```

---

## 🔧 فیلترهای پرکاربرد

### فیلترهای رشته:

```jinja2
{# تبدیل به حروف بزرگ/کوچک #}
{{ name | upper }}
{{ name | lower }}
{{ name | capitalize }}
{{ name | title }}

{# جایگزینی #}
{{ path | replace("/", "-") }}

{# برش رشته #}
{{ name | truncate(10) }}

{# حذف فاصله‌ها #}
{{ text | trim }}

{# regex #}
{{ text | regex_replace('^www\.', '') }}
{{ text | regex_search('([0-9]+)', '\\1') }}

{# Base64 #}
{{ secret | b64encode }}
{{ encoded | b64decode }}

{# Hash #}
{{ password | hash('sha256') }}
{{ password | password_hash('sha512') }}
```

### فیلترهای لیست:

```jinja2
{# طول لیست #}
{{ users | length }}

{# اولین/آخرین #}
{{ items | first }}
{{ items | last }}

{# مرتب‌سازی #}
{{ users | sort }}
{{ users | sort(attribute='name') }}
{{ users | reverse }}

{# یکتاسازی #}
{{ items | unique }}

{# فیلتر کردن #}
{{ users | selectattr('active', 'equalto', true) | list }}
{{ users | rejectattr('disabled') | list }}

{# استخراج attribute #}
{{ users | map(attribute='name') | list }}

{# ترکیب لیست‌ها #}
{{ list1 | union(list2) }}
{{ list1 | intersect(list2) }}
{{ list1 | difference(list2) }}

{# random #}
{{ ['a', 'b', 'c'] | random }}

{# flatten #}
{{ nested_list | flatten }}

{# join #}
{{ items | join(', ') }}
```

### فیلترهای عددی:

```jinja2
{# محاسبات #}
{{ value | int }}
{{ value | float }}
{{ value | abs }}
{{ value | round(2) }}

{# مجموع، حداقل، حداکثر #}
{{ numbers | sum }}
{{ numbers | min }}
{{ numbers | max }}

{# تبدیل واحد #}
{{ size_bytes | human_readable }}
{{ '1GB' | human_to_bytes }}
```

### فیلترهای JSON/YAML:

```jinja2
{# به JSON #}
{{ data | to_json }}
{{ data | to_nice_json(indent=2) }}

{# به YAML #}
{{ data | to_yaml }}
{{ data | to_nice_yaml }}

{# از JSON #}
{{ json_string | from_json }}
{{ yaml_string | from_yaml }}
```

### فیلترهای مسیر فایل:

```jinja2
{# نام فایل #}
{{ "/path/to/file.txt" | basename }}     {# file.txt #}

{# مسیر دایرکتوری #}
{{ "/path/to/file.txt" | dirname }}      {# /path/to #}

{# پسوند #}
{{ "file.tar.gz" | splitext }}           {# ['file.tar', '.gz'] #}

{# مسیر واقعی #}
{{ "~/file" | expanduser }}
{{ path | realpath }}
```

### فیلترهای شبکه:

```jinja2
{# بررسی IP #}
{{ '192.168.1.1' | ipaddr }}
{{ '192.168.1.0/24' | ipaddr('network') }}
{{ '192.168.1.0/24' | ipaddr('broadcast') }}

{# استخراج بخش‌ها #}
{{ 'http://example.com:8080/path' | urlsplit('hostname') }}
{{ 'http://example.com:8080/path' | urlsplit('port') }}
```

---

## 🔀 شرط‌ها (Conditionals)

```jinja2
{# if ساده #}
{% if user.admin %}
admin_mode = true
{% endif %}

{# if-else #}
{% if env == 'production' %}
debug = false
{% else %}
debug = true
{% endif %}

{# if-elif-else #}
{% if ansible_distribution == 'Ubuntu' %}
package_manager = apt
{% elif ansible_distribution == 'CentOS' %}
package_manager = yum
{% else %}
package_manager = unknown
{% endif %}

{# شرط‌های ترکیبی #}
{% if user.active and user.age >= 18 %}
...
{% endif %}

{% if role == 'admin' or role == 'superuser' %}
...
{% endif %}

{% if not user.disabled %}
...
{% endif %}

{# بررسی تعریف شدن متغیر #}
{% if my_var is defined %}
...
{% endif %}

{% if my_var is undefined %}
...
{% endif %}

{# بررسی خالی بودن #}
{% if items %}  {# اگر خالی نباشد #}
...
{% endif %}

{# تست‌ها (Tests) #}
{% if value is number %}
{% if value is string %}
{% if value is mapping %}   {# dictionary #}
{% if value is iterable %}
{% if path is file %}
{% if path is directory %}
```

---

## 🔄 حلقه‌ها (Loops)

```jinja2
{# حلقه ساده #}
{% for user in users %}
- {{ user }}
{% endfor %}

{# با index #}
{% for user in users %}
{{ loop.index }}. {{ user }}    {# شروع از 1 #}
{{ loop.index0 }}. {{ user }}   {# شروع از 0 #}
{% endfor %}

{# متغیرهای loop #}
{% for item in items %}
{{ loop.first }}    {# true اگر اولین باشد #}
{{ loop.last }}     {# true اگر آخرین باشد #}
{{ loop.length }}   {# تعداد کل #}
{{ loop.revindex }} {# شمارش معکوس #}
{% endfor %}

{# حلقه روی dictionary #}
{% for key, value in my_dict.items() %}
{{ key }} = {{ value }}
{% endfor %}

{# با شرط #}
{% for user in users if user.active %}
{{ user.name }}
{% endfor %}

{# با else (اگر لیست خالی باشد) #}
{% for user in users %}
{{ user }}
{% else %}
No users found!
{% endfor %}

{# حلقه تو در تو #}
{% for server in servers %}
[{{ server.name }}]
{% for port in server.ports %}
  port = {{ port }}
{% endfor %}
{% endfor %}

{# range #}
{% for i in range(5) %}
server{{ i }}
{% endfor %}
```

---

## 📄 Template Module در Playbook

### فایل template:

```jinja2
{# templates/nginx.conf.j2 #}

# Generated by Ansible for {{ ansible_hostname }}
# Do not edit manually!

user {{ nginx_user | default('www-data') }};
worker_processes {{ ansible_processor_vcpus }};

http {
    server {
        listen {{ http_port | default(80) }};
        server_name {{ server_name }};
        root {{ document_root }};
        
{% for location in locations %}
        location {{ location.path }} {
            proxy_pass {{ location.backend }};
        }
{% endfor %}
    }
}
```

### استفاده در Playbook:

```yaml
---
- name: Deploy nginx config
  hosts: webservers
  vars:
    nginx_user: nginx
    http_port: 80
    server_name: example.com
    document_root: /var/www/html
    locations:
      - path: /api
        backend: http://localhost:3000
      - path: /static
        backend: http://localhost:8080
  
  tasks:
    - name: Copy nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
        validate: 'nginx -t -c %s'
      notify: Reload nginx
```

---

## 🧪 تست‌های Jinja2

```jinja2
{# تست‌های متغیر #}
{% if var is defined %}
{% if var is undefined %}
{% if var is none %}

{# تست‌های رشته #}
{% if name is string %}
{% if value is number %}
{% if flag is sameas true %}

{# تست‌های لیست/dict #}
{% if data is iterable %}
{% if data is mapping %}
{% if data is sequence %}

{# تست‌های فایل (Ansible specific) #}
{% if path is file %}
{% if path is directory %}
{% if path is link %}
{% if path is exists %}

{# تست‌های مقایسه #}
{% if version is version('2.0', '>=') %}
{% if value is match('regex_pattern') %}
{% if value is search('substring') %}
```

---

## 🔧 Macros (توابع)

```jinja2
{# تعریف macro #}
{% macro render_server(name, ip, port=80) %}
server {
    name = {{ name }};
    address = {{ ip }}:{{ port }};
}
{% endmacro %}

{# استفاده از macro #}
{{ render_server('web1', '192.168.1.10') }}
{{ render_server('web2', '192.168.1.11', 8080) }}
```

---

## 📂 Include و Import

```jinja2
{# include کردن فایل دیگر #}
{% include 'header.j2' %}

{# include با متغیر #}
{% include ansible_os_family + '.j2' %}

{# include با ignore اگر نبود #}
{% include 'optional.j2' ignore missing %}

{# import macro از فایل دیگر #}
{% from 'macros.j2' import render_server %}
```

---

## 🎨 کنترل Whitespace

```jinja2
{# حذف whitespace قبل/بعد #}
{%- if condition -%}
content
{%- endif -%}

{# خط جدید نمایش داده نمی‌شود #}
{% for item in items -%}
{{ item }}
{%- endfor %}
```

---

## 📝 مثال کامل: فایل Hosts

```jinja2
{# templates/hosts.j2 #}

# /etc/hosts - Generated by Ansible
# Last updated: {{ ansible_date_time.iso8601 }}

127.0.0.1   localhost
127.0.1.1   {{ ansible_hostname }}

# IPv6
::1         localhost ip6-localhost ip6-loopback
ff02::1     ip6-allnodes
ff02::2     ip6-allrouters

# Application servers
{% for host in groups['webservers'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }}    {{ host }} {{ hostvars[host]['ansible_hostname'] }}
{% endfor %}

# Database servers
{% for host in groups['dbservers'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }}    {{ host }}
{% endfor %}

{% if custom_hosts is defined %}
# Custom entries
{% for entry in custom_hosts %}
{{ entry.ip }}    {{ entry.names | join(' ') }}
{% endfor %}
{% endif %}
```

---

## ⚠️ نکات مهم

1. **Escape کردن**: برای نمایش `{{` از `{{ '{{' }}` استفاده کنید
2. **Quote کردن**: در YAML، متغیرها را در quote بگذارید: `"{{ var }}"`
3. **Undefined**: همیشه برای متغیرهای اختیاری از `| default()` استفاده کنید
4. **Debug**: از `ansible -m debug -a 'msg={{ var | to_nice_json }}'` استفاده کنید

---

## 📚 منابع

- [Jinja2 Documentation](https://jinja.palletsprojects.com/)
- [Ansible Filters](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html)
- [Ansible Tests](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html)
