## 相关概念

### 简单请求

1. 请求方法是`head`、`get`、`post`三种之一
2. HTTP 没有自定义头信息，Content-Type 的值仅限于：
   - `application/x-www-form-urlencoded`
   - `text/plain`
   - `multipart/form-data`

### 非简单请求

​	不满足简单请求条件的请求都是非简单请求。非简单请求有一种`CORS-preflight`机制(预检机制)，即浏览器在真正通讯前发送一次 HTTP 预检`option`请求询问服务器，资源是否允许跨域以及可以使用的 HTTP 请求方法。服务器只需针对 `preflight`进行跨源许可计算，本身的响应代码则不会运行。可以做到默认禁止跨源请求。但是简单请求不能依赖`CORS-preflight`机制，所以做不到默认禁止跨源请求

## 跨域

​	在浏览器上当前访问的网站向另一个网站发送请求获取数据的过程就是跨域请求

## 正向代理和反向代理

### 正向代理

​	正向代理代理的是客户端。正向代理隐藏了真实的客户端，为客户端收发请求，使真实的客户端对服务器端不可见

### 反向代理

​	反向代理代理的是服务器端。反向代理隐藏了真实的服务器，为服务器收发请求，使真实的服务器端对服务器端不可见

![正向代理和反向代理](C:\Users\renzq\Desktop\ReadingNotes\nginx\imgs\reverse.jpg)

## 负载均衡

​	随着信息数量不断增长，访问量和数据量飞速增长，以及系统业务的复杂度持续增加。单一的服务器性能已无法满足需求。这时候就出现了服务器集群额概念。负载均衡就是，将请求分发到各个服务器上，将负载分配到不同的服务器，核心就是分摊压力

![负载均衡](C:\Users\renzq\Desktop\ReadingNotes\nginx\imgs\balance.png)

## 动静分离

​	将动态资源和静态资源分离，将静态资源部署在`nginx`上。如果请求的是静态资源，直接到静态资源目录获取资源，如果是动态资源，则利用反向代理，把请求转发给对应的后台应用处理，从而实现动静分离。

![动静分离](C:\Users\renzq\Desktop\ReadingNotes\nginx\imgs\client.png)

### 经典配置

```
user  nginx;                        # 运行用户，默认即是nginx，可以不进行设置
worker_processes  1;                # Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;   # Nginx 的错误日志存放目录
pid        /var/run/nginx.pid;      # Nginx 服务启动时的 pid 存放位置

events {
    use epoll;     # 使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
    worker_connections 1024;   # 每个进程允许最大并发数
}

http {   # 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
    # 设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;   # Nginx访问日志存放位置

    sendfile            on;   # 开启高效传输模式
    tcp_nopush          on;   # 减少网络报文段的数量
    tcp_nodelay         on;
    keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
    default_type        application/octet-stream;   # 默认文件类型

    include /etc/nginx/conf.d/*.conf;   # 加载子配置项
    
    server {
    	listen       80;       # 配置监听的端口
    	server_name  localhost;    # 配置的域名
    	
    	location / {
    		root   /usr/share/nginx/html;  # 网站根目录
    		index  index.html index.htm;   # 默认首页文件
    		deny 172.168.22.11;   # 禁止访问的ip地址，
    		allow 172.168.33.44； # 允许访问的ip地址，可以为all
    	}
    	
    	error_page 500 502 503 504 /50x.html;  # 默认50x对应的访问页面
    	error_page 400 404 error.html;   # 同上
    }
}
```

### 参考资料

1. [Nginx 从入门到实践，万字详解！](https://juejin.cn/post/6844904144235413512#heading-28)
2. [利用 git 的 openssl 模块生成 https 请求的 ssl 测试证书](https://blog.csdn.net/weixin_33701294/article/details/85998586)
3. [windows下nginx配置OpenSSL自签名证书](windows下nginx配置OpenSSL自签名证书)