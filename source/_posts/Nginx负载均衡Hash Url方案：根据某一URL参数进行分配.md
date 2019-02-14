---
title: Nginx负载均衡Hash Url方案：根据某一URL参数进行分配
tags: [Nginx,负载均衡]
grammar_cjkRuby: true
categories: [架构]
date: 2019-02-14
---
### 原理
- 使用upstream实现负载均衡
- 使用Url Hash实现按Url分配
- 通过自定义变量提取URL参数，使用变量Hash实现按变量值分配

### 配置文件
``` nginxconf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}


http {
    
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
	
	upstream www_load {
	    #根据参数进行Hash分配
		hash $defurlkey;
		server localhost:5000;
		server localhost:5001;
		server localhost:5002;
		}

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

		# 根据user_id和&在提取参数
        if ( $request_uri ~* "^\/.*user_id=(.*)&.*" ){
           set $defurlkey $1;
        }
          
        location / {
        }
        
        location /load{
           root /usr/share/nginx/html;
           proxy_set_header Host $host;
           proxy_pass http://www_load/;
           index index.html index.htm;
        }

        location /test{
            proxy_pass http://127.0.0.1:8084/testxj;
        }
        location /ocr{
            proxy_pass http://webapi.xfyun.cn/v1/service/v1;
        }
        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

	server {
	    listen       5000;
		server_name  _;

		location / {
            root   /usr/share/nginx/html/h0;
			#random_index on;
			index index.html index.htm;
		}
	}
	server {
	    listen       5001;
		server_name  _;
		
		location / {
		    root   /usr/share/nginx/html/h1;
			index index.html index.htm;
		}
	}
	
	server {
        listen       5002;
		server_name  _;
		location / {
            root   /usr/share/nginx/html/h2;
			index index.html index.htm;
		}
	}
}

```


### 参考资料
1、https://www.imooc.com/article/19980
2、https://blog.csdn.net/my_bai/article/details/77977975