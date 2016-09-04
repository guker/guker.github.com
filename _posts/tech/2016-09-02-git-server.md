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

### 服务器端
<pre>
1. 新建linux用户，命名为git
    useradd git    # 默认工作目录在/home/git,shell值为bash
    passwd  git    # 设置密码并确认

2. git的数据交互是基于ssh协议的，所以在git用户下新建目录.ssh
    cd  /home/git/
    mkdir .ssh  

3. 在/home/git/.ssh/目录中新建一个authorized_keys文件,此文件用于保存客户端提供的
  公钥(id_rsa.pub),只需追加到文件尾即可
    touch authorized_keys    # 创建文件，或者 vi authorized_keys
    cat id_rsa.pub >> authorized_keys  # 追加到文件尾 

4. 创建一个git裸仓库，假设当前路径为/home/git/
    cd /home/git
    git init --bare project.git  # --bare表示创建裸仓库
    
   或者
    cd /home/git
    mkdir project.git
    cd project.git
    git init --bare     # 同上述创建仓库的方式相同
   创建完毕后，会产生一下文件：
    [hqwsky@localhost project.git]$ ls
    branches  config  description  HEAD  hooks  info  objects  refs
   裸仓库没有工作目录，只能push或pull,本地不能进行commit。

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
   这样用户git可以正常通过ssh使用git,但无法登陆shell,每次登录都会自动退出。这样，用户
   git只能用ssh连接来推送和获取git仓库，而不能直接使用主机shell。
   NOTE：git-shell的位置
    which git-shell   

7. git服务器打开RSA认证
    sudo vi /etc/ssh/sshd_config
   可以找到被注释的几行
    #RSAAuthentication yes
    #PubkeyAuthentication yes
    #AuthorizedKeysFile     .ssh/authorized_keys
   取消注释即可
</pre>

#### 客户端
<pre>
1. 在工作目录下创建.ssh目录，生成公钥
    mkdir .ssh
    cd .ssh
    ssh-keygen -t rsa   # 一路回车即可
   生成两个文件id_rsa文件(密钥)和id_rsa.pub(公钥)
   接下来，把客户端的ssh公钥添加到用户authorized_keys文件中
    scp  ~/.ssh/id_rsa.pub git@yourservername:~/.ssh/  # scp将公钥复制到服务器
    ssh git@yourservername   # ssh登录到服务器，此处的yourservername可以是域名或者ip
    cd ~/.ssh
    cat id_rsa.pub >> authorized_keys   # 追加到authorized_keys
   注：
   1）. 当使用ssh连接到git服务器，会提示连接不能被建立，并生成git服务器的RSA key，然后问你
   是否继续机建立连接,yes之后，会在.ssh目录下生成known_hosts文件，并把git服务器的RSA key
   写入known_hosts文件中。
   2）. 一般ssh的默认端口为22，为了安全考虑，服务器会修改ssh的默认端口号，直接连接会出现超时
   此时，修改客户端的ssh配置
    cd ~/.ssh
    vi config   # 若没有config文件，新建即可
   添加如下
    Port   xxx   # xxx为服务器的ssh端口号
   有时，会出现 Bad owner or permissions on $HOME/.ssh/config
    chmod 600 config   # 这个提示看起来有点傻逼，可能是出于对安全的考虑

2. 本地新建git仓库
    git init  #初始化为git仓库，目录下出出现一个隐藏文件.git,类似于小型的数据库

3. 新建文件并推送到服务器
    vi readme.txt
    git add --a
    git commit -m "comment"
    git remote add origin git@xxx.xxx.xxx.xxx:/home/git/project.git
    git push origin master
   这样你的项目就推送到服务器上了。
   如果另一个开发者参与进来，只需要把他的公钥加进来即可，然后克隆一下
    git clone git@xxx.xxx.xxx.xxx:/home/git/project.git
   注意每次push之前，应该先git pull,这样可以避免冲突。

   当然也可以这样做，直接克隆服务器上的裸仓库
    git clone git@xxx.xxx.xxx.xxx:/home/git/project.git
   此时不需要指定远端的仓库为origin，默认为origin,此外，如果当时第一次通过ssh连接git服务器
   也会提示连接不能被建立，并生成git服务器的RSA key，然后问你是否继续机建立连接,yes之后，
   会在.ssh目录下生成known_hosts文件，并把git服务器的RSA key写入known_hosts文件中，下次连接，会
   跳过警告

</pre>

#### 说明
<pre>
   关于git服务器的ssh的说明
   用户git的下``~/.ssh/authorized_keys`` 为受信任列表，此列表可以添加多个ssh客户端的公钥
   当向git服务器添加自己的ssh Public Key(id_RSA.pub)后，服务器便把客户端关联起来，这样，
   客户端可以做到git服务器免密码push以及pull等操作。

   具体ssh免密码的原理如下：
   ssh客户端提前将ssh公钥储存在远程ssh服务器上，然后ssh客户端携带公钥向远程ssh服务器（known_hosts）
   发起登录请求。
   远程ssh服务器收到该请求之后，先在该服务器上的authorized_keys寻找你上传授权过的公钥，然后把它和你
   发送过来的公钥进行比较。
   如果两个公钥一致（Key Exchange Success），远程SSH服务器会向用户发送一段使用ssh公钥加密过的随机
   字符串进行身份质询（Challenge）。
   SSH客户端用自己的私钥解密后再发回给远程ssh服务器，远程ssh服务器对比回包中解密出来的随机字符串是否
   一致。如果一致，则证明用户（公钥或身份）是可信的，直接允许登录shell，不再要求密码

   常见问题
   在搭建git服务器的时候，遇到如下问题：
   在客户端用`ssh-keygen -t rsa`生成密钥和公钥，并将生成id_rsa.pub公钥内容追加到服务器
   的`~/.ssh/authorized_keys`中，并且重启服务器的sshd服务，但是客户端采用ssh连接服务器
   依然需要输入密码。
   
   经过google之后，在ssh连接的时候，加-vvT查看输出，进行检查ssh连接的详细过程,一般出现此
   问题都是权限的问题，将.ssh的权限改为700,authorized_keys权限改为600即可
   `ssh username@xxx.xxx.xxx.xxx`  
   可以免密码建立连接,如果username支持shell，则可以登录到其工作目录下
</pre>

#### 参考资料
[git book](https://git-scm.com/book/zh/v2/)  
[git服务器的建立——Git折腾小记](http://blog.csdn.net/xsl1990/article/details/25486211)

