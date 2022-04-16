### nginx for windows



##### 下载

​	下载地址：http://nginx.org/en/download.html

##### 反向代理

​	`访问 www.nes.com 直接跳转到 127.0.0.1:8080`

​	1.修改本地 `C:\WINDOWS\System32\drivers\etc\hosts` 文件，`127.0.0.1` 对应 `www.nes.com`

​	2.修改 nginx 配置文件 nginx.conf

```yaml
# 反向代理
server {
   listen       80;
   server_name  127.0.0.1;
   location ~ / {
       proxy_pass http://127.0.0.1:8080;
   }
}
```



`访问 http://127.0.0.1:9001/edu/ 直接跳转到 127.0.0.1:8081 `
`访问 http://127.0.0.1:9001/vod/ 直接跳转到 127.0.0.1:8082`

```yaml
 # 反向代理
 server {
    listen       9001;
    server_name  127.0.0.1;
    location ~ /data/ {
        proxy_pass http://127.0.0.1:8081;
    }
    
    location ~ /edu/ {
        proxy_pass http://127.0.0.1:8082;
    }
 }
```



location路径规则：

```
1、= ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配
成功，就停止继续向下搜索并立即处理该请求。 
2、~：用于表示 uri 包含正则表达式，并且区分大小写。 
3、~*：用于表示 uri 包含正则表达式，并且不区分大小写。 
4、^~：用于不含正则表达式的  uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。 
注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。
```



###### 负载均衡

访问 http://127.0.0.1:9002 路径，在8085、8086端口不断自行切换。

```yaml
  # 负载均横
  upstream myserver{
      server 127.0.0.1:8085;
      server 127.0.0.1:8086;
  }
  
  server {
      listen       9002;
      server_name  127.0.0.1;
     
      location / {
          proxy_pass http://myserver;
          proxy_connect_timeout 10;
      }
  }
```



**负载均衡规则**

1、轮询（默认） 
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。 
2、weight 
weight 代表权,重默认为 1,权重越高被分配的客户端越多。

```yaml
upstream server_pool{    
	server 192.168.5.21 weight=10;     
	server 192.168.5.22 weight=10;     
} 
```

3、ip_hash 
每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。

```
upstream server_pool{    
	ip_hash;     
	server 192.168.5.21:80;     
	server 192.168.5.22:80;     
}
```

4、fair（第三方） 
按后端服务器的响应时间来分配请求，响应时间短的优先分配。 

```
upstream server_pool{    
	server 192.168.5.21:80;     
	server 192.168.5.22:80;     
	fair;     
}
```



###### 动静分离

​	测试动静分离是否成功，之需要删除后端 tomcat 服务器上的某个静态文件，查看是否能访问，如果可以访问说明静态资源 nginx 直接返回了，不走后端 tomcat 服务器。

​	测试一般在linux系统，data路径从根目录开始计算。

```yaml
 # 静态分离
 server {
    listen       80;
    server_name  localhost;
    
    # 动态数据
    location /www/ {
        root /data/;
        index index.html index.htm;
    }
    
    #静态页面
    location /image/ {
        root /data/;
        autoindex on;
    }
 }
```



###### 高可用集群

**Keepalived+Nginx 高可用集群（主从模式）**





###### conf配置文件

