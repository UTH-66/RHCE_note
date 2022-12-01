# RHCE考试指北

## 0.前置准备

### 环境

本次考试将会使用6台虚拟机，分别是

| VM Name       | IP addresses     | Ansible Role           | Domain Name           |
| ------------- | ---------------- | ---------------------- | --------------------- |
| `bastion`     | `172.25.250.254` | `ansible control node` | `ansible.example.com` |
| `workstation` | `172.25.250.9`   | `ansible managed node` | `node1.example.com`   |
| `servera`     | `172.25.250.10`  | `ansible managed node` | `node2.example.com`   |
| `serverb`     | `172.25.250.11`  | `ansible managed node` | `node3.example.com`   |
| `serverc`     | `172.25.250.12`  | `ansible managed node` | `node4.example.com`   |
| `serverd`     | `172.25.250.13`  | `ansible managed node` | `node5.example.com`   |

所有的主机都已经安装了`ansible`，如果可以尝试`yum install ansible -y`命令进行手动安装。

### 关于用户

本次登录时，应该使用考试指定的用户，如`student`、`grep`和`devops`等，以下都称其为`[user]`，请根据实际需要自动将其替换为指定的用户。

`[user]`用户是具有一定的`root`权限的，因此最好使用`[user]`用户进行相关操作，在执行系统配置命令时，有时需要在命令前添加`sudo`。

### 配置文件

由于需要将所有的工作文件保存到控制节点的`/home/[user]/ansible`的目录中，因此需要注意复制或是更改输出配置文件，使所有相关文件输出至相关目录中。

因此最好使用`[user]`进行相关操作

由于环境使用的配置用户是`student`，因此题目将会是以`student`来进行编写

### 练习环境设置

需要将资源包安装到`/content`目录中

## 1.安装与配置`Ansible`

按照下方所述，在控制节点 `control.example.com` 上安装和配置 `Ansible`：

1. 安装所需的软件包
2. 创建名为`/home/student/ansible/inventory`的静态清单文件，以满足以下要求：
   1. `node1`是`dev`主机组的成员
   2. `node2`是`test`主机组的成员
   3. `node3`和`node4`是`prod`主机组的成员 
   4. `node5` 是`balancers`主机组的成员 
   5. `prod`组是`webservers`主机组的成员 
3. 创建名为`/home/student/ansible/ansible.cfg`的配置文件，以满足以下要求： 
   1. 主机清单文件为`/home/student/ansible/inventory`
   2. `playbook`中使用的角色的位置包括`/home/student/ansible/roles`

### Deploy

```sh
#1
#安装ansible
$ sudo yum install -y ansible
#----------------------------
#2
#创建用户目录下的ansible目录和roles角色目录
$ mkdir -p /home/student/ansible/roles
#最好进入到 ansible中，因为执行playbook时需要在该目录下
$ cd ./ansible
#创建并配置静态清单文件
$ vim /home/student/ansible/inventory
#添加下面这些配置
#出于后续配置的简化，建议配置环境变量，简化远程连接的操作，看考试环境来配置密码
```

```
[all:vars]
ansible_user=root
ansible_password=redhat
[dev]
172.25.250.9
[test]
172.25.250.10
[prod]
172.25.250.11
172.25.250.12
[balancers]
172.25.250.13
[webservers:children]
prod
```

```shell
#查看ansible的配置相关信息
$ ansible --version
#输出原始配置路径>>
config file = /etc/ansible/ansible.cfg
#复制一份ansible的配置文件，当前用户目录下的ansible配置文件优先级是最高的
$ cp /etc/ansible/ansible.cfg /home/student/ansible/
#再次查看ansible信息,发现配置文件已经修改
$ ansible --version
#>>
config file = /home/student/ansible.cfg
#修改ansible的配置文件，需要修改四行，直接复制粘贴即可
$ vim ansible.cfg
```

```
# line - 14 - inventory
inventory      = /home/student/ansible/inventory
# line - 68 - roles_path
roles_path    = /home/student/ansible/roles
# line - 71 - host_key_checking 取消主机密钥检查
host_key_checking = False
# line - 107 - remote_user 远程用户
remote_user = root
```

### Verify

