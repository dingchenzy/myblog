---
title: "Ansible-Playbook"
date: 2020-10-18T09:03:20-08:00
draft: true
tags: [Ansible,linux]
---

# Playbook 介绍

![img](image-20201018122729495.png)

- playbook 剧本是由一个或多个”play”组成的列表
- play的主要功能在于将预定义的一组主机，装扮成事先通过ansible中的task定义好的角色。Task实际是调用ansible的一个module，将多个play组织在一个playbook中，即可以让它们联合起来，按事先编排的机制执行预定义的动作
- Playbook 文件是采用YAML语言编写的

# Playbook 命令

## 格式

```
ansible-playbook <filename.yml> ... [options]
```

## 常用选项

```
--syntax-check    #语法检查
-C --check #只检测可能会发生的改变，但不真正执行操作
--list-hosts   #列出运行任务的主机
    jasonchen@DESKTOP-NHT2EP5:~$ ansible-playbook copy.yaml --list-hosts
--list-tags #列出tag
    jasonchen@DESKTOP-NHT2EP5:~$ ansible-playbook copy.yaml --list-tags
--list-tasks #列出task
    jasonchen@DESKTOP-NHT2EP5:~$ ansible-playbook copy.yaml --list-tasks
--limit 主机列表 #只针对主机列表中的特定主机执行
    jasonchen@DESKTOP-NHT2EP5:~$ ansible-playbook copy.yaml --limit 100.0.0.10
    # 设置 limit 的时候，此 IP 必须要存在于脚本定义的 hosts 
-v -vv  -vvv # 显示详细过程，v 越多显示的详细信息越多
```

# Playbook 核心组件

## hosts 组件

Hosts：playbook中的每一个play的目的都是为了让特定主机以某个指定的用户身份执行任务。hosts 用于指定要执行指定任务的主机，须事先定义在主机清单中，然后直接在 yaml 格式的文本中通过 hosts 关键字调用即可

```
one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*
Websrvs:dbsrvs   #或者，两个组的并集
Websrvs:&dbsrvs  #与，两个组的交集
webservers:!dbsrvs  #在websrvs组，但不在dbsrvs组
```

范例

```
hosts: Websrvs:dbsrvs
```

## remote_user 组件

remote_user: 可用于Host和task中。也可以通过指定其通过sudo的方式在远程主机上执行任务，其可
用于play全局或某任务；此外，甚至可以在sudo时使用sudo_user指定sudo时切换的用户

```
- hosts: websrvs
remote_user: root
tasks:
 - name: test connection
   ping:
   remote_user: magedu
   sudo: yes #默认sudo为root
   sudo_user: chen   #sudo为chen
```

## task 列表和 action 组件

play的主体部分是task list，task list中有一个或多个task,各个task 按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个task后，再开始第二个task

task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致

每个task都应该有其name，用于playbook的执行结果输出，建议其内容能清晰地描述任务执行步骤。
如果未提供name，则action的结果将用于输出

```
---
- hosts: websrvs
remote_user: root
gather_facts: no
tasks:
 - name: install httpd
   yum: name=httpd
 - name: start httpd
   service: name=httpd state=started enabled=yes
   ```

   ## handlers 和 notify 组件

类似于 mysql 中的触发器一样，当通过 notify 执行触发 handlers。Handlers本质是task list ，类似于MySQL中的触发器触发的行为，其中的task与前述的task并没有本质上的不同，主要用于当关注的资源发生变化时，才会采取一定的操作。而Notify对应的action可用于在每个play的最后被触发，这样可避免多次有改变发生时每次都执行指定的操作，仅在所有的变化发生完成后一次性地执行指定操作。在notify中列出的操作称为handler，也即notify中调用handler中定义的操作

范例

   ```
   ---
- hosts: websrvs
remote_user: root
gather_facts: no
tasks:
  - name: Install httpd
  yum: name=httpd state=present
  - name: Install configure file
  copy: src=files/httpd.conf dest=/etc/httpd/conf/
  notify: restart httpd
  - name: ensure apache is running
   service: name=httpd state=started enabled=yes
handlers:
  - name: restart httpd
   service: name=httpd state=restarted

   ```
   ## tags 组件

   为 task 打上一个标签，可以通过 ansible-playbook -t [tags] 标签来指定某一个 task 来执行。

