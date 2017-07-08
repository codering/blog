在公司内网服务器上搭建环境，由于不能连外网，只能自己将软件拷贝上去安装（可能需要编译安装）。
记录一些常用命令，以前总是用的时候重新google。

`以下命令执行都是 root用户，遇到权限问题请加sudo`

## ssh 相关命令

```bash

# 进入ssh目录
cd ~/.ssh

# 生成密钥，`-C` 选项可选
ssh-keygen -t rsa -C 'yourname@anymail.com'

# 一路回车采用默认配置，会生成两个文件 id_rsa, id_rsa.pub, 文件名可以在中途询问时自定义

# 给远程服务器设置别名，避免每次登陆时都要输用户名和IP，编辑.ssh目录中的config文件

vi config

# 假设登陆用户名为root,服务器IP为10.106.1.20, 添加配置如下

Host devServer
    HostName 10.106.1.20
    User root
    IdentityFile ~/.ssh/id_rsa # 配置私钥，就可以免密码登陆

# 保存退出，现在可以将原来 root@10.106.1.20 换成 devServer

# copy本地公钥到远处服务器

ssh-copy-id -i id_rsa.pub devServer

# 测试登陆

ssh devServer

# 在本地通过ssh执行远程服务上的命令，比如我在服务器上写了个上传的脚本，它可以将文件上传到另一台服务器上

ssh devServer "./scripts/upload.sh /tmp/test.tar.gz"

# 文件上传到服务器

scp ./test.tar.gz devServer:/tmp/test.tar.gz

# 端口映射, 用法很多,怕说不清楚,可以看看阮一峰的博客

ssh -L 8888:10.127.8.201:8180 devServer

# 效果是 访问本地的127.0.0.1:8888端口就可以访问到10.127.8.201:8180

```

## 安装java

```bash

# 下载jdk, jdk-8u131-linux-x64.tar.gz
http://www.oracle.com/technetwork/java/javase/downloads/index.html

# 解压到 /usr/java

tar -zxvf jdk-8u131-linux-x64.tar.gz /usr/java

# 添加环境变量

# 编辑/etc/profile文件
vi /etc/profile

# 在末尾添加下面两行
JAVA_HOME=/usr/java/jdk1.8.0_131
JRE_HOME=/usr/java/jdk1.8.0_131/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME PATH

# 当前配置手动触发生效
source /etc/profile

# 看下版本
java -version

```

## 安装nginx

```bash
# 参考 https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/

# 安装gcc

yum install gcc gcc-c++

# 安装 pcre，当前最好不要安装pcre2，可能编译错误
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz
tar -zxf pcre-8.40.tar.gz
cd pcre-8.40
./configure
make
make install

# 安装 zlib
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install

# 安装openssl
wget http://www.openssl.org/source/openssl-1.0.2f.tar.gz
tar -zxf openssl-1.0.2f.tar.gz
cd openssl-1.0.2f
./configure --prefix=/usr
make
make install

# 看下openssl版本
openssl version

# 以上都ok，就可以安装nginx了

wget http://nginx.org/download/nginx-1.12.0.tar.gz
tar zxf nginx-1.12.0.tar.gz
cd nginx-1.12.0

./configure \
--sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-pcre=../pcre-8.40 \
--with-zlib=../zlib-1.2.11 \
--with-openssl=../openssl-1.0.2f \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_sub_module \
--with-stream 

make
make install

# 启动nginx

nginx 

# 修改nginx配置

cd /usr/local/nginx/
vi nginx.conf

# 在最下面添加配置，方便对不同项目采用单独的文件配置

include conf/*.conf;

# 开启日志格式， 取消http下的log_format前面的 '#' 

log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

# 实例，假设为有个域名为 dopey.me, 我想对这个域名启用https, 当通过http访问时自动跳转到https.
# 后端rest请求代理到后端服务器上8080端口上

# 先申请证书，放到nginx安装目录中ssl目录

# 开始配置

vi conf/adopey.me.conf

# 内容如下

server {
    listen       80;
    server_name  dopey.me;
    rewrite ^(.*)$ https://$host$1 permanent;
}

server {
   listen 443 ssl;

   charset utf-8;

   server_name dopey.me;

   ssl_certificate     ssl/dopey.me.cer;
   ssl_certificate_key ssl/dopey.me.key;

   ssl_session_cache    shared:SSL:1m;
   ssl_session_timeout  5m;

   ssl_ciphers  HIGH:!aNULL:!MD5;
   ssl_prefer_server_ciphers  on;

   access_log logs/dopey.me.log main;
   error_log logs/dopey.me.error.log;

   location / {
      root  html;
      index  index.html;
   }

   location /api {
      proxy_pass http://127.0.0.1:8080/api;
      proxy_redirect off;
      proxy_set_header   Host $host:$server_port;
      proxy_hide_header Server;
      proxy_set_header X-Real-IP      $remote_addr;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_connect_timeout      15;
      proxy_send_timeout         25;
      proxy_read_timeout         25;
      proxy_intercept_errors     on;
      client_max_body_size       10m;
      proxy_buffering            off;
   }

   #error_page  404              /404.html;

   # redirect server error pages to the static page /50x.html
   #
   error_page   500 502 503 504  /50x.html;
   location = /50x.html {
      root   html;
   }
}

# 保存退出，检查nginx配置是否正确

nginx -t

# 重新加载nginx配置
nginx -s reload 


```

## 安装jetty

```bash

# 下载jetty, jetty-distribution-9.4.6.v20170531.tar.gz
http://www.eclipse.org/jetty/download.html

# 解压到 /usr/local/jetty
tar -zxvf jetty-distribution-9.4.6.v20170531.tar.gz -C /usr/local
# 重命名
mv jetty-distribution-9.4.6.v20170531 jetty

#创建 jetty 用户
useradd -m jetty

# 改变jetty文件夹的所属用户
chown -R jetty:jetty /usr/local/jetty

#将脚本jetty.sh拷贝init.d目录下
cp bin/jetty.sh /etc/init.d/jetty

# 配置/etc/default/jetty
vi /etc/default/jetty

# 添加内容
PATH=$PATH:/usr/java/jdk1.8.0_131/bin
JETTY_HOME=/usr/local/jetty
NO_START=0
JETTY_USER=jetty
JETTY_PORT=8080
JETTY_HOST=0.0.0.0

# 启动与关闭,重启
service jetty start
service jetty stop
service jetty restart

# 多实例配置

waiting ...

```
