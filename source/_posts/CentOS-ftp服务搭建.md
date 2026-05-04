---
title: CentOS ftp服务搭建
date: 2017-05-04 11:33:03
categories: Linux
tags: [Linux,CentOS,ftp]
---

> 好记性不如烂笔头

每次想要windows和linux互传文件时，都得去搜索*“Linux ftp服务安装配置”*，重复的次数太多，还是把它记录下来吧！！！

1. 安装vsftpd ftp

  ```bash
  yum -y install ftp vsftpd
  ```

1. 备份vsftpd原有的配置文件

  ```bash
  cd /etc/vsftpd/
  cp vsftpd.conf vsftpd.conf.origin
  ```

1. 创建密码明文文件

  ```bash
  vim /etc/vsftpd/vftpuser.txt
  # brucewar
  # password
  ```

1. 根据明文创建密码DB文件

  ```bash
  db_load -T -t hash -f /etc/vsftpd/vftpuser.txt /etc/vsftpd/vftpuser.db
  ```

1. 查看密码数据文件

  ```bash
  file /etc/vsftpd/vftpuser.db
  # /etc/vsftpd/vftpuser.db: Berkeley DB(Hash,version9,native byte-order)
  ```

1. 创建vftpd的guest账户

  ```bash
  useradd -d /ftp/private -s /sbin/nologin vftpuser
  ```

  **Note**：这一步可能会创建*/ftp/private*失败，可以手动创建文件夹，并将它的权限赋予vftpuser

  ```bash
  mkdir -p /ftp/private
  chown -R vftpuser.vftpuser /ftp
  ```

1. 打开*/etc/pam.d/vsftpd*, 将`auth`及`account`的所有配置行都注释掉，添加如下内容：

  ```bash
  auth required pam_userdb.so db=/etc/vsftpd/vftpuser
  account required pam_userdb.so db=/etc/vsftpd/vftpuser
  ```

1. 打开*/etc/vsftpd/vsftpd.conf*，将`anonymous_enable=YES`改为`anonymous_enable=NO`，在最下面添加如下内容：

  ```bash
  virtual_use_local_privs=YES
  guest_enable=YES
  guest_username=vftpuser
  chroot_local_user=YES
  allow_writeable_chroot=YES
  ```

1. 设置vsftpd开机启动

  ```bash
  systemctl enable vsftpd
  ```

1. 重启vsftpd服务

  ```bash
  systemctl restart vsftpd
  ```

1. 配置防火墙和SELinux

  ```bash
  firewall-cmd --permanent --zone=public --add-service=ftp
  firewall-cmd --reload

  getsebool -a | grep ftp
  setsebool -P ftpd_full_access on
  ```

1. 查看vsftpd服务状态

  ```bash
  systemctl status vsftpd
  ```
