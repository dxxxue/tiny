
# nginx优化

## 一、几个基本概念

- （1）pv 值(page views):页面的浏览量
概念：一个网站的所有页面，在一天内，被浏览的总次数。至少上千万的级别。
- （2）uv值(unique visitor)独立访客
概念：一个网站，在一天内，有多少个用户访问过我们的网站。10万以上
- （3）独立ip
概念：一个网站，在一天内，有多少个独立的ip地址来访问我们的网站。
 
如果要考虑公司的局域网，uv值略大于独立ip的。
 

## 二、解决高并发思路

如果一个网站的uv,pv,独立ip变大，则会导致高的并发，这时要对网站分层布局架构，采用负载均衡。
什么是网站并发连接？
网站服务器在单位时间内能够处理的最大连接数。
负载均衡 ？
硬件：立竿见影，效果非常好，价格非常昂贵，比如F5-BIGIP
软件：lvs(linux virtual server)  ，nginx(web服务器，负载均衡)
负载均衡实现策略 
- （1）轮询， 
负载均衡器把请求轮流转发给后面的web服务器。 
- （2）ip哈希， 
同一个地址的客户端，始终请求同一台主机。 
- （3）最少连接 
负载均衡器把请求给负载最小的哪台服务器。 

## 三、nginx的介绍

### 1、常用web服务器：
apache:功能完善，历史悠久，模块支持非常丰富，属于重量级产品，比较耗费内存。
缺点：处理每一个php比较费资源，导致如果高并发时会耗费服务器资源无法处理更多请求。
lighttpd：内存开销低，cpu占用率低，效能好，模块丰富等特点，轻量级web服务器。

nginx:省资源，省cpu，所以在高并发时能够处理更多的请求，高端能达到3万到5万的并发量。

### 2、nginx介绍
Nginx (engine x) 是一个高性能的HTTP服务器，也是一个IMAP/POP3/SMTP邮件服务器。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。
其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。2011年6月1日，nginx 1.0.4发布。
BSD (Berkeley Software Distribution，伯克利软件套件)
Nginx特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等

### 3、nginx优点
（1）它可以高并发连接，官方测试能够支撑5万并发连接，在实际生产环境中可以支撑2到4万并发连接。 
（2）内存消耗少 
Nginx+php(FastCGI)服务器再3万并发连接下，开启的10个Nginx进程消耗150MB内存（15MB*10=150MB）开启的64个php-cgi进程消耗1280MB内存（20MB*64=1280MB） 
（3）成本低廉 
购买F5 BIG-IP ,NetScaler等硬件负载均衡交换机需要10多万甚至几十万人民币。而Nginx为开源软件，可以免费试用，并且可用于商业用途。 
（4）配置文件非常简单：通俗易懂，即使非专业管理员也能看懂。 
（5）支持 rewrite重写规则：能根据域名、URL的不同，将HTTP请求分到不同的后端服务器群组。 
（6）内置的健康检查功能：如果nginx proxy后端的某台服务器宕机了，不会影响前端访问。 
（7）节省带宽，支持gzip压缩。 
（8）稳定性高：用于反向代理，宕机的概率微乎其微。 
（9）支持热部署。在不间断服务的情况下，对软件版本升级。
 
nginx在反向代理，rewrite规则，稳定性，静态化文件处理，内存消耗等方面，表现出了很强的优势，选用nginx取代传统的apache 服务器，将会获得多方面的性能提升。
（10）支持的操作系统
FreeBSD 3.x,4.x,5.x,6.x i386; FreeBSD 5.x,6.x amd64;
Linux 2.2,2.4,2.6 i386; Linux 2.6 amd64;
Solaris 8 i386; Solaris 9 i386 and sun4u; Solaris 10 i386;
MacOS X （10.4） PPC;
Windows XP，Windows Server 2003和Windows 7等。

## 四、安装nginx软件

### 1、下载软件
http://nginx.org/en/download.html
http://nginx.org/download/nginx-1.12.2.tar.gz

### 2、安装依赖
挂载光盘，安装如下3个依赖软件：

``` shell
[root@localhost ~] yum -y install pcre pcre-devel
[root@localhost ~] yum -y install openssl openssl-devel
[root@localhost ~] yum -y install zlib zlib-devel
```

