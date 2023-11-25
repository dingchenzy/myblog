---
title: "yaml语言"
date: 2020-10-18T09:03:20-08:00
draft: true
tags: [Ansible,linux]
---

# YAML 语言介绍

YAML：YAML Ain’t Markup Language，即YAML不是XML。不过，在开发的这种语言时，YAML的意
思其实是：”Yet Another Markup Language”（仍是一种标记语言）

YAML是一个可读性高的用来表达资料序列的格式。YAML参考了其他多种语言，包括：XML、C语言、
Python、Perl以及电子邮件格式RFC2822等。Clark Evans在2001年在首次发表了这种语言，另外Ingy
döt Net与Oren Ben-Kiki也是这语言的共同设计者,目前很多最新的软件比较流行采用此格式的文件存放
配置信息，如:ubuntu，anisble，docker，kubernetes等

YAML 官方网站：http://www.yaml.org

## YAML 语言特点

- YAML的可读性好
- YAML和脚本语言的交互性好
- YAML使用实现语言的数据类型
- YAML有一个一致的信息模型
- YAML易于实现
- YAML可以基于流来处理
- YAML表达能力强，扩展性好

# YAML 语法简介

- 在单一文件第一行，用连续三个连字号”-“ 开始，还有选择性的连续三个点号( … )用来表示文件的结尾
- 次行开始正常写Playbook的内容，一般建议写明该Playbook的功能
- 使用**#号注释代码**
- 缩进必须是统一的，不能空格和tab混用
- 缩进的级别也必须是一致的，同样的缩进代表同样的级别，程序判别配置的级别是通过缩进结合换行来实现的
- YAML文件内容是区别大小写的，key/value的值均需大小写敏感
- 多个key/value可同行写也可换行写，同行使用，分隔
- key后面冒号要加一个空格 比如: key: value
- value可是个字符串，也可是另一个列表
- YAML文件扩展名通常为yml或yaml

# 支持的数据类型

YAML 支持以下常用几种数据类型：

- 标量：单个的、不可再分的值
- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）

## scalar 标量

key对应value

```
name: wang
```

使用缩进的方式

```
name:
  wang
```

标量是最基本的，不可再分的值，包括：

- 字符串
- 布尔值
- 整数
- 浮点数
- Null
- 时间
- 日期

## Dictionary 字典

字典由多个key与value构成，key和value之间用 ：分隔, 并且 : 后面有一个空格，所有k/v可以放在一
行，或者每个 k/v 分别放在不同行

格式

```
account: { name: wang, age: 30 }
```

使用缩进方式

```
account:
  name: wang
  age: 30
```

范例:

```
#不同行
# An employee record
name: Example Developer
job: Developer
skill: Elite(社会精英)
#同一行,也可以将key:value放置于{}中进行表示，用,分隔多个key:value
# An employee record
{name: "Example Developer", job: "Developer", skill: "Elite"}
```

## List 列表

列表由多个元素组成，每个元素放在不同行，且元素前均使用”-“打头，并且 - 后有一个空格, 或者将所
有元素用 [ ] 括起来放在同一行

格式

```
course:
  - linux
  - golang
  - python
```

也可以使用[]

course: [ linux , golang , python ]

数据里面也可以包含对象

```
course:
  - linux: manjaro
  - golang: gin
  - python: django
```

范例：

```
#不同行,行以-开头,后面有一个空格
# A list of tasty fruits
- Apple
- Orange
- Strawberry
- Mango
#同一行
[Apple,Orange,Strawberry,Mango]
```

范例：YAML 表示一个家庭

```
name: John Smith
age: 41
gender: Male
spouse:
name: Jane Smith
age: 37
gender: Female
children:
- name: Jimmy Smith
  age: 17
  gender: Male
- {name: Jenny Smith, age: 13, gender: Female}
- {name: hao Smith, age: 20, gender: Male }  
```

# 三种常见的数据格式

- XML：Extensible Markup Lang
- JSON：JavaScript Object Notation, JavaScript 对象表记法，主要用来数据交换或配置，不支持注释
- YAML：YAML Ain’t Markup Language YAML 不是一种标记语言， 主要用来配置，大小写敏感，不支持tab

可以用工具互相转换，参考网站：

https://www.json2yaml.com/

http://www.bejson.com/json/json2yaml/