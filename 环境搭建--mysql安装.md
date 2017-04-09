
# Ubuntu16.04中MySQL安装配置

## 安装命令

```shell

sudo apt-get install mysql-server
sudo apt-get install mysql-client
sudo apt-get install libmysqlclient-dev

```

安装过程中，回提示输入设置root密码

## 检查是否安装成功

```shell

sudo netstat -tap | grep mysql 

```
会看到类似输出

`tcp6        0       0       [::]:mysql    [::]:*    LISTEN    7510/mysqld`

## 开启远程访问mysql

- 编辑mysql配置文件，注释掉"bind-address = 127.0.0.1"

配置文件路径: `/etc/mysql/mysql.conf.d/mysqld.cnf`

- 登录root账户

mysql -u root -p123456

- 修改权限

在mysql环境中输入

`grant all on *.* to zzb@’%’ identified by ‘password’;`
或者
`grant all on *.* to zzb@’%’ identified by ‘password’ with grand option;`

- 刷新flush privileges
flush privileges;

- 重启mysql

sudo /etc/init.d/mysql restart

## 常见问题

1045 access denied for user ‘root’@’localhost(ip)’ using password yes

1、mysql -u root -p;
2、GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
3、FLUSH PRIVILEGES;