pcre 和 pcre-devel:两个软件，必须有关系，pcre是线上产品，pcre-devel(develop)是开发版本，里边有一些功能是线上产品不不具备
pcre：		包括 perl 兼容的正则表达式库
openssl：	支持安全传输协议https(和财务有关系的请求会走的协议)
zlib：		优化、压缩支持

### 3、安装nginx软件

- （1）上传软件解压

``` shell
[root@localhost ~] cd /home/jinnan/tar
解压缩
[root@localhost ~] tar zxf nginx-1.12.2.tar.gz
```

- （2）进入nginx解压目录

``` shell
[root@localhost ~] cd nginx-1.12.2
```

- （3）configure配置nginx

``` shell
[root@localhost ~] 
./configure --prefix=/usr/local/nginx --with-http_ssl_module
 	
--with-http_ssl_module			//配置https安全型协议
```

- （4）编译&安装

``` shell
[root@localhost ~] make && make install
```

- （5）启动Nginx

``` shell
[root@localhost ~] /usr/local/nginx/sbin/nginx
```

- （6）查看服务

``` shell
[root@localhost ~] netstat -lntp
如果nginx配置文件有修改，可以通过如下方式测试配置文件是否正确：
[root@localhost ~] /usr/local/nginx/sbin/nginx -t
查看编译的模块
[root@localhost ~] /usr/local/nginx/sbin/nginx -V
注意：安装完成后，会有4个目录，

conf  配置文件  
html  网页文件，网站的根目录，就类似与apache里面的htdocs目录。
logs  日志文件 
sbin  主要二进制程序，启动程序命令
```

### 4、nginx的启动管理

- （1）启动，执行/usr/local/nginx/sbin/nginx文件，完成启动。
参数 "-c" 指定了配置文件的路径，如果不加 "-c" 参数，Nginx 会默认加载其安装目录的 conf 子目录中的 nginx.conf 文件。

- （2）测试是否启动成功
lsof –i :80或netstat –tunpl |grep 80;
 
浏览器中通过ip地址访问；
 
- （3）nginx的管理
通过查看./nginx –h帮助命令
 
``` shell 
nginx 启动
nginx  –s stop  停止nginx的服务
nginx  –s reload  不停止nginx的服务，重新加载配置文件。
nginx  –t  检测配置文件是否有错误。
nginx  -V查看Nginx编译时的参数？
[root@hadoop sbin]# ./nginx -V
nginx version: nginx/1.8.1
built by gcc 4.4.4 20100726 (Red Hat 4.4.4-13) (GCC)
built with OpenSSL 1.0.0-fips 29 Mar 2010
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

### 5、nginx的信号控制

信号控制，是管理nginx进程的一种方式。
具体的用法：kill –信号选项 nginx的主进程号
https://www.nginx.com/resources/wiki/start/topics/tutorials/commandline/
 
- （1）比如立即关闭进程：
kill –INT  nginx的主进程号
查看某个任务的进程号，可以用ps aux|grep nginx
 
- （2）优雅的关闭进程（即等请求结束后，再关闭）
kill –QUIT  nginx的主进程号

- （3）重读配置文件
kill –HUP  nginx的主进程号与nginx –s reload一样
把配置文件中的新更新的东西加载到正在运行的nginx的进程中，接着对用户提供服务，但是nginx的进程并没有关闭。即重读配置文件。
可以通过修改配置文件进行测试

- （4）重读日志文件
Kill  -USR1 `cat /xxx/path/log/nginx.pid`    
重新读日志文件，日志文件改名备份后，使用，否则仍然写入原来的日志文件。

- （5）可以使用nginx.pid文件代替nginx的主进程号
查看nginx.pid文件的位置，打开配置文件进行查看：
即：kill  –信号控制 `cat /xxx/path/log/nginx.pid`

## 五、配置文件

### 1、配置文件介绍，

下面是去掉注释后，配置文件里面的内容；

``` shell
egrep –v “#|^$” nginx.conf
 