```shell
#Verify_Time!!!
$ ansible all -m ping
# 输出的是绿色的就没有问题了 >>
172.25.250.11 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
172.25.250.12 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
172.25.250.13 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
172.25.250.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
172.25.250.9 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

## 2.创建和运行 Ansible 临时命令

作为系统管理员，您需要在受管节点上安装软件。

按下方所述，创建一个名为`/home/student/ansible/adhoc.sh`的`shell`脚本，该脚本将使用`Ansible`临时命令在各个受管节 点上安装`yum`存储库：

储存库 1：

1. 存储库的名称为 `EX294_BASE`
2. 描述为 `EX294 base software`
3. 基础`URL`为`http://content.example.com/rhel8.0/x86_64/dvd/BaseOS`
4. `GPG`签名检查为启用状态
5. `GPG`密钥`URL`为`http://content.example.com/materials/RPM-GPG-KEYexample`
6. 存储库为启用状态

储存库 1：

1. 存储库的名称为`EX294_STREAM`
2. 描述为`EX294 stream software`
3. 基础`URL`为`http://content.example.com/rhel8.0/x86_64/dvd/AppStream`
4. `GPG` 签名检查为启用状态
5. `GPG`密钥`URL`为`http://content.example.com/materials/RPM-GPG-KEYexample`
6. 存储库为启用状态

### Deploy

```shell
#记不住可以看这个文档
$ ansible-doc yum_repository
#按照要求创建一个脚本
$ vim /home/student/ansible/adhoc.sh
#添加下下面这些配置
```

```shell
#!/bin/bash
ansible all -m yum_repository -a \
'name="EX294_BASE" \
description="EX294 base software" \
baseurl="http://content.example.com/rhel8.0/x86_64/dvd/BaseOS" \
gpgcheck=yes \
gpgkey="http://content.example.com/materials/RPM-GPG-KEY-example" \
enabled=yes'
ansible all -m yum_repository -a \
'name="EX294_STREAM" \
description="EX294 stream software" \
baseurl="http://content.example.com/rhel8.0/x86_64/dvd/AppStream" \
gpgcheck=yes \
gpgkey="http://content.example.com/materials/RPM-GPG-KEY-example" \
enabled=yes'
```

```shell
#最好给脚本加上执行权限
$ chmod +x /home/student/ansible/adhoc.sh
#别忘了执行脚本
$ sh /home/student/ansible/adhoc.sh
#如果输出的内容是黄色的CHANGED就说明成功了
```

### Verify

```shell
#远程连接一台KVM，查看 /etc/yum.repos.d目录
$ ls /etc/yum.repos.d/
#看到对应的yum库就对了 >>
additional.repo  EX294_STREAM.repo  rhel_dvd.repo
EX294_BASE.repo  redhat.repo
#最好查看一下两个文件的内容是否符合要求
```

## 3.安装软件包

创建一个名为`/home/student/ansible/packages.yml`的`playbook`：

1. 将`php`和`mariadb`软件包安装到`dev`、`test`和`prod`主机组中的主机上
2. 将`RPM Development Tools`软件包组安装到`dev`主机组中的主机上
3. 将`dev`主机组中主机上的所有软件包更新为最新版本

### Deploy

```shell
#创建/home/student/ansible/packages.yml
$ vim /home/student/ansible/packages.yml
#添加下面的配置
#关于这个文件的配置说明
# - name: 配置名
#   hosts: 主机名
#   tasks: 任务的配置
#   - name: 任务名
#     yum: yum的配置
#       name: 软件包名称  -  {{ item }}表示循环使用loop内的包名
#       state: 安装的包的状态  -  latest表示最新
#     loop: 循环列表
#     - php  -  需要安装的包名
# 安装较长的包名应当使用双引号，并在包名前添加@ -> "@RPM Development Tools"
# 使用通配符*可以更新所有的包 -> "*"
```

```yaml
---
- name: install package
  hosts: dev,test,prod
  tasks:
  - name: install the latest version
    yum:
      name: "{{ item }}"
      state: latest
    loop:
    - php
    - mariadb
- name: install package group
  hosts: dev
  tasks:
    - name: install tools
      yum:
        name: "@RPM Development Tools"
        state: latest
    - name: upgrade package
      yum:
        name: "*"
        state: latest
```

```shell
#配置完成后就使用playbook执行
$ ansible-playbook /home/student/ansible/packages.yml
#可以看到输出
#>> changed: [172.25.250.11] => (item=php)
#>> ok: [172.25.250.9]
```

### Verify

