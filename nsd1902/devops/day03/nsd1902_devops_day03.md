# nsd1902_devops_day03

## ansible基础应用

### 在虚拟环境下安装ansible

```python
# cd ansible_pkg/
# pip3 install *

# 或 在线安装
# pip3 install ansible==2.7.2

# yum install -y sshpass
```

### 准备运行环境

```shell
# 创建工作目录并创建配置文件
# mkdir myansible
# cd myansible
# vim ansible.cfg
[defaults]
inventory = hosts
remote_user = root

# vim hosts
[dbservers]
node5.tedu.cn

[webservers]
node6.tedu.cn
node7.tedu.cn

# 配置名称解析
[root@room8pc16 nsd2019]# for i in {1..254}
> do
> echo -e "192.168.4.$i\tnode$i.tedu.cn\tnode$i" >> /etc/hosts
> done


# 收集远程主机的密钥并保存。用户登陆时，不再提示是否接受密钥
[root@room8pc16 nsd2019]# ssh-keyscan node{5..7} node{5..7}.tedu.cn 192.168.4.{5..7} >> ~/.ssh/known_hosts 


# 测试环境
# ansible all -m ping -k
```

### ansible之adhoc（临时命令）

```shell
# ansible 主机　-m 模块　-a "参数"
```

### ansible之playbook

- 配置远程主机的密钥

```shell
# 查模块帮助
# ansible-doc authorized_key

# vim auth_key.yml
---
- name: configure ssh key
  hosts: all
  tasks:
    - name: upload key
      authorized_key:
        user: root
        state: present
        key: "{{ lookup('file', '/root/.ssh/id_rsa.pub') }}"
# 检查语法
# ansible-playbook --syntax-check auth_key.yml

# 执行playbook
# ansible-playbook auth_key.yml -k
```

- 配置yum

```shell
# mkdir files
# vim files/servers.repo
# 编写playbook
# vim  mkrepo.yml
---
- name: configure yum
  hosts: all
  tasks:
    - name: upload repo file
      copy:
        src: files/servers.repo
        dest: /etc/yum.repos.d/server.repo

# ansible-playbook mkrepo.yml
```

### 配置lamp分离结构

```shell
# vim lamp.yml
---
- name: configure webservers
  hosts: webservers
  tasks:
    - name: install web pkgs
      yum:
        name: [httpd, php, php-mysql]
        state: present
    - name: configure web service
      service:
        name: httpd
        state: started
        enabled: yes

- name: configure dbservers
  hosts: dbservers
  tasks:
    - name: install db pkgs
      yum:
        name: mariadb-server
        state: present
    - name: configure db service
      service:
        name: mariadb
        state: started
        enabled: yes
# ansible-playbook lamp.yml
```

### 命名元组

命名元组是对元组的一个扩展，它也支持原始的元组的操作，同时它给每个元素起名。访问元组的元素时，既可以通过下标访问，也可以通过名称访问。

```python
>>> from collections import namedtuple
>>> Point = namedtuple('Point', ['x', 'y', 'z'])
>>> p1 = Point(10, 20, 30)
>>> p1
Point(x=10, y=20, z=30)
>>> p1.x
10
>>> p1.y
20
>>> p1.z
30
>>> p1[0]
10
>>> p1[:2]
(10, 20)
```












