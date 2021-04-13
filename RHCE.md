# RHCE

| VM Name       | IP addresses     | Ansible Role           | Domain Name           |
| ------------- | ---------------- | ---------------------- | --------------------- |
| `bastion`     | `172.25.250.254` | `ansible control node` | `ansible.example.com` |
| `workstation` | `172.25.250.9`   | `ansible managed node` | `node1.example.com`   |
| `servera`     | `172.25.250.10`  | `ansible managed node` | `node2.example.com`   |
| `serverb`     | `172.25.250.11`  | `ansible managed node` | `node3.example.com`   |
| `serverc`     | `172.25.250.12`  | `ansible managed node` | `node4.example.com`   |
| `serverd`     | `172.25.250.13`  | `ansible managed node` | `node5.example.com`   |

所有的`root`用户的密码为`redhat`

## 1.安装与配置 `Ansible`

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

<details>
<summary>Command</summary>
<pre>
<code>
$ sudo yum install -y ansible
$ mkdir -p /home/student/ansible/roles
$ cd ./ansible
$ vim inventory
</code>
</pre>
</details>

<details>
<summary>inventory</summary>
<pre>
<code>
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
[blancers]
172.25.250.13
[webservers:children]
prod
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ cp /etc/ansible/ansible.cfg ./
$ vim ansible.cfg
</code>
</pre>
</details>

<details>
<summary>ansible.cfg</summary>
<pre>
<code>
inventory      = /home/student/ansible/inventory
roles_path    = /home/student/ansible/roles
host_key_checking = False
remote_user = root
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ ansible all -m ping
</code>
</pre>
</details>

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

<details>
<summary>Command</summary>
<pre>
<code>
$ vim adhoc.sh
</code>
</pre>
</details>

<details>
<summary>adhoc.sh</summary>
<pre>
<code>
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ chmod +x adhoc.sh
$ sh adhoc.sh
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ ls /etc/yum.repo.d/
</code>
</pre>
</details>

## 3.安装软件包

创建一个名为`/home/student/ansible/packages.yml`的`playbook`：

1. 将`php`和`mariadb`软件包安装到`dev`、`test`和`prod`主机组中的主机上
2. 将`RPM Development Tools`软件包组安装到`dev`主机组中的主机上
3. 将`dev`主机组中主机上的所有软件包更新为最新版本

<details>
<summary>Command</summary>
<pre>
<code>
$ vim packages.yml
</code>
</pre>
</details>

<details>
<summary>packages.yml</summary>
<pre>
<code>
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook packages.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
# yum install php mariadb
</code>
</pre>
</details>

## 4.使用`RHEL`系统角色

安装`RHEL`系统角色软件包，并创建符合以下条件的`playbook` `/home/student/ansible/timesync.yml`：

1. 在所有受管节点上运行
2. 使用`timesync`角色
3. 配置该角色，以使用当前有效的`NTP`提供商
4. 配置该角色，以使用时间服务器`172.20.20.254`
5. 配置该角色，以启用`iburst`参数

<details>
<summary>Command</summary>
<pre>
<code>
sudo yum install rhel-system-roles -y
vim ansible.cfg
</code>
</pre>
</details>

<details>
<summary>ansible.cfg</summary>
<pre>
<code>
roles_path    = /home/student/ansible/roles:/usr/share/ansible/roles
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ cp /usr/share/doc/rhel-system-roles/timesync/example-timesync-playbook.yml timesync.yml
$ vim timesync.yml
</code>
</pre>
</details>

<details>
<summary>timesync.yml</summary>
<pre>
<code>
---
- hosts: all
  vars:
    timesync_ntp_servers:
      - hostname: 172.20.20.254
        iburst: yes
  roles:
    - rhel-system-roles.timesync
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook timesync.yml
</code>
</pre>
</details>

## 5.使⽤`Ansible Galaxy`安装⾓⾊

创建需求文件`/home/student/ansible/roles/requirements.yml`，用来从以下`URL`下载角色，并安装到`/home/student/ansible/roles/`：

1. `http://content.example.com/materials/haproxy.tar.gz`此角色名为`balancer`
2. `http://content.example.com/materials/phpinfo.tar.gz`此角色名为`phpinfo`

