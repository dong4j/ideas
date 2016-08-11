# Nginx 一个端口多个站点(多域名虚拟主机配置)
需求是同一台服务器上有多个前端站点,
要求一台 Nginx 能负责所有站点的反向代理.

## 一个文件多个域名的写法的具体配置
```
# user  nobody;
# 和 cpu 核心数相同
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    # 服务1 访问aaa.com时匹配
    server {
        # 监听80端口(浏览器访问)
        listen       8002;
        # 主机名
        server_name  aaa.com;
        # 访问日志文件存放路径
        access_log    /Users/xxx/Documents/logs/nginx/aaa.com.access.log    combined;
        # 设置静态页面目录
        root /Users/xxx/Documents/MyFiles/Work/shuguang/code/sofn-front;
        # 默认首页
        index index.html;
        location / {
            # 用户浏览器端的缓存设置
            location ~ .*\.(js|css|jpg|jpeg|gif|png|swf|htm|html|json|xml|svg|woff|ttf|eot|map|ico)$ {
                expires 1h;
                if (-f $request_filename) {
                    break;
                }
            }
            # 动态页面请求的地址
            if ( !-e $request_filename) {
                proxy_pass       http://127.0.0.1:18088;
            }
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    # 服务2 访问bbb.com时匹配
    server {
        # 监听80端口(浏览器访问)
        listen       8002;
        # 主机名
        server_name  bbb.com;
        # 访问日志文件存放路径
        access_log    /Users/xxx/Documents/logs/nginx/bbb.com.access.log    combined;
        # 设置静态页面目录
        root /Users/xxx/Documents/MyFiles/Work/shuguang/code/sofn-front_2;
        # 默认首页
        index index.html;
        location / {
            # 用户浏览器端的缓存设置,如果以这些后缀结尾的 直接返回
            location ~ .*\.(js|css|jpg|jpeg|gif|png|swf|htm|html|json|xml|svg|woff|ttf|eot|map|ico)$ {
                # 缓存1小时
                expires 1h;
                if (-f $request_filename) {
                    break;
                }
            }
            # 动态页面 如果没有配置的,则去服务器取
            if ( !-e $request_filename) {
                proxy_pass       http://127.0.0.1:28088;
            }
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

设置 hosts 文件
127.0.0.1 aaa.com
127.0.0.1 bbb.com

## 每个域名一个文件的写法的具体配置

```
cd  /usr/local/etc/nginx/servers
touch www.aaa.com.conf
touch www.bbb.com.conf
```
把方法一中的2个 server 模块 分别拷贝的新建的2歌配置文件中

修改 nginx.conf 文件

```
# user  nobody;
worker_processes  1;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    # 包含 servers 目录下的所有 server 配置
    include servers/*;
}

```

# Nginx 负债均衡
前端发送请求后, nginx 根据不同的算法,均分请求
## 具体配置

```
# user  nobody;
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    # 负债均衡设置
    upstream webservers {
        ip_hash;
        server 127.0.0.1:18088 weight=2 max_fails=1 fail_timeout=10s;
        server 127.0.0.1:28088 weight=2 max_fails=1 fail_timeout=10s;
        server 127.0.0.1:38088 weight=2 max_fails=1 fail_timeout=10s;
        keepalive 512;
    }
    server {
        # 监听80端口(浏览器访问)
        listen       8002;
        server_name  localhost;
        # 设置静态页面目录
        root /Users/xxx/Documents/MyFiles/Work/shuguang/code/sofn-front;
        # 默认首页
        index index.html;
        location / {
            # 用户浏览器端的缓存设置
            location ~ .*\.(js|css|jpg|jpeg|gif|png|swf|htm|html|json|xml|svg|woff|ttf|eot|map|ico)$ {
                expires 1h;
                if (-f $request_filename) {
                    break;
                }
            }
            # 动态页面
            if ( !-e $request_filename) {
                # 和 upstream 同名 
                proxy_pass http://webservers;
            }
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    include servers/*;
}
```

此设置将通过 ip_hash 算法选择服务器响应请求,不过有 session 不一致问题


