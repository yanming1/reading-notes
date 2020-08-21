### 一、nginx安装配置(主备机器过程相同)

#### 1.配置nginx yum安装源

新建/etc/yum.repos.d/nginx.repo,添加如下内容

```properties
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

#### 2.安装

```shell
yum install nginx
```

#### 3.加入开机启动

```shell
systemctl enable nginx
```

#### 4.配置

vi /etc/nginx/conf.d/sys.conf，添加代理配置，负载均衡策略采用ip_hash

```nginx
#upstream要与proxy_pass对应你
upstream sys {
        ip_hash;
        server 192.168.1.100:8280;
        server 192.168.1.101:8280;
}

#为了查看请求到底代理到哪台服务器上了，特添加日志特殊格式
log_format  upstream  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                        '$upstream_addr $upstream_response_time $request_time ';
access_log  /var/log/nginx/odm.access.log  upstream;

server {
    listen       80;
    server_name  localhost;
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
	
    #proxy_pass 配置要与upstream配置对应
    location /sys {
             proxy_pass http://sys;
             proxy_redirect off ;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header REMOTE-HOST $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```



#### 5.启动

```
systemctl start nginx
```

#### 6.查看状态

```
systemctl status nginx
```

#### 7.关闭防火墙

```
systemctl stop firewalld.service
```

#### 8.访问应用进行验证

### 二、keepalived安装配置（主备机器安装过程相同，配置差别只体现在第4步）

#### 1.安装

```shell
yum install keepalived
```

#### 2.加入开机启动

```
systemctl enable keepalived
```

#### 3.编写nginx监控脚本

##### (1) vi /etc/keepalived/nginx_check.sh

```shell
#!/bin/bash
counter=$(ps -C nginx --no-heading|wc -l)
if [ "${counter}" = "0" ]; then
    systemctl start nginx
    sleep 2
    counter=$(ps -C nginx --no-heading|wc -l)
    if [ "${counter}" = "0" ]; then
       systemctl stop keepalived.service 
    fi
fi
```

##### (2)添加脚本执行权限

```shell
chmod +x nginx_check.sh
```

#### 4.keepalived抢占式配置配置

##### (1)master配置：vi /etc/keepalived/keepalived.conf

```nginx
! Configuration File for keepalived

global_defs {
#   notification_email {
#     acassen@firewall.loc
#     failover@firewall.loc
#     sysadmin@firewall.loc
#   }
#   notification_email_from Alexandre.Cassen@firewall.loc
#   smtp_server 192.168.200.1
#   smtp_connect_timeout 30
   router_id LVS_DEVEL #标识信息,一个名字而已
#   vrrp_skip_check_adv_addr
#   vrrp_strict
#   vrrp_garp_interval 0
#   vrrp_gna_interval 0
}

# 定义nginx监控脚本
vrrp_script chk_nginx {
        script "/etc/keepalived/nginx_check.sh"
        interval 2 #执行间隔
        weight -20 #根据脚本执行结果调整weight权重
}

vrrp_instance VI_1 {
    state MASTER #角色MASTER
    interface eth0 #虚拟IP绑定网口 通过ip a可查看
    virtual_router_id 51 #让master 和backup在同一个虚拟路由里，id 号必须相同
    priority 100 #优先级 master要高于backup
    advert_int 1 #心跳间隔
    authentication {
        auth_type PASS #认证方式，可以是PASS或AH两种认证方式
        auth_pass 1111 #认证密码
    }
    track_script {
        chk_nginx ## nginx监控脚本
    }
    virtual_ipaddress {
        172.31.4.14 ## 虚拟ip
    }
}
```

##### (2)backup配置：vi /etc/keepalived/keepalived.conf

```nginx
! Configuration File for keepalived

global_defs {
#   notification_email {
#     acassen@firewall.loc
#     failover@firewall.loc
#     sysadmin@firewall.loc
#   }
#   notification_email_from Alexandre.Cassen@firewall.loc
#   smtp_server 192.168.200.1
#   smtp_connect_timeout 30
   router_id dreamerl #标识信息，不要与master相同
#   vrrp_skip_check_adv_addr
#   vrrp_strict
#   vrrp_garp_interval 0
#   vrrp_gna_interval 0
}

# 定义nginx监控脚本
vrrp_script chk_nginx {
        script "/etc/keepalived/nginx_check.sh"
        interval 2 #执行间隔
        weight -20 #根据脚本执行结果调整weight权重
}

vrrp_instance VI_1 {
    state BACKUP #角色BACKUP
    interface eth0 #虚拟IP绑定网口 通过ip a可查看
    virtual_router_id 51 #让master 和backup在同一个虚拟路由里，id 号必须相同
    priority 90 #优先级 backup要低于master
    advert_int 1 #心跳间隔
    authentication {
        auth_type PASS #认证方式，可以是PASS或AH两种认证方式
        auth_pass 1111 #认证密码
    }
    track_script {
        chk_nginx ## nginx监控脚本
    }
    virtual_ipaddress {
        172.31.4.14 ## 虚拟ip
    }
}
```

#### 5.启动

```shell
systemctl start keepalived
```

#### 6.验证

在master上执行

```
ip a
```

正常情况下会发现虚拟ip已挂在网口上

![ip a](E:\learn\reading-notes\软件安装\ip a.png)

在master上执行停掉keepalived服务

```shell
systemctl stop keepalived
```

在backup上执行

```shell
ip a
```

查看虚拟ip已切换到backup上