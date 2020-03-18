### 1.正向代理与反向代理有什么区别？各自作用是什么？

**<u>反向代理</u>**：即服务端代理，接收客户端请求，然后转发到内部网络服务器。

​	作用：① 安全，可以作为内容服务器的替身。

​		    ② 可实现负载均衡。

**<u>正向代理</u>**：即客户端代理，类似一个跳板机代理访问外部资源。

​	作用：① 访问原本无法访问的资源。如内网服务器通过代理访问外网。

​		    ② 可以做缓存，加速访问资源。

​		    ③ 对客户端统一认证及授权。

### 2.Nginx正向代理http请求

① 代理服务器配置Nginx

```nginx
命令行：vim /usr/local/nginx/conf/nginx.conf

server{
    resolver IP1 IP2；//DNS服务器
    listen 8090;
    location /{
        proxy_pass http://$http_host$request_uri;
    }
}

命令行：service nginx restart
```

② 内网服务器测试http请求

```shell
curl "http://p.qlogo.cn/bizmail/DakhHFWWGsnVNQnKVXnUGibNxCg8JjCV0mYTDXvIsPnAP2k37T3XJAA/0" -v -x 172.31.236.10:8090
```

③ 假如内网服务器tomcat的所有对外http请求都需要走代理服务器，可以如下设置

```shell
命令行：vim catalina.sh

JAVA_OPTS="-Dhttp.proxyHost=172.31.236.10 -Dhttp.proxyPort=8090"
```

### 3.Nginx正向代理https请求

① 代理服务器需要添加支持https的模块ngx_http_proxy_connect_module

```shell
1. 下载ngx_http_proxy_connect_module-master.zip
   地址：https://github.com/chobits/ngx_http_proxy_connect_module
   
2. 上传zip至服务器某个路径下,如/opt/hmap/nginx
   解压tar -zxvf ngx_http_proxy_connect_module-master.zip

3. cd 到Nginx路径下，如/opt/hmap/nginx/nginx-1.10.3

4. patch命令打补丁
命令行：patch -p1</opt/hmap/nginx/ngx_http_proxy_connect_module-master/patch/proxy_connect.patch
【注：不同版本nginx需要将上述命令最后的patch文件路径替换（本文使用的补丁适用于nginx1.4-1.12版本，高版本的补丁也都在patch目录下，具体使用哪个版本可在patch同级目录README.md里的INSTALL/Select patch查找】

5. 重新编译nginx
【使用/usr/local/nginx/sbin/nginx -V 查看你原来安装nginx的参数，在此基础上再加上--add-module】

命令行：./configure --with-http_ssl_module --with-openssl=/opt/hmap/nginx/openssl-1.0.1j --add-module=/opt/hmap/nginx/ngx_http_proxy_connect_module-master

命令行：make

命令行：cp objs/nginx /usr/local/nginx/sbin/nginx
```

② 代理服务器配置Nginx

```nginx
命令行：vim /usr/local/nginx/conf/nginx.conf

server{
    resolver IP1 IP2；#DNS服务器
    listen 8090;
    proxy_connect;
    proxy_connect_allow            443 563;
    proxy_connect_connect_timeout  10s;
    proxy_connect_read_timeout     10s;
    proxy_connect_send_timeout     10s;
    location /{
        proxy_pass https://$http_host$request_uri;
    }
}

命令行：service nginx restart
```

③ 内网服务器测试https请求

```shell
curl "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=xxxxxxx&corpsecret=xxxxxx" -v -x 172.31.236.10:8090
```

④ 假如内网服务器tomcat的所有对外https请求都需要走代理服务器，可以如下设置

```shell
vim catalina.sh
【注意是https】
JAVA_OPTS="-Dhttps.proxyHost=172.31.236.10 -Dhttps.proxyPort=8090"
```