```shell
#远程连接一台KVM，尝试安装之前的软件包
$ yum install php
#看到对应的这个包已经安装 >>
Nothing to do
```

## 4.使用`RHEL`系统角色

安装`RHEL`系统角色软件包，并创建符合以下条件的`playbook` `/home/student/ansible/timesync.yml`：

1. 在所有受管节点上运行
2. 使用`timesync`角色
3. 配置该角色，以使用当前有效的`NTP`提供商
4. 配置该角色，以使用时间服务器`172.20.20.254`
5. 配置该角色，以启用`iburst`参数

### Deploy

```shell
#安装服务
$ sudo yum -y install rhel-system-roles
#修改ansible的配置文件
$ vim ansible.cfg
#修改68行的roles_path,导入角色路径
roles_path    = /home/student/ansible/roles:/usr/share/ansible/roles
#查看ansible角色
$ ansible-galaxy list
#查看time文件的配置示列
$ cat /usr/share/doc/rhel-system-roles/timesync/example-timesync-playbook.yml
#复制文件
$ cp /usr/share/doc/rhel-system-roles/timesync/example-timesync-playbook.yml /home/student/ansible/timesync.yml
#编辑角色的配置文件
$ vim /home/student/ansible/timesync.yml
```

```yaml
# 修改文件
---
- hosts: all
  vars:
    timesync_ntp_servers:
      - hostname: 172.20.20.254
        iburst: yes
  roles:
    - rhel-system-roles.timesync
```

```shell
#执行文件
$ ansible-playbook timesync.yml
#可以看到输出
#>> 172.25.250.10              : ok=17   changed=4    unreachable=0    failed=0    skipped=18   rescued=0    ignored=6
```

### Verify

```shell
#远程连接一台KVM，查看chrony.conf配置
$ cat /etc/chrony.conf
#可以看到主机地址>>
server 172.20.20.254 iburst
```

## 5.使⽤`Ansible Galaxy`安装⾓⾊

创建需求文件`/home/student/ansible/roles/requirements.yml`，用来从以下`URL`下载角色，并安装到`/home/student/ansible/roles/`：

1. `http://content.example.com/materials/haproxy.tar.gz`此角色名为`balancer`
2. `http://content.example.com/materials/phpinfo.tar.gz`此角色名为`phpinfo`

### Deploy

```shell
#创建文件
$ vim /home/student/ansible/roles/requirements.yml
```

```yaml
---
- src: http://content.example.com/materials/haproxy.tar.gz
  name: balancer
- src: http://content.example.com/materials/phpinfo.tar.gz
  name: phpinfo
```

```shell
#运行playbook执行文件
$ ansible-galaxy install -r requirements.yml
$ cd ~/ansible
$ ansible-galaxy install -r roles/requirements.yml
#可以看到已经安装完成了>>
- balancers was installed successfully
- phpinfo was installed successfully
```

### Verify

```shell
#查看角色
$ ansible-galaxy list
#可以看到以下输出>>
# /home/student/ansible/roles
- balancers, (unknown version)
- phpinfo, (unknown version)
```

## 6.创建和使用角色

根据下列要求，在`/home/greg/ansible/roles`中创建名为`apache`的角色：

1.`httpd`软件包已安装，设为在系统启动时启用并启动
2.防火墙已启用并正在运行，并使用允许访问`Web`服务器的规则 
3.模板文件 `index.html.j2` 已存在，用于创建具有以下输出的文件`/var/www/html/index.html`： `Welcome to HOSTNAME on IPADDRESS`其中，`HOSTNAME`是受管节点的完全限定域名，`IPADDRESS`则是受管节点的`IP`地址。 
4.创建`playbook` `/home/greg/ansible/apache.yml`,使用`apache`的角色,在 `webservers`主机组。

### Deploy

```shell
#按照题目要求选中这个目录先
$ cd /home/student/ansible/roles
#创建角色apache
$ ansible-galaxy init apache
#>> - apache was created successfully
#编辑角色的任务列表
$ vim /home/student/ansible/roles/apache/tasks/main.yml
```

```yaml
---
- name: install latest httpd
  yum:
    name: httpd
    state: latest
- name: start httpd
  service:
    name: httpd
    state: started
    enabled: yes
- name: set firewalld
  firewalld:
    service: http
    permanent: yes
    state: enabled
    immediate: yes
- name: template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
```

