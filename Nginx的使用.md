#### Nginx的使用

##### 概述

+ 什么是 Nginx ？

  ~~~ html
  Nginx （engine x） 是一款经量级的 Web 服务器、反向代理服务器 及 电子邮件（IMAP/POP3）代理服务器。
  ~~~

+ 代理服务器
~~~html
 代理服务器，客户机在发送请求时，不会直接发送给目的主机，而是先发送给代理服务器，代理服务器接收客户机请求之后，再向主机发出，并接受目的主机返回的数据，存放在代理服务器的硬盘中，再发送给客户机。
~~~


  ![](http://dunwu.test.upcdn.net/images/web/nginx/nginx.jpg)

- 什么是反向代理？

  ~~~ html
  反向代理（Reverse Proxy）方式是指代理服务器来接收 Internet 上的连接请求，然后将请求转发给 内部网络上的服务器，并将从服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。
  ~~~

  ![](http://dunwu.test.upcdn.net/images/web/proxy-server.jpg)



#### 安装与使用

##### 安装

[nginx 官网下载地址](http://nginx.org/)

发布版本为 Linux 和 windows 版本。

也可以下载源码，编译后运行。

##### 从源代码编译 Nginx 

- 把源码解压缩之后，在终端里运行如下命令：

  ~~~xml
  //sudo 为 superuser do 的简写，使用超级用户来执行命令，一般是指root用户
  
  $ ./configure
  $ make
  $ sudo make install 
  ~~~

  默认情况下，Nginx 会 被 安装 在 `/user/local/nginx`  通过设定 [编译选项](http://tool.oschina.net/uploads/apidocs/nginx-zh/NginxChsInstallOptions.htm)，你可以改变这个设定。

##### Windows 安装

+ 为了安装 Nginx / Win64 ，需先下载它，然后解压，然后运行即可。

  下面 以 自定义 的盘根目录 (D盘 ) 为例说明（D:\nginx\nginx-1.14.2，以看到有一个 nginx.exe运行程序 为准 ）

  ~~~html
  D:
  D:\>cd D:\nginx\nginx-1.14.2 start nginx
  ~~~
  Nginx / Win32 是运行 在一个控制台程序，而非windows 服务方式的。服务器方式目前还是开发尝试中。

##### 使用

常用到的命令如下：

|   常用命令   |  命令解释 |
| :------------ | :----------------------------------------------------: |
| nginx -s  stop | 快速关闭Nginx，可能不保存相关信息，并迅速终止web服务。       |
| nginx -s  quit | 平稳关闭Nginx，保存相关信息，有安排的结束web服务。 |
| nginx -s reload | 因改变了Nginx相关配置，需要重新加载配置而重载。 |
| nginx -s reopen | 重新打开日志文件 |
| nginx -c filename | 为 Nginx 指定一个配置文件，来代替缺省的。 |
| nginx -t | 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。 |
| nginx -v | 显示 nginx 的版本。 |
| nginx -V | 显示 nginx 的版本，编译器版本和配置参数。 |

如果不想每次都敲命令，可以在 nginx 安装目录 下 新添 一个 启动批处理文件 startup.bat，双击即可运行。内容如下：

~~~ xml
@echo off
rem 如果启动前已经 启动 nginx 并记录下 pid文件，会kill 指定进程
nginx.exe -s stop 

rem 测试配置文件语法正确性
nginx.exe -t -c conf/nginx.conf

rem 显示版本信息
nginx.exe -v

rem 按照指定配置去启动nginx
nginx.exe -c conf/nginx.conf
~~~

如果是运行在Linux下，写一个shell脚本，差不都都是一样。

##### nginx 配置实践

+ http 反向代理配置

  ~~~ html
  我们先实现一个小目标：不考虑复杂的配置，仅仅是完成一个 http 反向代理。
  ~~~

  nginx.conf 配置文件如下:


​     ` 注：conf / nginx.conf 是 nginx 的默认配置文件。你也可以使用 nginx -c 指定你的配置文件`

~~~ xml
# 运行用户
#user somebody；

#启动进程，通常设置成和cup的数量相等
worker_processes 1;

#全局错误日志 (可以指定存放的位置)
error_log E:/Tools/nginx/logs/error.log;
error_log E:/Tools/nginx/logs/notice.log notice;
error_log E:/Tools/nginx/logs/info.log info;

#PID 文件，记录当前启动的nginx进程ID
pid E:/Tools/nginx/logs/nginx.pid;

#工作模式及连接数上限
events{
	# 单个后台 worker process 进程的最大并发链接数
	worker_connections 1024;
}

#设定 http 服务器，利用它的反向代理功能提供负载均衡支持
http{
	#设定 mime类型 （邮件支持类型），类型有mime.types文件定义;
	include E:/Tools/nginx/conf/mime.types;
	defaul_type application/octet-stream;
	#设定日志的输出格式
	log_format main  '[$remote_addr] - [$remote_user] [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"';
	access_log D:/Tools/nginx/logs/access.log main;
	rewrite_log 	on;
	#sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用
	#必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
	sendfile 		on;
	#tcp_nopush     on;
	
	#连接超时时间
	keepalive_timeout	120;
	tcp_nodelay		on;
	
	#gzip 压缩开关
	#gzip	on;
	
	#设定实际的服务器列表
	upstream zp_server1{
		server 192.168.8.44:8089;
	}
	
	# HTTP 服务器
	server{
		#监听80端口，80端口是知名端口号，用户HTTP协议
		listen	80;
		#定义使用 www.xx.com
		server_name www.helloword.com;
		#首页
		index index.html
		#指向 webapp 的目录
		D:\01_Workspace\Project\github\zp\SpringNotes\spring-security\spring-shiro\src\main\webapp;
		
		#编码格式
        charset utf-8;

        #代理配置参数
        proxy_connect_timeout 180;
        proxy_send_timeout 180;
        proxy_read_timeout 180;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarder-For $remote_addr;

        #反向代理的路径（和upstream绑定），location 后面设置映射的路径
        location / {
            proxy_pass http://zp_server1;
        }

        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            root D:\01_Workspace\Project\github\zp\SpringNotes\spring-security\spring-shiro\src\main\webapp\views;
            #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
            expires 30d;
        }

        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status           on;
            access_log            on;
            auth_basic            "NginxStatus";
            auth_basic_user_file  conf/htpasswd;
        }

        #禁止访问 .htxxx 文件
        location ~ /\.ht {
            deny all;
        }

        #错误处理页面（可选择性配置）
        #error_page   404              /404.html;
        #error_page   500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}
	}

}

~~~