通过观察，该配置文件有两段
events {
}
http {
	server {
}
}
注意：每一行用分号结束，内容与{之间要有空格。
```

### 2、全局配置

``` shell
worker_processes  1;配置工作进程的个数，推荐设置为cpu的个数*核心数。
//不同错误信息存储的位置
//全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

//存储nginx进程号的文件
#pid        logs/nginx.pid;
```

### 3、事件配置

``` shell
events {
    //单个cpu进程的最大并发连接数
    //根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行
    //同时要考虑，服务器并不是100%为nginx服务，还有其他工作要做，因此不能达到理论峰值
    worker_connections  1024;
    //并发总数是 worker_processes 和 worker_connections 的乘积
    //即 max_clients = worker_processes * worker_connections
}

worker_connections  1024;配置每个工作进程支持的最大连接数（一个进程的并发量）
```

### 4、虚拟主机的配置
在http段里面的server段就是配置虚拟主机的，http中每一个server段就是一个虚拟主机
- （1）基于域名的虚拟主机 
配置一个www.abc.com的虚拟主机，
第一步：打开nginx的配置文件，进行如下配置。
 
 ``` shell
  server {
        #侦听80端口
        listen       80;
        #定义使用www.abc.com访问
        server_name  www.abc.com;
        
        location / {
            root   /abc;      #定义服务器的默认网站根目录位置
            index index.html index.htm;   #定义首页索引文件的名称
        }
 ```
第二步：修改完成配置文件后，要执行重写加载配置文件

``` shell
/usr/local/nginx/sbin/nginx –s reload
``` 

第三步：在/usr/local/nginx/新建一个abc的目录，该目录就是www.abc.com域名的根目录。
并在里面添加一些测试文件。
 
在window系统hosts 文件里面做好解析；
 
 
也可以通过在linux主机里面本地测试

``` shell
[root@hadoop nginx]# echo "127.0.0.1 www.abc.com" >> /etc/hosts
[root@hadoop abc]#echo 'www.abc.com' >>./index.html
[root@hadoop abc]# curl www.abc.com
www.abc.com
```

- （2）基于端口的虚拟主机 
在配置文件里面，如下配置，
 
 ``` shell
  server {
        #侦听8080端口
        listen       8080;
        #定义使用ip访问
        server_name  _;
        
        location / {
            root   /abc;      #定义服务器的默认网站根目录位置
            index index.html index.htm;   #定义首页索引文件的名称
        }
 ```
 

- （3）基于IP的虚拟主机配置实战
主要实现方式是通过单网卡多ip的方式来实现的；通过如下命令临时在eth0网卡上增加1一个不同的IP地址；
注意：基于IP的虚拟主机配置在生产环境中不经常使用，一般配置在负载均衡后面的服务器上面。

### 5、规范优化nginx配置文件 

大家如果了解apache软件，就会知道apache主配置文件包含虚拟主机子文件的方法，这里也借鉴了apache的这种包含方法，
可以把多个虚拟主机配置成一个个单独的配置文件，仅仅和nginx的主配置文件nginx.conf分离开即可。
这里使用语法是 include,
 
### 6、别名配置 
所谓虚拟主机别名，就是为虚拟主机设置除了主域名以外的一个或多个域名名字，这样就能实现用户访问的多个域名对应同一个虚拟主机网站的功能。
语法：只需在server_name所在行后面添加别名即可。
 
## 六、日志管理
与 nginx日志相关的指令主要有两条，一条是log_format,用来设置日志的格式，另外一条是access_log,用来指定日志文件的存放路径、格式和缓存大小。两条指令在nginx配置文件中的位置可以在http{}之间，也可以在虚拟主机之间，即server{}两个大括号之间。 
### 1、用log_format来设置日志的格式

``` shell
log_format   格式名称   格式样式 
比如如下： 
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' 
                  '$status $body_bytes_sent "$http_referer" ' 
                '"$http_user_agent" "$http_x_forwarded_for"'; 

在日志格式样式中: 
$remote_addr和$http_x_forwarded_for用于记录IP地址
$remote_user用于记录远程客户端用户名称；
$time_local用于记录访问时间与时区；
$request用于记录请求URL与HTTP协议；
$status用于记录请求状态，例如成功时状态为200，页面找不到时状态为404；
$body_bytes_sent用于记录发送给客户端的文件主体内容大小；
$http_referer用于记录是从哪个页面链接访问过来的；
$http_user_agent用于记录客户端浏览器的相关信息。 
```

### 2、用access_log指令指定日志文件存放路径。

Nginx允许针对不同的server做不同的Log ,(有的web服务器不支持,如lighttp) 

``` shell
access_log    logs/access_8080.log      mylog;   
声明log      log位置                log格式; 
注意点：
$remote_addr和$http_x_forwarded_for都用于记录IP地址，区别是什么？
通过$remote_addr变量拿到的将是反向代理服务器（负载均衡）的IP地址。但是，反向代理服务器（负载均衡）在转发请求的HTTP头信息中，可以增加 X-Forwarded-For信息，用以记录原有的客户端IP地址和原来客户端请求的服务器地址。这时候，就要用log_format 指令来设置日志格式，让日志记录X-Forwarded-For信息中的IP地址，即客户的真实IP
```



### 3、定时任务，完成日志分割

``` shell
#!/bin/bash
#此脚本用于自动分割Nginx的日志，包括access.log和error.log
#每天00:00执行此脚本 将前一天的access.log重命名为access-xxxx-xx-xx.log格式，并重新打开日志文件
#0 0 * * * /bin/bash /srv/scripts/cut_nginx_log.sh

#Nginx日志文件所在目录
LOG_PATH=/home/wwwlogs/

#获取昨天的日期
YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)

#获取pid文件路径
PID=/usr/local/nginx/logs/nginx.pid

#分割日志
mv ${LOG_PATH}salesman_access.log ${LOG_PATH}salesman_access-${YESTERDAY}.log
mv ${LOG_PATH}api_access.log ${LOG_PATH}api_access-${YESTERDAY}.log
mv ${LOG_PATH}manage_access.log ${LOG_PATH}manage_access-${YESTERDAY}.log

#向Nginx主进程发送USR1信号，重新打开日志文件
kill -USR1 `cat ${PID}`
```
 
## 七、location语法

#### 1、location的作用
location指令的作用是根据用户请求的URL来执行不同的应用。
在虚拟主机的配置中，是必不可少的，location可以把网站的不同部分定位到不同的处理方式上。
比如：碰到.php 如何调用php解释器，这时就需要location.

#### 2、基本语法如下：

``` shell
location [=|~|~*|^~] patt {

}
```

中括号可以不写任何参数，此时成为一般匹配。
也可以写参数。
因此，大类型可以分为3种。

``` shell
精确匹配
location = patt {
}
一般匹配
location patt{
} 
正则匹配
location ~ patt{
} 
```

#### 3、curl工具使用
在Linux中curl是一个利用URL规则在命令行下工作的文件传输工具，可以说是一款很强大的http命令行工具。
注意点：在linux下面配置的虚拟主机测试时，要在/etc/hosts文件里面做好解析；
vim  /etc/hosts
 
基本用法
``` shell
（1）请求页面
curl    http://www.inux.com
 
（2）-o保存访问的网页，是小写的o
curl –o abc.txt http://www.abc.com
 （3）-s不输出进度
curl –s –o abc.txt  http://www.abc.com
 
（4）-I和-i
-i是显示返回头信息和网页内容
-I只显示返回头信息
curl –i http://www.abc.com
 
（5）-w 输出指定格式的内容
输出格式由普通字符串和任意数量的变量组成，输出变量需要按照%{variable_name}的格式
%{http_code}表示状态码。
curl –s –o /dev/null –I –w "%{http_code}\n" http://www.abc.com
```

#### 4、匹配实例
- （1）精准匹配与一般匹配

``` shell
location = /demo {

}
location  /demo {

}
注意：= 开头表示精确匹配
```
 
- （2）一般匹配长度问题

``` shell
location /demo{}
location /demo/abc {}
```
 
访问的路径中，如果有多个location都符合，则匹配到最长字符串
（location）优先；

- （3）一般匹配与正则匹配

``` shell
location   /images/ {
	config D
}
location ~* \.(gif|jpg|jpeg)$ {
	config E
}
注意：如下
~ 开头表示区分大小写的正则匹配
~* 开头表示不区分大小写的正则匹配
 
域名/images/abc.jpg

总结：如果常规字符串，与正则都匹配，则优先匹配正则
注意：也可以通过再字符串规则前设置^~   表示匹配到常规字符串，不做正则匹配检查
location ^~ /image {}
location ~* \.(gif|jpg|png|js|css)${}
```
 
- （4）默认匹配
``` shell
location   / {
	具体配置
}

/为默认匹配，即如果没有匹配上其他的location，则最后匹配‘默认匹配’部分； 
```

总结：location的命中过程是这样的；
（1）先判断精准匹配，如果匹配成功，立即返回结果并结束解析过程
（2）判断一般匹配，如果有多个匹配成功，记录下来最长的匹配规则，
（3）继续判断正则匹配，按匹配里的正则表达式顺序为准，由上到下开始匹配，一旦匹配成功一个，立即返回结果，结束解析过程。
注意：一般匹配中，顺序无所谓，是按匹配的场地来确定的；正则匹配中，顺序有所谓，因为是从前向后匹配的。

``` shell
不同URL及特殊字符组合匹配的顺序说明
顺序	不同URL及特殊字符组合匹配	匹配说明
1	 location =/{	精确匹配
2	 location ^~ {	匹配常规字符串，不做正则匹配检查
3	 location ~* \.(gif|jpg)|	正则匹配
4	 location /demo/	匹配常规字符串，如果有正则，则优先匹配正则
5	 location / {	所有location都不能匹配后的默认匹配

监控站点可用性
curl -o /dev/null -s -w %{http_code} "http://www.kklinux.com"
```


## 八、Nginx rewrite
http://www.abc.com/index.html   http://www.abc.com/index.php

Nginx rewrite主要功能是实现URL地址重写，需要PCER的支持，前面已经安装。
语法： 
``` shell
rewrite  匹配url 目标url  [flag],应用位置 server location if段中。
rewrite 是实现URL重写的关键指令，根据匹配url部分的内容，重定向到目录url上，结尾是flag标记；
比如：
rewrite  ^/(.*) http://www.abc.com/$1 permanent;

Flag标记符号	说明
last	本条规则匹配完成后，继续向下匹配新的locationURL规则
break	本条规则匹配完成即终止，不再匹配后面的任何规则
redirect	返回302临时重定向，浏览器地址栏会显示跳转后的URL地址
permanent	返回301永久重定向，浏览器地址栏会显示跳转后的url地址
在以上的flag标记中，last和break用来实现url重写，浏览器地址栏的URL地址不变，单在服务器端访问的程序及路径发生了变化。redirect和permanent用来实现URL跳转，浏览器地址栏会显示跳转后的URL地址。
```

### 1、基本案例
案例1：简单重写，访问index.php 重写到abc.html

``` shell
rewrite  ^/index\.php  /abc.html  last;
使用 www.nihao.com域名来测试，打开extra目录下面的nihao.conf配置文件。
```

案例2：实现访问 http://www.nihao.com时跳转到http://www.abc.com 

``` shell
rewrite ^/(.*)   http://www.abc.com/$1  break;
点号，在正则里面表示除了换行符以外的任何字符，
星号，表示0到多个。
使用.*来表示任何字符。
$1符号，表示前面(.*)里面的内容。
```
 

### 2、break与last的区别；

``` shell
location ~ ^/break {
	rewrite ^/break /test  break;
}
location ~ ^/last {
	rewrite ^/last  /test  last;
}
location /test{
	root abc;
	index 123.html;
}
```

last 本条规则匹配完成后，继续向下匹配新的locationURL规则

### 3、redirect与permanent区别

301永久重定向，浏览器会记住，
比如a.com网站 301到 b.com网站，
浏览器中输入a.com时，就不请求a.com了，就直接请求b.com网站了；
302临时重定向，浏览器不记住，
比如a.com网站 302到 b.com网站，
浏览器中输入a.com时，还是请求a.com网站，根据a.com网站响应的location内容，再去请求b.com网站；
301重定向；
rewrite ^/(.*)  http://www.nihao.com/$1 permanent;
 
302重定向；
 
302重定向只是暂时的重定向，搜索引擎会抓取新的内容而保留旧的地址，因为服务器返回302，所以，搜索搜索引擎认为新的网址是暂时的。
而301重定向是永久的重定向，搜索引擎在抓取新的内容的同时也将旧的网址替换为了重定向之后的网址。
 



## 九、gzip压缩提升网站速度；

``` shell
gzip on;  
#开启gzip压缩功能
gzip_min_length 1k;#(开始压缩的最小长度，再小不要压缩)
#设置允许压缩的页面最小字节数，页面字节数从header头的content-length中获取。默认值是0,不管页面多大都进行压缩。建议设置成大于1k。如果小于1k可能会越压越大。
gzip_buffers 4 16k;#(压缩在内存中缓冲几块，每块多大？)
#压缩缓冲区大小。表示申请4个单位为16k的内容作为压缩结果流缓存，默认值是申请与原始数据大小相同的内存空间来存储gzip压缩结果。
gzip_disable;#正则匹配，什么样的uri不进行gzip压缩。
gzip_http_version 1.0;
#压缩版本（默认1.1，前端为squid2.5时使用1.0）用于设置识别http协议版本，默认是1.1,目前大部分浏览器已经支持gzip解压，使用默认即可。
gzip_comp_level 6;
#压缩比率。用来指定gzip压缩比，1压缩比量小，处理速度快；9压缩比量大，传输速度快，但处理最慢，也必将消耗cpu资源。
gzip_types text/plain application/x-javascript text/css application/xml;
#用来指定压缩的类型，“text/html”类型总是会被压缩。
gzip_vary on;#是否传输gzip压缩标志。
#vary header支持。该选项可以让前端的缓存服务器缓存经过gzip压缩的页面，例如用squid缓存经过nginx压缩的数据。
要注意：需要和不需要压缩的对象
```
注意：
（1）小于1k的纯文本文件html,js,css,xml,html不要压缩
（2）图片，视频等不要压缩，因为不但不会减小，在压缩时消耗cpu和内存资源。


## 十、LNMP环境安装PHP
apache+PHP：php是apache内部的功能模块

nginx+PHP：php作为独立服务运行，与nginx地位平等

监听 127.0.0.1的9000端口，当ngxin遇到php文件时，会把这个php的请求转给9000端口的php来处理，nginx本身不能处理php.

### 1、安装

``` shell
[root@localhost ~] rz         //上传php-7.0.25.tar.gz到/home/jinnan/tar
 

[root@localhost ~] cd /home/jinnan/tar
[root@localhost ~] tar zxf php-7.0.25.tar.gz
[root@localhost ~] cd php-7.0.25
现在我们给php做configure配置，就是去除apache参数，增加enable-fpm和--with-config-file-path参数，就两个变化
[root@localhost ~] ./configure \
--prefix=/usr/local/php7.0 \
--with-config-file-path=/usr/local/php7.0/etc \
--with-curl --with-freetype-dir --with-gd \
--with-gettext --with-iconv-dir --with-kerberos \
--with-libxml-dir \
--with-mysqli --with-openssl --with-pcre-regex \
--with-pdo-mysql --with-pear --with-png-dir \
--with-xmlrpc --with-xsl --with-zlib \
--enable-bcmath --enable-libxml \
--enable-inline-optimization \
--enable-gd-native-ttf \
--enable-mbregex --enable-mbstring --enable-opcache \
--enable-pcntl --enable-shmop --enable-soap \
--enable-sockets --enable-sysvsem \
--enable-xml --enable-zip \
--enable-fpm


参数说明：
--with-config-file-path    设定php.ini的存储目录
--with-curl					打开curl的支持
--with-freetype-dir			字体库支持
--with-gd 						画图技术支持
--with-gettext 				支持开发多语言系统
--with-iconv-dir 			iconv函数库能够完成各种字符集间的转换
--with-kerberos 				kerberos支持
--with-libxml-dir 			libxml2库的支持
--with-mysqli 				Mysqli数据库的支持
--with-openssl 				openssl的支持，加密传输时用到的
--with-pcre-regex 			正则表达式支持
--with-pdo-mysql 			pdo-mysql支持
--with-pear 				pear是PHP的扩展和应用程序库，包含了很多有用的类
--with-png-dir 				png图片支持
--with-xmlrpc 				xml相关的扩展库支持
--with-xsl 		打开XSLT 文件支持，扩展了libXML2库 ，需要libxslt软件
--with-zlib 					zlib压缩库支持
--enable-bcmath 				图片大小调整技术支持
--enable-libxml 				xml支持
--enable-inline-optimization 优化线程，给php整体做性能优化处理
--enable-gd-native-ttf 	画图字体库支持，支持TrueType字符串函数库
--enable-mbregex 			正则表达式支持
--enable-mbstring 			mb宽字节函数库支持
--enable-opcache 			缓存支持
--enable-pcntl 				pcntl扩展可以支持php的多线程操作
--enable-shmop 				shmop是一个易于使用的功能集，允许PHP读，写，创建和删除UNIX共享内存段
--enable-sysvsem   			作用同上
--enable-soap 				SOAP 的全称为简单对象访问协议 (Simple Object Access Protocol)。它是一种基于 XML 的，可扩展的通信协议。SOAP 提供了一种标准，使得运行在不同平台上并使用不同的编程语言编写的应用程序可以互相进行通信
--enable-sockets 			sockets 支持
--enable-xml 					xml支持
--enable-zip					php支持对zip压缩包处理
--enable-fpm					将php作为独立服务运行

这一步会遇到一些报错，提示各种依赖缺失，按照提示一个一个用yum安装就行，完毕再重复执行configure指令

通过一条指令统一处理下述依赖(yum可以一次性安装多个软件)：
[root@localhost ~] yum -y install libxml2-devel openssl-devel libcurl libcurl-devel libpng-devel freetype-devel libxslt-devel
值重复执行php的configure指令
 
/*****************************依赖区***********************************/
error: xml2-config not found. Please check your libxml2 installation
[root@localhost ~] yum -y install libxml2-devel
(xml库依赖支持)

error: Cannot find OpenSSL\'s <evp.h>
[root@localhost ~] yum -y install openssl-devel
(openssl：	支持安全传输协议https)

checking for cURL in default path... not found
[root@localhost ~] yum -y install libcurl libcurl-devel
(curl是服务器彼此间调用的接口，常用于爬虫技术)

configure: error: png.h not found.
[root@localhost ~] yum -y install libpng-devel
(png图片格式支持)

configure: error: freetype-config not found.
[root@localhost ~] yum -y install freetype-devel
(画图字体库支持)

configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution
[root@localhost ~] yum -y install libxslt-devel
(打开XSLT 文件支持，扩展了libXML2库)
/*****************************依赖区***********************************/

[root@localhost ~] make && make install

php执行完毕configure的效果：
 



[root@localhost ~] make && make install
```

### 2、配置

``` shell
（1）复制php.ini-development配置文件
安装完成后将php.ini-development 复制到/usr/local/php7.0/etc/php.ini
[root@localhost ~] cp php.ini-development /usr/local/php7.0/etc/php.ini
 
（2）给php服务，服务名称是fpm创建服务配置文件
[root@localhost ~] cd /usr/local/php7.0/etc
//给php服务，服务名称是fpm创建服务配置文件
[root@localhost ~] cp php-fpm.conf.default    php-fpm.conf
 
[root@localhost ~] vim /usr/local/php7.0/etc/php-fpm.conf
找到如下内容：
;pid = run/php-fpm.pid
去除;分号修改为：
pid = run/php-fpm.pid
上述文件是用于保存php服务进程号码的
 
:wq保存退出
（3）配置辅助配置文件
[root@localhost ~] cd /usr/local/php7.0/etc/php-fpm.d
[root@localhost ~] cp www.conf.default   www.conf
 
/usr/local/php7.0/etc/php-fpm.conf:			是php服务的主配置文件
/usr/local/php7.0/etc/php-fpm.d/www.conf:	是php服务辅助配置文件，需要被上述的php-fpm.conf引入
```

### 3、控制服务

``` shell
启动php服务
[root@localhost ~] /usr/local/php7.0/sbin/php-fpm
 

查看php服务进程
[root@localhost ~] netstat -lntp
 

设置环境变量，使得在任何地方都可以直接访问php-fpm
[root@localhost ~] vim /etc/profile
再文档最后设置如下内容：
export PATH=/usr/local/php7.0/sbin:$PATH
 
:wq保存退出
使得环境变量立即生效
[root@localhost ~] source /etc/profile
也可以配置nginx的环境变量
export PATH=/usr/local/nginx/sbin:$PATH
关闭php服务：
> killall  php-fpm
杀掉php服务进程：
 

启动php服务进程：
> php-fpm
```

### 4、修改配置

``` shell
修改php时区：
> vi /usr/local/php7.0/etc/php.ini
 
重启php服务：
```

### 5、编译 php与nginx整合 

``` shell
（1）编辑文件：
shell># vim     /usr/local/nginx/conf/fastcgi.conf 
直接使用conf配置目录下面的fastcgi.conf文件也可以，不需要自己在建立fcgi.conf 文件了。
 

并写入如下内容 
fastcgi_param  GATEWAY_INTERFACE  CGI/1.1; 
fastcgi_param  SERVER_SOFTWARE    nginx; 
fastcgi_param  QUERY_STRING       $query_string; 
fastcgi_param  REQUEST_METHOD     $request_method; 
fastcgi_param  CONTENT_TYPE       $content_type; 
fastcgi_param  CONTENT_LENGTH     $content_length; 
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name; 
fastcgi_param  SCRIPT_NAME        $fastcgi_script_name; 
fastcgi_param  REQUEST_URI        $request_uri; 
fastcgi_param  DOCUMENT_URI       $document_uri; 
fastcgi_param  DOCUMENT_ROOT      $document_root; 
fastcgi_param  SERVER_PROTOCOL    $server_protocol; 
fastcgi_param  REMOTE_ADDR        $remote_addr; 
fastcgi_param  REMOTE_PORT        $remote_port; 
fastcgi_param  SERVER_ADDR        $server_addr; 
fastcgi_param  SERVER_PORT        $server_port; 
fastcgi_param  SERVER_NAME        $server_name; 
  
# PHP only, required if PHP was built with --enable-force-cgi-redirect 
fastcgi_param  REDIRECT_STATUS    200; 

（2）编辑nginx配置文件(找到server项目设置如下内容)：
shell># vim /usr/local/nginx/conf/nginx.conf 
location ~ .*\.(php|php5)?$ 
{ 
  fastcgi_pass  127.0.0.1:9000;        
  #fastcgi（管理PHP）， 
  fastcgi_index index.php; 
  include fastcgi.conf; 
} 
```
 
### 6、nginx上面部署TP框架
具体的配置如下，可以把如下内容，添加到虚拟主机里面，即可。

``` shell
location ~ .+\.php($|/) { 
   set $script    $uri; 
   set $path_info  "/"; 
   if ($uri ~ "^(.+\.php)(/.+)") { 
       set $script     $1; 
       set $path_info  $2; 
    } 
   
   fastcgi_pass 127.0.0.1:9000; 
   fastcgi_index  index.php?IF_REWRITE=1; 
   include fastcgi.conf; 
   fastcgi_param PATH_INFO $path_info; 
   fastcgi_param SCRIPT_FILENAME  $document_root/$script; 
   fastcgi_param SCRIPT_NAME $script; 
} 
```
 

### 7、nginx上面部署TP框架实现伪静态

``` shell
只需要在虚拟主机的配置文件里面，添加如下内容即可：
if (!-e $request_filename){
	rewrite ^/(.*)$  /index.php/$1  last;
} 
```

## 十一、nginx的负载均衡的配置

``` shell
# 指定集群服务器
upstream services {
    server 10.20.14.11:8088;
    server 10.20.14.12:8088;
}

server {
        listen       80;
        location / {
            proxy_pass   http://services; # 转发到集群
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_redirect default;
            proxy_buffer_size 512k;
            proxy_buffers 6 512k;
            proxy_busy_buffers_size 512k;
            proxy_temp_file_write_size 512k;
            client_max_body_size 100m;
        }
}

server指令：
语法：server name [参数] 
使用环境：upstream 
该指令用于指定后端服务器的名称和参数。服务器的名称可以是一个域名，-个ip地址，端口号。
在后端服务器名称之后，可以跟以下参数：
weight=number  设置服务器的权重，权重数值越高，被分配到的客户端请求数越多。
如果没有设置权重，则为默认权重为1. 
upstream  连接池名称{
	server  192.168.0.100  weight=5;
	server  192.168.0.200  weight=1;
	server  192.168.0.210  weight=3;
}
max_fails=number 在参数fail_timeout指定的时间内对后端服务器请求失败的次数，如果检测到后端服务器无法连接及发生服务器错误（404错误除外），则标记为失败。如果没有设置，则为默认值1。设为数值0将关闭这项检查。
fail_timeout=time(30s)在经历参数max_fails设置的失败次数后，暂停的世界。
down 标记服务器为永久离线状态，用于ip_hash指令。
backup 仅仅在非backup服务器全部宕机或繁忙的时候，才启用。
```