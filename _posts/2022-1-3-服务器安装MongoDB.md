---
layout: post
author: shopkeeper
categories: mongodb
---

之前一直使用的是新加坡的免费云数据库，首先限制500M容量，另外有多个项目共享连接时，时而能连上时而连不上，体验很差。本文用于记录我在服务器上安装、连接本地MongoDB服务器时的经历。

## 安装

首先根据官网上的方式安装MongoDB，[点击前往](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/#install-mongodb-community-edition)。

默认的路径为：
- 数据目录：/var/lib/mongo
- 日志目录：/var/log/mongodb

## 使用并创建root账号

启动MongoDB服务，同样可以参照官网，[点击前往](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/#run-mongodb-community-edition):

1. 启用服务`sudo systemctl start mongod`。如果无法启动或配置发生修改时，需要执行`sudo systemctl daemon-reload`。

2. 验证服务启动`sudo systemctl status mongod`。让MondoDB随系统自启动`sudo systemctl enable mongod`。

3. 停止服务`sudo systemctl stop mongod`。

4. 重启服务`sudo systemctl restart mongod`。

连接MongoDB时，可以使用`mongosh`指令来直接连接并进入默认的test数据库。也可以使用`mongo`并可以在后面加上自定义参数。现以`mongo`为例，进入后创建root账号：

```javascript
mongo
db.createUser({user:"root",pwd:"123456",roles:[{role:"root",db:"admin"}]})
```

现在这个数据库有了管理员用户，但是仍然不需要账户密码就能连接。如果一个数据库是没有安全认证的，不使用用户名密码即可登陆，这样是不安全的，所以我们应当授予权限才能操作数据库，这样再企业中才能保证数据安全性。

为此需要增加安全认证。

## 增加安全认证

需要修改MongoDB的配置文件，默认路径为`/etc/mongod.conf`，需要增加以下两行：
```shell
security:
  authorization: enabled
```

注意需要先停止服务，并重新加载守护进程，再启动进程，这时修改的配置才会生效。

这时连接数据库后，需要验证身份，否则不能查看数据库

```javascript
mongo
use admin
db.auth("root","123456")
```
也可以在连接数据库时就验证身份
```javascript
mongo -u "fred" -p "changeme1" --authenticationDatabase "admin"
```

## 远程连接数据库

使用MongoDB Compass来远程连接服务器的数据库，需要干三件事

1. 修改服务器的安全组，添加27017端口的规则
![添加27017端口规则](/assets/images/2022-1-3-服务器安装MongoDB-1.png)

2. 修改MongoDB配置，使其能接收来自所有ip的连接请求
```shell
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0  # Enter 0.0.0.0,:: to bind to all IPv4 and IPv6 addresses or, alternatively, use the net.bindIpAll setting.
```

3. 如果有防火墙的需要开放27017端口
```shell
firewall-cmd --permanent --add-port=27017/tcp
```
使用以下指令查看防火墙端口是否开放
```shell
firewall-cmd --query-port=27017/tcp
```

可以通过url来连接：
```
mongodb://root:*****@<your ip here>/?authSource=admin&readPreference=primary&appname=MongoDB%20Compass&ssl=false
```

也可以使用Compass提供的填入参数登录。

## mongoose连接数据库

最终我还是需要使用mongoose来连接数据库并进行增删改查的，此内容待实际使用时再完成。