```yaml
# <全局块>
#定义Nginx运行的用户和用户组
#user  nobody; 

#nginx进程数，建议设置为等于CPU总核心数。
worker_processes  1;

#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程pid文件
#pid        logs/nginx.pid;

#指定进程可以打开的最大描述符：数目
#工作模式与连接数上限
#这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
#现在在linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。
#这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
# worker_rlimit_nofile 65535;

# <events块>
events {

    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型
    #是Linux 2.6以上版本内核中的高性能网络I/O模型，linux建议epoll，如果跑在FreeBSD上面，就用kqueue模型。
    #补充说明：
    #与apache相类，nginx针对不同的操作系统，有不同的事件模型
    #A）标准事件模型
    #Select、poll属于标准事件模型，如果当前系统不存在更有效的方法，nginx会选择select或poll
    #B）高效事件模型
    #Kqueue：使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X.使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
    #Epoll：使用于Linux内核2.6版本及以后的系统。
    #/dev/poll：使用于Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+。
    #Eventport：使用于Solaris 10。 为了防止出现内核崩溃的问题， 有必要安装安全补丁。
    # use epoll;
    
    #单个进程最大连接数（最大连接数=连接数*进程数）
    #根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。
    worker_connections  1024;
    
    #keepalive超时时间。
    # keepalive_timeout 60;
    
    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。
    #分页大小可以用命令getconf PAGESIZE 取得。
    #[root@web001 ~]# getconf PAGESIZE
    #4096
    #但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。
    # client_header_buffer_size 4k;
    
    #这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
    # open_file_cache max=65535 inactive=60s;
    
    #这个是指多长时间检查一次缓存的有效信息。
    #语法:open_file_cache_valid time 默认值:open_file_cache_valid 60 使用字段:http, server, location 这个指令指定了何时需要检查open_file_cache中缓存项目的有效信息.
    # open_file_cache_valid 80s;
    
    #open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
    #语法:open_file_cache_min_uses number 默认值:open_file_cache_min_uses 1 使用字段:http, server, location  这个指令指定了在open_file_cache指令无效的参数中一定的时间范围内可以使用的最小文件数,如果使用更大的值,文件描述符在cache中总是打开状态.
    # open_file_cache_min_uses 1;
    
    #语法:open_file_cache_errors on | off 默认值:open_file_cache_errors off 使用字段:http, server, location 这个指令指定是否在搜索一个文件是记录cache错误.
    # open_file_cache_errors on;
}

# <http块>
#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {

	# <http全局块>
    #文件扩展名与文件类型映射表
    include       mime.types;
    
    #默认文件类型
    default_type  application/octet-stream;

    #默认编码
    #charset utf-8;
    
    #服务器名字的hash表大小
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，
    #并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，
    #那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
    # server_names_hash_bucket_size 128;
    
    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    # client_header_buffer_size 32k;
    
    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    # large_client_header_buffers 4 64k;
    
    #设定通过nginx上传文件的大小
    # client_max_body_size 8m;
    
    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    #sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    # # sendfile on;
    
    #开启目录列表访问，适合下载服务器，默认关闭。
    # autoindex on;
    
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    #此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    #tcp_nopush     on;
    
    # tcp_nodelay on;

    #长连接超时时间，单位是秒
    #keepalive_timeout  0;
    keepalive_timeout  65;
    
    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    # fastcgi_connect_timeout 300;
    # fastcgi_send_timeout 300;
    # fastcgi_read_timeout 300;
    # fastcgi_buffer_size 64k;
    # fastcgi_buffers 4 64k;
    # fastcgi_busy_buffers_size 128k;
    # fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    # gzip on; #开启gzip压缩输出
    # gzip_min_length 1k;    #最小压缩文件大小
    # gzip_buffers 4 16k;    #压缩缓冲区
    # gzip_http_version 1.0;    #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    # gzip_comp_level 2;    #压缩等级
    # gzip_types text/plain application/x-javascript text/css application/xml;    #压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    # gzip_vary on;
    
    #开启限制IP连接数的时候需要使用
    #limit_zone crawler $binary_remote_addr 10m;

	# <server块>
    #虚拟主机的配置
    server {
    
    	# <server全局块>
        #监听端口
        listen       80;
        #域名可以有多个，用空格隔开
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

		# <location块>
        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
    
    server {
        listen       80;
        
        server_name  127.0.0.1;
        
        location / {
            proxy_pass  http://127.0.0.1:8080;
            index  index.html index.htm index.jsp;
        }
    }
    
    server {
        listen       9090;
        
        server_name  127.0.0.1;
        
        location ~ /edu/ {
            proxy_pass  http://127.0.0.1:8081;
            index  index.html index.htm index.jsp;
        }
        location ~ /vod/ {
            proxy_pass  http://127.0.0.1:8082;
        }
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
       
}
```



##### nginx原理

**master-workers 的机制** 

1. 对于每个 worker 进程，独立的进程，不需要加锁，省掉了锁带来的开销。
2. 采用独立的进程，可以让互相之间不会影响，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快启动新的worker 进程。
3. worker 进程的异常退出，肯定是程序有 bug 了，异常退出，会导致当前 worker 上的所有请求失败，不过不会影响到所有请求，所以降低了风险。



**需要设置多少个 worker** 

1. Nginx  同 redis 类似都采用了 io 多路复用机制，每个 worker 都是一个独立的进程，但每个进
   程里只有一个主线程，通过异步非阻塞的方式来处理请求。每个 worker 的线程可以把一个 cpu 的性能发挥到极致。	
2. worker 数和服务器的 cpu数相等是最为适宜的。



**设置 worker 数量** 
worker_processes 4



**连接数 worker_connection** 

1. 表示每个 worker 进程所能建立连接的最大值，一个 nginx 能建立的最大连接数，应该是 worker_connections  *  worker_processes。
2. 如果是支持 http1.1 的浏览器每次访问要占两个连接，所以普通的静态访问最大并发数是：  worker_connections  *  worker_processes  /2，
3. 而如果是 HTTP 作  为反向代理来说，最大并发数量应该是 worker_connections *   worker_processes/4。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服
   务的连接，会占用两个连接。