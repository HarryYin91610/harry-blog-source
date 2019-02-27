---
title: 个人网站部署笔记
date: 2019-02-19 17:18:00
tags:
- ECS 
- 域名
- ssh
- nodejs
- nginx
- 防火墙
- DNS
- MongoDB 
- pm2

categories:
- Deploy

---

## 搭建流程
### 1 购买域名
* 厂商选择：阿里云（万网）、爱名网、dnspod，godaddy。为方便备案最终选择阿里云的域名；
* 搜索想要的域名前缀，选择纯英文的域名；
* 后缀优先选择： .cn， .com， .cc；
* 域名需要实名认证。

<!--more-->

--------------------------------------------

### 2 购买服务器
* 厂商选择：亚马逊AWS，阿里云ECS，Linode，Heroku，DigitOcean，青云，UCloud等；
* 同样为方便备案选择国内的阿里云；
* 公共镜像：Ubuntu 14.04 64位；
* 设置root密码

--------------------------------------------

### 3 域名备案
* 在阿里云直接进行备案，备案完成前首次备案的域名在线上是不允许访问的；
* 首先申请备案服务号；
* [完成备案流程](https://beian.aliyun.com/?spm=a3c00.7621333.1280361.5.67c6kaR0kaR0ch)

--------------------------------------------

### 4 ssh远程登录服务器
#### 4.1 root登录
* 在mac终端，使用root身份登录：  
  >a. 输入：ssh root@服务器公网ip；
  >b. 输入之前在服务器设置的密码；
  >c. 出现欢迎信息即完成登录。

* 查看硬盘使用的命令
```
df -h
```
* 退出登录
```
ctrl + d
```

#### 4.2 添加用户配置权限管理服务器
* 添加用户
  >a. 输入：adduser 用户名；  
  >b. 输入密码及确认密码；  
  >c. 输入用户信息（可不填）
* 对用户进行授权（用户可以通过sudo命令并输入密码来使用权限）
```
gpasswd -a 用户名 sudo
```
* 输入
```
sudo visudo
```
* 在vim界面User privilege specification下加一行
```
# User privilege specification
用户名 ALL=(ALL:ALL) ALL
```
* 输入下面命令保存配置
```
ctrl + x
shift + y 并回车。
```
* 重启ssh命令
```
service ssh restart
```
* 用户登录命令
```
ssh 用户名@服务器ip
```

#### 4.3 ssh配置无密码登录
##### 4.3.1 在本地终端
* 进入.ssh文件夹
```
cd ~/.ssh
```
* 查看是否有id_rsa和id_rsa.pub
```
ls
```
* 若没有，则继续下面的命令
```
cd ..
ssh-keygen -t rsa -b 4096 -C "一个邮箱地址"
```
* 连续回车，生成公钥和私钥
* 进入.ssh，打印公钥内容
```
cd .ssh
cat id_rsa.pub
```
* 运行ssh代理
```
eval "$(ssh-agent -s)"
```
* 将本机的私钥加入到代理中
```
ssh-add ~/.ssh/id_rsa
```

##### 4.3.2 在服务器端
* 登录服务器
* 同样生成公私钥
```
ssh-keygen -t rsa -b 4096 -C "一个邮箱地址"
```
* 同样运行ssh代理
``` 
eval "$(ssh-agent -s)"
```
* 同样将服务器上的私钥加入到代理中
```
ssh-add ~/.ssh/id_rsa
```
* 进入.ssh，生成授权文件
```
vi authorized_keys
```
* 将刚刚本地生成的公钥写入authorized_keys文件
* 给authorized_keys授权
```
chmod 600 authorized_keys
```
* 重启ssh服务
```
sudo service ssh restart
```
* 完成用户免密登录

#### 4.4 修改服务器默认登录端口
* 登录服务器
* 修改配置文件
```
sudo vi /etc/ssh/sshd_config
```
* 在sshd_config文件里，修改端口（默认22，不要使用0~1024， 可使用：1025~65535）
```
Port 39999
```
* 在sshd_config文件里，末尾增加一行（用户名是服务器刚刚添加的用户）
```
AllowUsers 用户名
```
* 保存，退出sshd_config文件
* 重启ssh
```
sudo service ssh restart
```
* 进入sshd_config
```
sudo vi /etc/ssh/sshd_config
```
* 关闭root密码登录
```
PermitRootLogin no
```
* 关闭密码授权
```
PasswordAuthentication no
```
* 重启ssh
```
sudo service ssh restart
```
#### 4.5 完成配置，使用特定端口登录
```
ssh -p 39999 用户名@服务器ip
```

--------------------------------------------

### 5 配置iptables 和 fail2ban 增强安全保护

* 登录服务器
* 更新Ubuntu系统
```
sudo apt-get update && sudo apt-get upgrade
```
#### 5.1 配置iptables
iptables是隔离主机及网络的工具，通过自己设定的规则及处理动作对数据报文进行检测以及处理。 

* 清空iptables规则
```
sudo iptables -F
```
* 打开配置文件
```
sudo vi /etc/iptables.up.rules
```
* 在文件内添加规则
```
*filterss

# allow all connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# allow out traffic
-A OUTPUT -j ACCEPT

# allow http https
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT

# allow ssh port login
-A INPUT -p tcp -m state --state NEW --dport 7153 -j ACCEPT

# ping
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

# log denied calls
-A INPUT -m limit --limit  5/min -j LOG --log-prefix "iptables denied:" --log-level 7

# drop incoming sensitive connections
-A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --set
-A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --update --seconds 60 --hitcount 150 -j DROP

# reject all other inbound
-A INPUT -j REJECT
-A FORWARD -j REJECT
```
* 保存配置
* 更新iptables配置
```
sudo iptables-restore </etc/iptables.up.rules
```
* 激活防火墙
```
sudo ufw enable
```
* 查看防火墙是否被激活
```
sudo ufw status
```
* 配置开机自动启动，打开文件
```
sudo vi /etc/network/if-up.d/iptables
```
* 写入脚本
```
#!/bin/sh
iptables-restore /etc/iptables.up.rules
```
* 保存退出
* 赋予脚本执行权限
```
sudo chmod +x /etc/network/if-up.d/iptables
```
#### 5.2 配置fail2ban
fail2ban可以监视你的系统日志，然后匹配日志的错误信息（正则式匹配）执行相应的屏蔽动作（一般情况下是调用防火墙屏蔽），如:当有人在试探你的HTTP、SSH、SMTP、FTP密码，只要达到你预设的次数，fail2ban就会调用防火墙屏蔽这个IP，而且可以发送e-mail通知系统管理员。
* 安装
```
sudo apt-get install fail2ban
```
* 打开配置文件
```
sudo vi /etc/fail2ban/jail.conf
```
* 修改邮箱地址
```
destemail = 邮箱地址
```
* 修改action
```
# globally (section [DEFAULT]) or per specific section
action = %(action_mw)s
```
* 保存退出
* fail2ban相关命令
```
# 查看运行状态
sudo service fail2ban status
# 终止fail2ban
sudo service fail2ban stop
# 开启fail2ban
sudo service fail2ban start
```
--------------------------------------------

### 6 搭建服务器nodejs环境
* 安装相关依赖模块
```
sudo apt-get install vim openssl build-essential libssl-dev wget curl git
```
* [安装nvm，管理node 版本](https://github.com/creationix/nvm)
* 安装指定版本nodejs
```
nvm install v10.15.1
nvm use v10.15.1
# 设置默认版本
nvm alias default v10.15.1
```
* 设置国内npm淘宝源
```
npm --registry=https://registry.npm.taobao.org install -g npm
```
* 设置系统文件监控数目
```
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```
* 安装其他模块
```
npm i pm2 webpack gulp grunt-cli -g
```
--------------------------------------------
### 7 借助pm2让nodejs常驻
pm2是node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等，而且使用非常简单。
* 相关命令
```
# 启动服务
pm2 start app.js 
# 查看服务列表
pm2 list
# 查看某个服务详细信息
pm2 show app
# 查看实时日志
pm2 logs
```
--------------------------------------------
### 8 nginx端口代理
Nginx (engine x) 是一个高性能的HTTP和反向代理服务，也是一个IMAP/POP3/SMTP服务。
* 安装nginx
```
sudo apt-get install nginx
```
* 查看版本
```
nginx -v
```
* 进入nginx配置目录
```
cd /etc/nginx/conf.d
```
* 新增一个nginx配置文件(域名-后缀-端口.conf)
```
sudo vi www-harryyin-cn-8081.conf
```
* 写入配置
```
upstream example{
  server 127.0.0.1:8081;
}

server {
  listen 80;
  server_name 服务器ip或域名地址（域名解析后）;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Nginx-Proxy true; 

    proxy_pass http://example;
    proxy_redirect off;
  }
  location ~* ^.+\.(html|jpg|jpeg|gif|png|ico|css|js|pdf|txt) {
    root /www/agileui/production/current/dist;
  }
}
```
* 保存退出
* 检测配置文件是否有错
```
sudo nginx -t
```
* 重启nginx
```
sudo nginx -s reload 
```
* 打开nginx主配置
```
cd /etc/nginx
sudo vi nginx.conf
```
* 修改配置在服务器返回内容上隐藏nginx版本
```
# Basic Settings
server_tokens off;
```
* 重载nginx
```
sudo service nginx reload
```
--------------------------------------------
### 9 域名解析
域名解析是把域名指向网站空间IP，让人们通过注册的域名可以方便地访问到网站的一种服务。IP地址是网络上标识站点的数字地址，为了方便记忆，采用域名来代替IP地址标识站点地址。域名解析就是域名到IP地址的转换过程。域名的解析工作由DNS服务器完成。

#### 9.1 更改域名的DNS根服务器
为了对不同厂商的域名进行批量管理，可以将不同厂商的域名转到DNSPod。
* [打开DNSPod](https://www.dnspod.cn/)
* 注册
* [进入帮助中心“万网注册商域名修改DNS地址”](https://support.dnspod.cn/Kb/showarticle/tsid/40/)
* 找到DNSPod的2个DNS短地址
* 打开阿里云所购买域名的管理页面，选择修改DNS
* 将这两个短地址填入，完成修改。（大概4~5个小时可完成修改）

#### 9.2 配置域名解析
* [打开DNSPod](https://www.dnspod.cn/)
* 登录，进入域名解析
* 添加一个域名，点击进入管理页面
* 添加一条记录，主机记录填子域名、记录类型选择A、记录值填服务器ip

--------------------------------------------

### 10 git代码托管（私有仓库）

#### 10.1 与本地项目关联
* 选择一个托管网站：[码云](https://gitee.com/)， [github](https://github.com/)，我选择了码云
* 创建一个仓库
* 将本地项目初始化为git仓库，并和线上私有仓库进行关联，[相关git操作](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

#### 10.2 与服务器关联
* 登录服务器
* 进入.ssh
```
cd ~/.ssh
```
* 打印public key，并复制
```
cat id_rsa.pub
```
* 在码云添加刚刚复制的服务器公钥（标题： Aliyun Ubuntu 14.04 server）

--------------------------------------------

### 11 配置pm2一键部署线上项目结构
[官方文档](http://pm2.keymetrics.io/docs/usage/cluster-mode/)

#### 11.1 在服务器端
* 登录服务器
* 创建部署的目的文件夹
```
sudo mkdir /www
cd /www
sudo mkdir /website
```
* 让用户对新建的文件拥有可读写权限
```
cd /www
sudo chmod 777 website
```

#### 11.2 在本地项目
* 进入本地项目文件夹
* 新建pm2配置文件ecosystem.json
```
{
  "apps": [
    {
      "name": "website", // pm2应用名称
      "script": "app.js", // 服务器启动脚本
      "env": {
        "COMMON_VARIABLE": "true"
      },
      "env_production": {
        "NODE_ENV": "production"
      }
    }
  ],
  "deploy": {
    "production": {  // 环境名
      "user": "manager", // 服务器登录的用户名
      "host": ["127.0.0.1"], // 服务器ip，可以多个
      "port": "8081", // 服务器登录端口
      "ref": "origin/master", // 拉取仓库的分支
      "repo": "", // 关联的私有仓库ssh地址
      "path": "/www/website/production", // 部署的目录
      "ssh_options": "StrictHostKeyChecking=no",
      // 项目部署后在服务器执行的命令
      "post-deploy": "npm install --registry=https://registry.npm.taobao.org && pm2 startOrRestart ecosystem.json --env production",
      "env": {
        "NODE_ENV": "production"
      }
    }
  }
}
```
* 执行部署项目的命令
```
pm2 deploy ecosystem.json production setup
```

#### 11.3 让pm2无交互可运行
* 登录服务器
* 打开.bashrc
```
sudo vi .bashrc 
```
* 注释掉下面的内容
```
# If not running interactively, don't do anything
#case $- in
 #   *i*) ;;
 #     *) return;;
#esac
```
* 保存退出

#### 11.4 添加防火墙规则
* 登录服务器
* 打开iptables.up.rules
```
sudo vi /etc/iptables.up.rules
```
* 添加规则
```
# website
-A INPUT -s 127.0.0.1 -p tcp --destination-port 服务的端口号（例 ：8080） -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -d 127.0.0.1 -p tcp --source-port 服务的端口号（例 ：8080） -m state --state ESTABLISHED -j ACCEPT
```
* 保存退出
* 重载防火墙
```
sudo iptables-restore </etc/iptables.up.rules
```
--------------------------------------------

### 12 选购申请免费的SSL证书
* 厂商选择： 阿里云，腾讯云，七牛，又拍云

--------------------------------------------

## 网站内容更新流程

1. 修改本地代码；
2. git提交代码到私有仓库；
```
git add .
git commit -m '+ update'
git push
```
3. 本地运行命令， 同步代码到线上
```
pm2 deploy ecosystem.json production
```





