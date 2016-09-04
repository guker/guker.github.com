---
layout: post
categories: [tech]
share: true
comments: true
title: "搭建Git服务器教程"
date: '2016-09-04'
tags: [linux,git]
author: hqwsky
---  

### Server
<pre>
1. 新建linux用户，命名为git
    useradd git    # 默认工作目录在/home/git,shell值为bash
    passwd  git    # 设置密码并确认

2. 在git用户下新建目录.ssh
    cd  /home/git/
    mkdir .ssh  

3. 在/home/git/.ssh/目录中新建一个authorized_keys文件,此文件
   用于保存客户端提供的公钥(id_rsa.pub),只需追加到文件尾即可
    touch authorized_keys    # 创建文件，或者 vi authorized_keys
    cat id_rsa.pub >> authorized_keys  # 追加到文件尾 

4. 创建一个git裸仓库，假设当前路径为/home/git/
    cd /home/git
    git init --bare project.git  # --bare表示创建裸仓库，没有工作目录，只是用来共享
    
   或者
    cd /home/git
    mkdir project.git
    cd project.git
    git init --bare     # 同上述创建仓库的方式相同
   创建完毕后，会产生一下文件：
    [hqwsky@localhost project.git]$ ls
    branches  config  description  HEAD  hooks  info  objects  refs

5. 此时若project.git的所有者不是用户git，需要变更所有者与用户组,需要root权限
    sudo  chown -R  git：git  project.git
    sudo  chown -R  git:git   /home/git/.ssh
   这样的目的保证用户git对其具有读写执行的权限。
   当然，如果用户git的shell没有被禁用，可以直接登录到git的shell，创建上述的仓库以及
   .ssh就不用更改用户，因为默认是所有者为创建者。

6. 为了安全考虑，禁用用户git的shel登录
    sudo vi /etc/passwd    
   找到如下：
    git:x:533:533::/home/git:/bin/bash # 用户git的工作目录/home/git,shell为bash
   改为：
    git:x:533:533::/home/git:/usr/bin/git-shell 
   这样用户git可以正常通过ssh使用git,但无法登陆shell,每次登录都会自动退出。 

7. git服务器打开RSA认证
    sudo vi /etc/ssh/sshd_config
   可以找到被注释的几行
    #RSAAuthentication yes
    #PubkeyAuthentication yes
    #AuthorizedKeysFile     .ssh/authorized_keys
   取消注释即可
</pre>





scp

 
如何无密码访问git服务器
ssh


git 如何修改ssh端口 .ssh必须在用户的工作空间 .ssh/config

修改RSA认证 


