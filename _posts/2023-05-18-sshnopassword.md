

---
layout: post
title: ssh免密配置
---

{{ page.title }}
================

<p class="meta">18 May 2023 - HangZhou</p>

# root权限

- 能使用22默认端口

- 能修改.ssh下文件

直接配置 [tools: 1 - Gitee.com](https://gitee.com/blicerain/tools/tree/master/auto_ssh)可以使用该脚本配置

# 非root权限

- 不能使用22端口

宿主机 22 端口无法访问，故修改容器 sshd 服务的默认端口，使其与宿主机区分开

## 自定义ssh服务

```shell
cd ~
HOME='pwd'

# setup sshd dir
mkdir -p $(HOME)/etc
ssh-keygen -f $(HOME)/etc/ssh_host_rsa_key -N '' -t rsa
mkdir -p $(HOME)/etc/ssh $(HOME)/etc/var/run

# setup sshd config(listen at 2222 port)
cat <<EOF > $(HOME)/etc/ssh/sshd_config
Port 2222
HostKey $(HOME)/etc/ssh_host_rsa_key
AuthorizedKeysFile $(HOME)/etc/.ssh/authorized_keys
PidFile $(HOME)/etc/var/run/sshd.pid
StrictModes no
UsePAM no
EOF
```

## 不能修改已生成的.ssh文件

但是.ssh/config里面已经配置StrictHostKeyChecking no

```shell
# cat rsa_pub to authorized_keys
cat (HOME)/.ssh/id_rsa.pub >> (HOME)/etc/.ssh/authorized_keys
```

## 能修改.ssh文件

```shell
# generate ssh key
ssh-keygen -t rsa -f ${HOME}/.ssh/id_rsa -P ''
cat ${HOME}/.ssh/id_rsa.pub >> ${HOME}/etc/.ssh/authorized_keys

# disable ssh host key checking for all hosts
cat <<EOF > ${HOME}/.ssh/config
Host *
   StrictHostKeyChecking no
EOF
```

# 测试ssh免密登录

## 启动sshd服务

```shell
$(which sshd) -f $(HOME)/etc/ssh/sshd_config
```

## 确认启动

```shell
ps -ef | grep sshd | grep -v grep
work         932       0  0 09:57 ?        00:00:00 /usr/sbin/sshd -f /home/work/etc/ssh/sshd_config

netstat -anp | grep 2222
tcp        0      0 0.0.0.0:2222            0.0.0.0:*               LISTEN      932/sshd
```

## 测试免密登录

```
root@abc:~$ ssh -p 2222 127.0.0.1
Warning: Permanently added '[127.0.0.1]:2222' (RSA) to the list of known hosts
root@abc:~$ exit
logout
Connection to 127.0.0.1 closed
```