```
   vim httpd.yml
---
# tags example
- hosts: websrvs
remote_user: root
gather_facts: no
tasks:
  - name: Install httpd
  yum: name=httpd state=present
  - name: Install configure file
  copy: src=files/httpd.conf dest=/etc/httpd/conf/
  tags: conf
  - name: start httpd service
  tags: service
   service: name=httpd state=started enabled=yes
[root@ansible ~]#ansible-playbook –t conf,service httpd.yml
```

# Ansible-Playbook 的条件判断

## when

## 格式

```
- name: when regex
  template: src=ngxin.conf.j2 dest=/etc/nginx/nginx.conf
  when: ansible_distribution_major_version == "7"

如果系统为 7 那么就会执行这个 jinja 脚本
```

## 范例

复制不同的配置文件到不同版本的主机

1. 目录树结构

```
[root@DESKTOP-NHT2EP5 ~]#tree
.
├── temp1.yaml
└── templates
    ├── nginx.conf.6.j2
    └── nginx.conf.7.j2
```

2. temp1.yaml 文件内容

```
[root@DESKTOP-NHT2EP5 ~]#cat temp1.yaml
---
- hosts: test
  remote_user: root
  tasks:
    - name: install httpd
      yum: name=httpd state=present
    - name: template 6
      template: src=nginx.conf.6.j2 dest=/etc/httpd/
      when: ansible_distribution_major_version == "6"
    - name: template 7
      template: src=nginx.conf.7.j2 dest=/etc/httpd/
      when: ansible_distribution_major_version == "7"
    - name: start httpd
      service: name=httpd state=started
```

## nginx 文件内容

```
[root@DESKTOP-NHT2EP5 ~]#cat templates/*
this is centos 6
this is centos 7
```

# Playbook 使用迭代 with_items(loop)

迭代：当有需要重复性执行任务时，可以使用迭代机制。对迭代项的引用，固定变量名为 “item” 要在 task 中使用 with_items 给定要迭代的元素列表。

注意：ansible 2.5 后可以使用 loop 代替 with_items

## 列表元素格式：

- 字符串格式，直接使用列表来内容
- 字典，通过键值和数据的对应来定义元素（key value ）值来定义
- 调用时可以直接使用 item.key 来调用 key 对应的数据

## 范例

## 创建不同的用户对应不同的组

```
[root@DESKTOP-NHT2EP5 ~]#cat useradd.yaml
---
- hosts: test
  remote_user: root
  tasks:
    - name: group add
      group: name={{ item }}
      with_items:
        - shandong
        - guiyang
        - hebei
    - name: user add
      user: name={{ item.user }} group={{ item.group }}
      with_items:
        - { user: chen , group: shandong }
        - { user: yang , group: shandong }
        - { user: dong , group: guiyang }
        - { user: chao , group: hebei }
```

## 删除不同的用户

```
[root@DESKTOP-NHT2EP5 ~]#cat useradd.yaml
---
- hosts: test
  remote_user: root
  tasks:
    - name: user del
      user: name={{ item.user }} state=absent remove=yes
      with_items:
        - user: chen
        - user: yang
        - user: dong
        - user: chao
    - name: group del
      group: name={{ item }} state=absent
      with_items:
        - shandong
        - guiyang
        - hebei
```

# 管理节点过多导致的超时问题解决方法(serial 参数)

默认情况下主机执行剧本是将所有主机的第一个 task 执行完成后才会继续往下执行第二个 task，因为是所有主机一起执行的。所以当主机如果出现故障，那么所有的机器都会出现更新到中间的状态。

所以可以使用 serial 关键字指定将 task 动作全部执行完成的主机数量，也可以执行百分比。

```
serial: 2            ---每次执行完成两台主机（执行完所有 task）
serial: 20%            ---每次执行主机数量的百分之二十
```

## 案例

**每次执行 1 台主机**

```
[root@DESKTOP-NHT2EP5 ~]#ansible-playbook useradd.yaml
PLAY [test] 

TASK [user add] 
changed: [100.0.0.104]

PLAY [test] 

TASK [user add] 
changed: [100.0.0.12]

100.0.0.104                : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
100.0.0.12                 : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## 每次执行百分之二十

```
- name: test serail
hosts: all
serial: "20%"  #每次只同时处理20%的主机
```