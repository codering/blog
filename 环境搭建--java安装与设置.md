## 卸载openjdk

sudo apt-get purge openjdk*

## 安装 java8

$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer

## 多版本控制

sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-8-oracle/jre/bin/java 1008
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-7-oracle/jre/bin/java 1007
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-6-oracle/jre/bin/java 1006

sudo update-alternatives --config java

## 删除多版本控制

sudo update-alternatives --remove-all java