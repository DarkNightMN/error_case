1.添加用户

```shell
useradd user01
passwd user01
```

2.root为用户添加sudo权限

```shell
vim /etc/sudoers
## 添加用户
user01 ALL=(ALL) ALL
```

3.安装JDK

```shell
tar -zxvf jdk-8u181-linux-x64.tar.gz -C ../
sudo vim /etc/profile

export JAVA_HOME=/home/user01/jdk1.8.0_181
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

source /etc/profile
java -version
```

4.安装tomcat

```shell
tar -zxvf apache-tomcat-8.5.34.tar.gz -C ../

## 配置为Service
sudo cp apache-tomcat-8.5.34/bin/catalina.sh /etc/rc.d/init.d/tomcat

chmod 755 tomcat
vim tomcat

# chkconfig:2345 10 90
# description:Tomcat Service
JAVA_HOME=/home/user01/jdk1.8.0_181
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
CATALINA_HOME=/home/user01/apache-tomcat-8.5.34

chkconfig --add tomcat
sudo service tomcat start
```

5.CentOS防火墙

```shell
启动： systemctl start firewalld
关闭： systemctl stop firewalld
查看状态： systemctl status firewalld 

开机禁用： systemctl disable firewalld
开机启用： systemctl enable firewalld
```

6.安装redis

```shell
tar -zxvf redis-4.0.10.tar.gz -C ../
cd redis-4.0.10
make
cd src
make install

## 配置为Service
sudo cp redis-4.0.10/redis.conf /etc/redis/6379.conf
sudo cp redis-4.0.10/utils/redis_init_script /etc/rc.d/init.d/redis

chmod 755 redis
vim redis

# chkconfig:2345 10 90
# 防止启动时占屏，需要添加&
	echo "Starting Redis server..."
    $EXEC $CONF &


chkconfig --add redis
sudo service redis start
```

7.离线安装GCC环境

```shell
https://blog.csdn.net/qq1031893936/article/details/80396499
下载rpm包【D:\ArtTools\Setup\GCC】，上传至linux，执行rpm -ivh *.rpm --nodeps --force
```

8.离线配置本地yum源

```shell
https://blog.csdn.net/qq_38840475/article/details/83353660
挂载iso镜像，修改配置即可。
```

9.安装nginx

```shell
https://blog.csdn.net/qq_23966661/article/details/88309635
配置为service
https://www.cnblogs.com/shihaiming/p/6290219.html
```

10.安装mysql5.7

```
https://blog.csdn.net/u010199866/article/details/80997485
文件【D:\ArtTools\Setup\Linux软件\linuxmysql】
```