```shell
#查看完全限定域名和地址
$ ansible all -m setup -a 'filter=*fqdn*'
$ ansible all -m setup -a 'filter=*ipv4*'
#编辑index.html.j2文件
$ vim /home/student/ansible/roles/apache/templates/index.html.j2
```

```html
Welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4.address }}
```

```shell
#创建playbook的yml配置文件 运行apache角色
$ vim /home/student/ansible/apache.yml
```

```yaml
---
- name: use apache
  hosts: webservers
  roles:
  - apache
```

```shell
#执行配置
$ ansible-playbook apache.yml
#执行完成后可以根据IP访问网页
$ curl 172.25.250.12
#>> Welcome to serverc.lab.example.com on 172.25.250.12
```

### Verify

```shell
#查看角色
$ ansible-galaxy list
#可以看到新创建的apache>> apache, (unknown version)
```

## 7.从`Ansible Galaxy`使用角色

根据下列要求，创建一个名为`/home/student/ansible/roles.yml`的`playbook`：

1. `playbook`中包含一个`play`，该`play`在`balancers`主机组中的主机上运行并将使用`balancer`角色。
   1. 此角色配置一项服务，以在`webservers`主机组中的主机之间平衡`Web`服务器请求的负载。
   2. 浏览到`balancers`主机组中的主机（例如 `http://node3.example.com/`）将生成以下输出：`Welcom to node3.example.com on 172.24.22.11`
   3. 重新加载浏览器将从另一 Web 服务器生成输出：`Welcom to node4.example.com on 172.24.22.12`
2. `playbook`中包含一个`play`， 该`play`在`webservers`主机组中的主机上运行并将使用`phpinfo`角色。
   1. 请通过`URL /hello.php`浏览到`webservers`主机组中的主机将生成以下输出：`Hello PHP World from FQDN`
   2. 其中，`FQDN`是主机的完全限定名称。

- 例如，浏览到`http://node3.example.com/hello.php`会生成以下输出：`Hello PHP World from node3.example.com`另外还有`PHP`配置的各种详细信息，如安装的`PHP`版本等。

- 同样，浏览到`http://node4.example.com/hello.php`会生成以下输出：`Hello PHP World from node4.example.com`另外还有`PHP`配置的各种详细信息，如安装的`PHP`版本等

### Deploy

```shell
#创建playbook运行的yaml文件
$ vim /home/student/ansible/roles.yml
```

```yaml
---
- name: start apache
  hosts: webservers
  roles:
  - apache
- name: start balancer
  hosts: balancers
  roles:
  - balancer
- name: start phpinfo
  hosts: webservers
  roles:
  - phpinfo
```

```shell
#执行
$ ansible-playbook /home/student/ansible/roles.yml
```

### Verify

```shell
#访问上述网址
$ curl 172.25.250.11
Welcome to serverb.lab.example.com on 172.25.250.11
$ curl 172.25.250.11
Welcome to serverc.lab.example.com on 172.25.250.12
#http://172.25.250.11/hello.php，浏览器打开网址，可以看到页面已经可以浏览了
```

## 8.创建和使⽤逻辑卷

创建一个名为`/home/student/ansible/lv.yml`的`playbook` ，它将在所有受管节点上运行以执行下列任务：

1. 创建符合以下要求的逻辑卷：
   1. 逻辑卷创建在`research`卷组中
   2. 逻辑卷名称为`data`
   3. 逻辑卷大小为`1500 MiB`
2. 使用`ext4`文件系统格式化逻辑卷
3. 如果无法创建请求的逻辑卷大小，应显示错误信息`Could not create logical volume of that size`，并且应改为使用大小 `800 MiB`。
4. 如果卷组`research`不存在，应显示错误信息`Volume group done not exist`。
5. 不要以任何方式挂载逻辑卷

```shell
#查看相关文档
$ ansible-doc lvol
#创建文件
$ vim /home/student/ansible/lv.yml
```

```yaml
---
- name: create lv
  hosts: all
  tasks:
  - block:
    - name: create lv of 1500m
      lvol:
        vg: research
        lv: data
        size: 1500
    - name: create ext4
      filesystem:
        fstype: ext4
        dev: /dev/research/data
    rescue:
    - debug:
        msg: Could not create logical volume of that size
    - name: create lv of 800m
      lvol:
        vg: research
        lv: data
        size: 800
      when: ansible_lvm.vgs.research is defined
      ignore_errors: yes
    - debug:
        msg: Volume group done not exist
      when: ansible_lvm.vgs.research is undefined
```

