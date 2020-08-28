话不多说上配置

```
server {
    listen 443 ssl;                                              #ssl
    server_name  sp-scm.inspur.com;
    ssl_certificate /home/ssl_cer/sp-scm.inspur.com.crt;         #crt证书
    ssl_certificate_key /home/ssl_cer/sp-scm.inspur.com.key;     #私钥
    client_header_buffer_size 16k;
    large_client_header_buffers 4 64k;
    client_max_body_size 15m;
    proxy_connect_timeout 600s;
    proxy_send_timeout 600s;
    proxy_read_timeout 600s;
    proxy_buffer_size 128k;
    proxy_buffers   16 128k;
    proxy_busy_buffers_size 128k;
    proxy_temp_file_write_size 128k;


    #charset koi8-r;
     access_log  /var/log/nginx/spare.access.log  main;

     location / {
             proxy_pass http://192.168.1.2:8080;
             #proxy_redirect off ;
             proxy_redirect http:// $scheme://;
             proxy_set_header Host $host:443;              #假如统有用到重定向，得加上端口
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header REMOTE-HOST $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header    X-Scheme $scheme;
             proxy_set_header X-Forwarded-Proto $scheme;   #这一步为应用重定向时获取client真实														   #协议提供可能
             											   #参考链接1
        }

    error_page 497 https://$host:$server_port$uri$is_args$args;
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

#### 链接1

[Jetty/Tomcat + Nginx反向代理获取客户端真实IP、域名、协议、端口]: http://xxgblog.com/2017/06/23/jetty-tomcat-nginx-proxy-config/?utm_source=tuicool&amp;utm_medium=referral

