---
title: "jinja2语言-template模板"
date: 2020-10-18T09:03:20-08:00
draft: true
tags: [Ansible,linux]
---

# Jinja语言

jinja2 语言使用字面量，有下面形式：

字符串：使用单引号或双引号

数字：整数，浮点数

列表：[item1, item2, …]

元组：(item1, item2, …)

字典：{key1:value1, key2:value2, …}

布尔型：true/false

算术运算：+, -, *, /, //, %, **

比较操作：==, !=, >, >=, <, <=

逻辑运算：and，or，not

流表达式：For，If，When

## 字面量

表达式最简单的形式就是字面量。字面量表示诸如字符串和数值的 Python 对象。如”Hello World”

双引号或单引号中间的一切都是字符串。无论何时你需要在模板中使用一个字符串（比如函数调用、过滤器或只是包含或继承一个模板的参数），如42，42.23

数值可以为整数和浮点数。如果有小数点，则为浮点数，否则为整数。在 Python 里， 42 和 42.0 是不一样的

## 算数运算

Jinja 允许用计算值。支持下面的运算符

+：把两个对象加到一起。通常对象是素质，但是如果两者是字符串或列表，你可以用这 种方式来衔接它们。无论如何这不是首选的连接字符串的方式！连接字符串见 ~ 运算符。 2 等于 2

-：用第一个数减去第二个数。 1 等于 1

/：对两个数做除法。返回值会是一个浮点数。 0.5 等于 0.5

//：对两个数做除法，返回整数商。 2 等于 2

%：计算整数除法的余数。 4 等于 4

*：用右边的数乘左边的操作数。 4 会返回 4 。也可以用于重 复一个字符串多次。 NaN会打印 80 个等号的横条

**：取左操作数的右操作数次幂。 8 会返回 8

## 比较操作符

== 比较两个对象是否相等

!= 比较两个对象是否不等

\> 如果左边大于右边，返回 true

= 如果左边大于等于右边，返回 true

< 如果左边小于右边，返回 true

<= 如果左边小于等于右边，返回 true

## 逻辑运算符

对于 if 语句，在 for 过滤或 if 表达式中，它可以用于联合多个表达式

and 如果左操作数和右操作数同为真，返回 true

or 如果左操作数和右操作数有一个为真，返回 true

not 对一个表达式取反

(expr)表达式组

true / false true 永远是 true ，而 false 始终是 false

## template

template 功能：可以根据和参考模板文件，动态生成类似的配置文件

template 文件必须存放于 templates 目录下，且命名为 .j2 结尾

yaml/yml 文件需要和 templates 目录平级

```
template 功能：可以根据和参考模板文件，动态生成类似的配置文件

template 文件必须存放于 templates 目录下，且命名为 .j2 结尾

yaml/yml 文件需要和 templates 目录平级
```

## 逻辑判断类型

if 判断例子

```
{% if variable is defined %}    ---{% if variable is defined %} 等同于 {% if variable != None %} 
    value of variable: {{ variable }}
{% elif variable == 3 %}
    value is 3
{% else %}
    variable is not defined
{% endif %}

{% if username == 'AAA' %}            ---判断 username 变量的内容
    server_id {{ ID }}
{% elif username == 'BB' %}
    server_id {{ ID }}
{% else %}


for 循环例子

{% for var1 in var2 %}
    server_id {{ var1 }};
{% endfor %}

```

# 范例

## 利用 template 同步 nginx 配置文件

1. 目录树

```
[root@DESKTOP-NHT2EP5 ~]#tree
.
├── temp1.yaml
└── templates
    └── nginx.conf.j2
```

2. templates/nginx.conf.j2 文件配置内容

```
# 通过 ansible 的 setup 模块中定义的变量来为主机定义 worker 进程
worker_processes {{ ansible_processor_vcpus*2 }};
```

3. temp1.yaml 文件内容

```
[root@DESKTOP-NHT2EP5 ~]#cat temp1.yaml
---
- hosts: test
  remote_user: root
  gather_facts: yes
  tasks:
    - name: install nginx
      yum: name=nginx state=present
    - name: copy nginx.conf
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
    - name: start nginx
      service: name=nginx state=started
```

## 使用 for 循环定义多个 nginx 实例

1. 目录树

```
[root@DESKTOP-NHT2EP5 ~]#tree
.
├── temp1.yaml
└── templates
    └── nginx.conf.j2
```

2. templates/nginx.conf.j2 文件内容

```
{% for port in nginx %}
    server {
        listen       {{ port }} default_server;
}
{% endfor %}
```

3. temp1.yaml 文件内容

```
---
- hosts: test
  remote_user: root
  gather_facts: no
  vars:
    nginx: [ 80 , 8080 , 89 ]
    # nginx: { 80, 8080, 89 }
  tasks:
    - name: install nginx
      yum: name=nginx state=present
    - name: template
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
```

**生成结果**

```
server {
        listen       80 default_server;
}
    server {
        listen       8080 default_server;
}
    server {
        listen       89 default_server;
}
```

## 在模板文件中还可以使用 if 条件判断，决定是否生成相关的配置信息

1. temp.yaml 文件

```
- hosts: websrvs
remote_user: root
vars:
 nginx_vhosts:
  - web1:
   listen: 8080
   root: "/var/www/nginx/web1/"
  - web2:
   listen: 8080
   server_name: "web2.magedu.com"
   root: "/var/www/nginx/web2/"
  - web3:
   listen: 8080
   server_name: "web3.magedu.com"
   root: "/var/www/nginx/web3/"
tasks:
 - name: template config to
  template: src=nginx.conf5.j2 dest=/data/nginx5.conf
```

2. jinja2 文件内容

```
{% for vhost in nginx_vhosts %}
server {
 listen {{ vhost.listen }}
 {% if vhost.server_name is defined %}
server_name {{ vhost.server_name }}
 {% endif %}
root  {{ vhost.root }}
}
{% endfor %}
```

3. 生成结果

```
server {
 listen 8080
 root /var/www/nginx/web1/
}
server {
 listen 8080
 server_name web2.magedu.com
 root /var/www/nginx/web2/
}
server {
 listen 8080
 server_name web3.magedu.com
 root /var/www/nginx/web3/
}
```