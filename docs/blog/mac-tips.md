# Mac Tips

### Mac 使用小技巧
1. 在终端打开访达，直接运行 open 命令即可。比如想往一个目录复制文件，在命令行里cd到路径后，通过open命令打开访达，复制文件更方便
```
open .     // 打开当前目录
```
### 关于zsh终端的一个坑
修改完 ~/.bash_profile 添加全局变量后，用zsh终端重新打开，还是找不到命令。解决办法，在 ~/.zshrc 里添加

```
source ~/.bash_profile
```
即每次打开终端，自动执行一遍配置文件，就能找到对应命令了

### Mac安装mysql后配置文件
[Mac安装mysql后配置文件](https://blog.csdn.net/thomas0713/article/details/82928020)
```
mysql -u root -proot   本机mysql账号密码
```

### Mac 自带apache 的使用
```
sudo apachectl start   // 启动
sudo apachectl stop    // 停止
```
配置文件目录
```
/etc/apache2/httpd.conf
```
站点根目录
```
/Library/WebServer/Documents/
```

### Mac 安装 mongodb
```
brew install mongodb

To have launchd start mongodb now and restart at login:
  brew services start mongodb
Or, if you don't want/need a background service you can just run:
  mongod --config /usr/local/etc/mongod.conf
==> Summary
🍺  /usr/local/Cellar/mongodb/4.0.3_1: 18 files, 258.1MB
```
### Mac 安装 nginx
安装
```
brew install nginx
```
安装完提示
```
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
```
根据提示，直接输入 nginx 命令就可以打开服务器了。
站点根目录在 /usr/local/var/www。配置文件在 /usr/local/etc/nginx/nginx.conf。

### 搭建一个MQTT服务器，mac版
MQTT协议之所以适用于物联网，是因为其对带宽要求不高，对数据传输的有效性要求不高
1. brew install mosquitto，安装成功提示
```
You can make changes to the configuration by editing:
    /usr/local/etc/mosquitto/mosquitto.conf

To have launchd start mosquitto now and restart at login:
  brew services start mosquitto
Or, if you don't want/need a background service you can just run:
  mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf
```
启动/重启命令
```
brew services start mosquitto #启动服务
brew services restart mosquitto #重启服务 
brew services stop mosquitto #停止服务
```
Mosquitto配置
```
/usr/local/etc/mosquitto/mosquitto.conf
```
修改配置文件，其默认配置文件是注释掉的，需要去掉#
```
bind_address 127.0.0.1
port 1883
```
测试， 打开三个终端，分别输入下面的命令
```
brew services start mosquitto  #第一个终端，开启mqtt服务器
mosquitto_sub -h localhost -t "test" #第二个终端，订阅主题为test
mosquitto_pub -h localhost -t "test" -m "hello" #发布主题test,发送内容为hello
```
也可以下载客户端mqttfx，可视化操作
```
brew install mqttfx
```

### 安装puppeteer
问题：安装Chromium报错，网上有几种解决方法，经过实践，最简单的一种如下
```
PUPPETEER_DOWNLOAD_HOST=https://npm.taobao.org/mirrors npm i --save puppeteer
```
[参考网址](https://www.v2ex.com/t/485373)  
安装完之后Chromium的路径(我的是mac)
```
/Users/sam/work/movie-fans/node_modules/puppeteer/.local-chromium/mac-624492
```