```shell
#执行
$ ansible-playbook /home/student/ansible/lv.yml
```

### Verify

```shell
#远程一台KVM
$ ls /dev/research/data
#如果打印出来就是成功了
```

## 9.生成主机文件

1. 将一个初始模板文件从`http://content.example.com/materials/hosts.j2`下载到`/home/student/ansible`
2. 完成该模板，以便用它生成以下文件：针对每个清单主机包含一行内容，其格式与`/etc/hosts`相同
3. 创建名为`/home/student/ansible/hosts.yml`的`playbook`，它将使用此模板在`dev` 主机组中的主机上生成文件`/etc/myhosts`。

该`playbook`运行后， `dev`主机组中主机上的文件`/etc/myhosts`应针对每个受管主机包含一行内容： 

```
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.250.9 workstation.example.com workstation
172.25.250.10 servera.lab.example.com servera
172.25.250.11 serverb.lab.example.com serverb
172.25.250.12 serverc.lab.example.com serverc
172.25.250.13 serverd.lab.example.com serverd
```

### Deploy

```shell
#下载模板文件
$ wget http://content.example.com/materials/hosts.j2
#修改模板文件
$ vim /home/student/ansible/hosts.j2
#变量示列如下，变量之间使用空格进行分隔
#{{ hostvars[host]['ansible_facts']['变量名'] }}
```

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['fqdn'] }} {{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %}
```

```shell
#创建playbook运行文件
$ vim /home/student/ansible/hosts.yml
```

```yaml
---
- name: create hosts
  hosts: all
  tasks:
  - name: template create
    template:
      src: /home/student/ansible/hosts.j2
      dest: /etc/myhosts
    when: '"dev" in group_names'
```

```shell
#执行文件
$ ansible-playbook /home/student/ansible/hosts.yml
#可以看到仅有一台主机设置了文件>>
skipping: [172.25.250.10]
skipping: [172.25.250.13]
skipping: [172.25.250.11]
skipping: [172.25.250.12]
ok: [172.25.250.9]
```

### Verify

```shell
#远程到设置了的主机，查看/etc/myhosts文件
$ cat /etc/myhosts
# 可以看到输出>>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.250.9 workstation.lab.example.com workstation
172.25.250.10 servera.lab.example.com servera
172.25.250.13 serverd.lab.example.com serverd
172.25.250.11 serverb.lab.example.com serverb
172.25.250.12 serverc.lab.example.com serverc
```

## 10.修改文件内容

修改文件内容 按照下方所述，创建一个名为`/home/student/ansible/issue.yml`的 `playbook`：

1. 该`playbook`将在所有清单主机上运行
2. 该`playbook`会将`/etc/issue`的内容替换为下方所示的一行文本：
   1. 在`dev`主机组中的主机上，这行文本显示 为：`Development`
   2. 在`test`主机组中的主机上，这行文本显示 为：`Test`
   3. 在`prod`主机组中的主机上，这行文本显示 为：`Production`

### Deploy

```shell
#创建文件
$ vim /home/student/ansible/issue.yml
```

```yaml
---
- name: change file
  hosts: all
  tasks:
  - name: change dev
    copy:
      content: 'Development'
      dest: /etc/issue
    when: 'inventory_hostname in groups.dev'
  - name: change test
    copy:
      content: 'Test'
      dest: /etc/issue
    when: 'inventory_hostname in groups.test'
  - name: change prod
    copy:
      content: 'Production'
      dest: /etc/issue
    when: 'inventory_hostname in groups.prod'
```

```shell
#执行playbook
$ ansible-playbook /home/student/ansible/issue.yml
```

### Verify

```shell
#远程一台主机查看 /etc/issue文件
$ cat /etc/issue
#可以看到输出了替换的文本
#>>
Test
```

## 11.创建`Web`内容目录

创建`Web`内容目录 按照下方所述，创建一个名为`/home/student/ansible/webcontent.yml`的`playbook`：

1. 该`playbook`在`dev`主机组中的受管节点上运行

2. 创建符合下列要求的目录`/webdev`：
   1. 所有者为`webdev`组 
   2. 具有常规权限：`owner=read+write+execute` `group=read+write+execute` `other=read+execute`
   3. 具有特殊权限：`SetGID`
3. 用符号链接将`/var/www/html/webdev`链接到`/webdev`

4. 创建文件`/webdev/index.html`，其中包含如下所示的单行文件：`Development`

### Deploy

```shell
#创建文件
$ vim /home/student/ansible/webcontent.yml
#一共三个任务，1创建目录 2创建链接 3创建文件
```

```yaml
---
- name: create Web
  hosts: dev
  tasks:
  - name: create dir
    file:
      path: /webdev
      state: directory
      group: webdev
      mode: '2775'
  - name: link
    file:
      src: /webdev
      dest: /var/www/html/webdev
      state: link
  - name: copy content
    copy:
      content: 'Development'
      dest: /webdev/index.html
      setype: httpd_sys_content_t
```

```shell
#执行playbook
$ ansible-playbook /home/student/ansible/webcontent.yml
```

### Verify

```shell
#远程到dev主机，查看新建的链接
$ ll /var/www/html/webdev*
#可以到创建了一个链接>>
lrwxrwxrwx. 1 root root 7 Apr  2 13:25 /var/www/html/webdev -> /webdev
```

## 12.生成硬件报告

生成硬件报告 创建一个名为`/home/student/ansible/hwreport.yml`的`playbook`，它将在所有受管节点上生成含有以下信息的输出文件`/root/hwreport.txt` ：

1. 清单主机名称
2. 以`MB`表示的总内存大小
3. `BIOS`版本
4. 磁盘设备`vda`的大小
5. 磁盘设备`vdb`的大小
6. 输出文件中的每一行含有一个`key=value`对。

您的`playbook`应当：

1.    从`http://content.example.com/materials/hwreport.empty`下载文件，并将它保存为`/root/hwreport.txt`
2.    使用正确的值写入到`/root/hwreport.txt`
3.    如果硬件项不存在，相关的值应设为`NONE`

### Deploy

```shell
#创建任务配置
$ vim /home/student/ansible/hwreport.yml
#文件的结构如下
#任务名
#执行的主机名:全部主机
#变量:需要替换的key和ansible变量中提取的value
#任务:下载模板文件、替换模板文件中的内容
```

```yaml
---
- name: hardware report
  hosts: all
  vars:
    hw_all:
    - hw_name: HOST
      hw_cont: "{{ inventory_hostname | default('NONE',true) }}"
    - hw_name: MEMORY
      hw_cont: "{{ ansible_memtotal_mb | default('NONE',true) }}"
    - hw_name: BIOS
      hw_cont: "{{ ansible_bios_version | default('NONE',true) }}"
    - hw_name: DISK_SIZE_VDA
      hw_cont: "{{ ansible_devices.vda.size | default('NONE',true) }}"
    - hw_name: DISK_SIZE_VDB
      hw_cont: "{{ ansible_devices.vdb.size | default('NONE',true) }}"
  tasks:
  - name: get template
    get_url:
      url: http://content.example.com/materials/hwreport.empty
      dest: /root/hwreport.txt
  - name: change file
    lineinfile:
      path: /root/hwreport.txt
      regexp: "^{{ item.hw_name }}="
      line: "{{ item.hw_name }}={{ item.hw_cont }}"
    loop: "{{ hw_all }}"
```

```shell
#执行playbook
$ ansible-playbook /home/student/ansible/hwreport.yml
#全部主机都会执行一遍
```

### Verify

```shell
#远程一台主机，查看/root/hwreport.txt
$ cat /root/hwreport.txt
#>>
# Hardware report
HOST=172.25.250.9
MEMORY=1829
BIOS=1.11.1-3.module+el8+2529+a9686a4d
DISK_SIZE_VDA=10.00 GB
DISK_SIZE_VDB=NONE
```

## 13.创建密码库

按照下方所述，创建一个`Ansible`库，来存储用户密码： 

1. 库名称为`/home/student/ansible/locker.yml`
2. 库中含有两个变量，名称如下： 
   1. `pw_developer`，值为`Imadev`
   2. `pw_manager`，值为`Imamgr`
3. 用于加密和解密该库的密码为`redhat`
4.  密码存储在文件`/home/student/ansible/secret.txt`中

```shell
#创建文件保存两个变量
$ vim /home/student/ansible/locker.yml
```

```yaml
---
pw_developer: Imadev
pw_manager: Imamgr
```

```shell
#创建保存密码的文件
$ vim /home/student/ansible/secret.txt
```

```txt
redhat
```

```shell
#编辑ansible配置文件，使两个文件进行关联
$ vim /home/student/ansible/ansible.cfg
#修改vault所在的行 - 140line
#将其中的文件路径修改为密钥的路径
```

```shell
vault_password_file = /home/student/ansible/secret.txt
```

```shell
#加密文件
$ ansible-vault encrypt locker.yml
#>>
Encryption successful
```

### Verify

```shell
#查看加密后的文件
$ cat locker.yml
#变成了不可读字符>>
$ANSIBLE_VAULT;1.1;AES256
37313263396463393330343933613062313963373235336661616632343431303936663866393363
3961666463383862646533353766333731643565326365390a376435303834616431303064396363
63333464623838366331386133643531383332383130656133643534643236366563316538336532
3138356364316362310a343266313930363432336265393964666161616266383262373231653930
64613561353930646536376665343666623430343834643265376263623662363233396139613433
3133656235613938646561383664306261633732383061623638
```

## 14.创建用户帐户

1. 从`http://content.example.com/materials/user_list.yml`下载要创建的用户的列表，并将它保存到`/home/student/ansible`
2. 在本次考试中使用在其他位置创建的密码库`/home/student/ansible/locker.yml`。创建名为`/home/student/ansible/users.yml`的`playbook`，从而按以下所述创建用户帐户：
   1. 职位描述为`developer`的用户应当：
      1. 在`dev`和`test`主机组中的受管节点上创建
      2. 从`pw_developer`变量分配密码
      3. 是附加组`devops`的成员
   2. 职位描述为`manager`的用户应当：
      1. 在`prod`主机组中的受管节点上创建
      2. 从`pw_manager`变量分配密码
      3. 是附加组`opsmgr`的成员
3. 密码采用`SHA512`哈希格式。
4. 您的`playbook`应能够在本次考试中使用在其他位置创建的库密码文件 `/home/student/ansible/secret.txt`正常运行

### Deploy

```shell
#下载用户列表
$ wget http://content.example.com/materials/user_list.yml
#创建playbook文件
$ vim /home/student/ansible/users.yml
#文件的结构如下
#任务名
#执行主机:dev,test
#变量文件:
#- 引入的的加密文件
#- 引入的用户列表
#任务:
#- 任务名
#  组模块:
#    创建的组名:devops
#    状态:present
#  用户模块
#
#
```

```yaml
---
- name: create job developer
  hosts: dev,test
  vars_files:
  - /home/student/ansible/locker.yml
  - /home/student/ansible/user_list.yml
  tasks:
  - name: create group devops
    group:
      name: devops
      state: present
  - name: create user for dev and test
    user:
      name: "{{ item.name }}"
      password: "{{ pw_developer | password_hash('sha512') }}"
      group: devops
    loop: "{{ users }}"
    when: item.job == "developer"
- name: create job manager
  hosts: prod
  vars_files:
  - /home/student/ansible/locker.yml
  - /home/student/ansible/user_list.yml
  tasks:
  - name: create group opsmgr
    group:
      name: opsmgr
      state: present
  - name: create user for prod
    user:
      name: "{{ item.name }}"
      password: "{{ pw_manager | password_hash('sha512') }}"
      group: opsmgr
    loop: "{{ users }}"
    when: item.job == "manager"
```

```shell
#执行playbook
$ ansible-playbook /home/student/ansible/users.yml
#可以到输出了创建的用户
```

### Verify

```shell
#远程一台主机，查看新创建的用户
tail -n 2 /etc/passwd
#可以看到新创建的用户>>
tom:x:1002:1001::/home/tom:/bin/bash
tim:x:1003:1001::/home/tim:/bin/bash
```

## 15.更新`Ansible`库的密钥

按照下方所述，更新现有 Ansible 库的密钥：

1. 从`http://content.example.com/materials/expense.yml`下载`Ansible`库到`/root/student/ansible`

2. 当前的库密码为`redhat`

3. 新的库密码为`important`

4. 库使用新密码保持加密状态

### Deploy

```shell
#下载文件
$ wegt http://content.example.com/materials/expense.yml
#修改密码
$ ansible-vault rekey --ask-vault-pass expense.yml
#输入旧的密码，更新新的密码>>
Vault password:
New Vault password:
Confirm New Vault password:
Rekey successful
```

### Verify

```shell
#查看加密文件
$ ansible-vault view salaries.yml
```
