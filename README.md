# MultiTenancyDomain
多租户系统中，openresty反向代理中的域名处理

## Openresty 服务器  
192.168.0.225

## Web应用服务器
192.168.0.203

## web应用站点
192.168.0.203:8001  www.abc.com
192.168.0.203:8002  www.a.com
192.168.0.203:8003  www.b.com
192.168.0.203:8004  www.c.com

## Hosts文件
192.168.0.225    www.abc.com   
192.168.0.225    a.abc.com   
192.168.0.225    b.abc.com   
192.168.0.225    c.abc.com   
192.168.0.225    www.a.com   
192.168.0.225    www.b.com   
192.168.0.225    www.c.com   

192.168.0.225    abc.com   
192.168.0.225    a.abc.com   
192.168.0.225    b.abc.com   
192.168.0.225    c.abc.com   
192.168.0.225    a.com   
192.168.0.225    b.com   
192.168.0.225    c.com  

## nginx.conf
<!-- lang: lua --> 
    #user  nobody;
    worker_processes  1;

    #error_log  logs/error.log;
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;

    #pid        logs/nginx.pid;


    events {
        worker_connections  1024;
    }


    http {
        include       mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

        #access_log  logs/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        #keepalive_timeout  0;
        keepalive_timeout  65;

        upstream abc{
            server 192.168.0.203:8001;
        }
        upstream a{
            server 192.168.0.203:8002;
        }
        upstream b{
            server 192.168.0.203:8003;
        }
        upstream c{
            server 192.168.0.203:8004;
        }
        #gzip  on;
        server {
            listen       80;
            server_name  ~^(?<subdomain>.+)\.abc\.com$;
            #charset koi8-r;
            access_log  logs/abc.access.log  main;
            location / {
                rewrite_by_lua '
                    if ngx.var.subdomain ~= nil and ngx.var.subdomain ~= "www" and ngx.var.subdomain ~= "" then
                        return ngx.redirect("http://"..ngx.var.subdomain..".com",302)
                    end
                ';
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://abc;
            }
            #error_page  404              /404.html;
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
        server {
            listen       80;
            server_name  *.a.com a.com;
            #charset koi8-r;
            access_log  logs/a.access.log  main;
            location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://a;
            }
            #error_page  404              /404.html;
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
        server {
            listen       80;
            server_name  *.b.com b.com;
            #charset koi8-r;
            access_log  logs/b.access.log  main;
            location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://b;
            }
            #error_page  404              /404.html;
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
        server {
            listen       80;
            server_name  *.c.com c.com;
            #charset koi8-r;
            access_log  logs/c.access.log  main;
            location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://c;
            }
            #error_page  404              /404.html;
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
    }