<details>
<summary>Command</summary>
<pre>
<code>
$ cd roles/
$ vim requirements.yml
</code>
</pre>
</details>

<details>
<summary>requirements.yml</summary>
<pre>
<code>
---
- src: http://content.example.com/materials/haproxy.tar.gz
  name: balancer
- src: http://content.example.com/materials/phpinfo.tar.gz
  name: phpinfo
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-galaxy install -r requirements.yml
$ cd ..
$ ansible-galaxy install -r roles/requirements.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ ansible-galxy list
</code>
</pre>
</details>

## 6.创建和使用角色

根据下列要求，在`/home/student/ansible/roles`中创建名为`apache`的角色：

1. `httpd`软件包已安装，设为在系统启动时启用并启动
2. 防火墙已启用并正在运行，并使用允许访问`Web`服务器的规则 
3. 模板文件 `index.html.j2` 已存在，用于创建具有以下输出的文件`/var/www/html/index.html`： `Welcome to HOSTNAME on IPADDRESS`其中，`HOSTNAME`是受管节点的完全限定域名，`IPADDRESS`则是受管节点的`IP`地址。 
4. 创建`playbook` `/home/student/ansible/apache.yml`,使用`apache`的角色,在 `webservers`主机组。

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-galaxy init apache
$ vim apache/tasks/main.yml
</code>
</pre>
</details>

<details>
<summary>main.yml</summary>
<pre>
<code>
---
- name: Install Latest httpd
  yum:
    name: httpd
    state: latest
- name: Start httpd
  service:
    name: httpd
    state: started
    enabled: yes
- name: Set Firewalld
  firewalld:
    service: http
    permanent: yes
    state: enabled
    immediate: yes
- name: Template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ vim roles/apache/templates/index.html.j2
</code>
</pre>
</details>

<details>
<summary>index.html.j2</summary>
<pre>
<code>
Welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4.address }}
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ vim apache.yml
</code>
</pre>
</details>

<details>
<summary>apache.yml</summary>
<pre>
<code>
---
- name: Use Apache
  hosts: webservers
  roles:
  - apache
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook apache.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ ansible-galaxy list
</code>
</pre>
</details>

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

<details>
<summary>Command</summary>
<pre>
<code>
$ vim roles.yml
</code>
</pre>
</details>

<details>
<summary>roles.yml</summary>
<pre>
<code>
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook roles.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ curl [你的webservers主机]
$ curl [你的balancers主机]
</code>
</pre>
</details>

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

<details>
<summary>Command</summary>
<pre>
<code>
$ vim lv.yml
</code>
</pre>
</details>

<details>
<summary>lv.yml</summary>
<pre>
<code>
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook lv.yml
</code>
</pre>
</details>

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

<details>
<summary>Command</summary>
<pre>
<code>
$ wget http://content.example.com/materials/hosts.j2
$ vim hosts.j2
</code>
</pre>
</details>

<details>
<summary>hosts.j2</summary>
<pre>
<code>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for host in groups['all'] %}
{{ hostvars[host].ansible_default_ipv4.address }} {{ hostvars[host].ansible_fqdn }} {{ hostvars[host].ansible_hostname }}
{% endfor %}
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ vim hosts.yml
</code>
</pre>
</details>

<details>
<summary>hosts.yml</summary>
<pre>
<code>
---
- name: create hosts
  hosts: all
  tasks:
  - name: template create
    template:
      src: /home/student/ansible/hosts.j2
      dest: /etc/myhosts
    when: '"dev" in group_names'
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook hosts.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ cat /etc/myhosts
</code>
</pre>
</details>

## 10.修改文件内容

修改文件内容 按照下方所述，创建一个名为`/home/student/ansible/issue.yml`的 `playbook`：

1. 该`playbook`将在所有清单主机上运行
2. 该`playbook`会将`/etc/issue`的内容替换为下方所示的一行文本：
   1. 在`dev`主机组中的主机上，这行文本显示 为：`Development`
   2. 在`test`主机组中的主机上，这行文本显示 为：`Test`
   3. 在`prod`主机组中的主机上，这行文本显示 为：`Production`

<details>
<summary>Command</summary>
<pre>
<code>
$ vim issue.yml
</code>
</pre>
</details>

<details>
<summary>issue.yml</summary>
<pre>
<code>
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook issue.yml
</code>
</pre>
</details>

## 11.创建`Web`内容目录

创建`Web`内容目录 按照下方所述，创建一个名为`/home/student/ansible/webcontent.yml`的`playbook`：

1. 该`playbook`在`dev`主机组中的受管节点上运行

2. 创建符合下列要求的目录`/webdev`：
   1. 所有者为`webdev`组 
   2. 具有常规权限：`owner=read+write+execute` `group=read+write+execute` `other=read+execute`
   3. 具有特殊权限：`SetGID`
3. 用符号链接将`/var/www/html/webdev`链接到`/webdev`

4. 创建文件`/webdev/index.html`，其中包含如下所示的单行文件：`Development`

<details>
<summary>Command</summary>
<pre>
<code>
$ vim webcontent.yml
</code>
</pre>
</details>

<details>
<summary>webcontent.yml</summary>
<pre>
<code>
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook webcontent.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ ll /var/www/html/
$ cat /var/www/html/webdev/index.html
</code>
</pre>
</details>

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

<details>
<summary>Command</summary>
<pre>
<code>
$ vim hwreport.yml
</code>
</pre>
</details>

<details>
<summary>hwreport.yml</summary>
<pre>
<code>
---
- name: hardware report
  hosts: all
  vars:
    hw_all:
    - name: HOST
      cont: "{{ inventory_hostname | default('NONE',true) }}"
    - name: MEMORY
      cont: "{{ ansible_memtotal_mb | default('NONE',true) }}"
    - name: BIOS
      cont: "{{ ansible_bios_version | default('NONE',true) }}"
    - name: DISK_SIZE_VDA
      cont: "{{ ansible_devices.vda.size | default('NONE',true) }}"
    - name: DISK_SIZE_VDB
      cont: "{{ ansible_devices.vdb.size | default('NONE',true) }}"
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook hwreport.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ cat hwreport.txt
</code>
</pre>
</details>

## 13.创建密码库

按照下方所述，创建一个`Ansible`库，来存储用户密码： 

1. 库名称为`/home/student/ansible/locker.yml`
2. 库中含有两个变量，名称如下： 
   1. `pw_developer`，值为`Imadev`
   2. `pw_manager`，值为`Imamgr`
3. 用于加密和解密该库的密码为`redhat`
4. 密码存储在文件`/home/student/ansible/secret.txt`中

<details>
<summary>Command</summary>
<pre>
<code>
$ vim locker.yml
</code>
</pre>
</details>

<details>
<summary>locker.yml</summary>
<pre>
<code>
---
pw_developer: Imadev
pw_manager: Imamgr
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ vim secret.txt
</code>
</pre>
</details>

<details>
<summary>secret.txt</summary>
<pre>
<code>
redhat
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ vim ansible.cfg
</code>
</pre>
</details>

<details>
<summary>ansible.cfg</summary>
<pre>
<code>
140 vault_password_file = /home/student/ansible/secret.txt
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-vault encrypt locker.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ cat locker.yml
$ ansible-vault view locker.yml
</code>
</pre>
</details>

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

<details>
<summary>Command</summary>
<pre>
<code>
$ wget http://content.example.com/materials/user_list.yml
$ vim users.yml
</code>
</pre>
</details>

<details>
<summary>users.yml</summary>
<pre>
<code>
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
</code>
</pre>
</details>

<details>
<summary>Command</summary>
<pre>
<code>
$ ansible-playbook users.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ grep devops /etc/group
$ tail -n 2 /etc/passwd
$ grep opsmgr /etc/group
$ tail -n 2 /etc/passwd
</code>
</pre>
</details>

## 15.更新`Ansible`库的密钥

按照下方所述，更新现有 Ansible 库的密钥：

1. 从`http://content.example.com/materials/expense.yml`下载`Ansible`库到`/root/student/ansible`

2. 当前的库密码为`redhat`

3. 新的库密码为`important`

4. 库使用新密码保持加密状态

<details>
<summary>Command</summary>
<pre>
<code>
$ wegt http://content.example.com/materials/expense.yml
$ ansible-vault rekey --ask-vault-pass expense.yml
</code>
</pre>
</details>

<details>
<summary>Verify</summary>
<pre>
<code>
$ ansible-vault view salaries.yml
</code>
</pre>
</details>
