# Nginx - 深入理解Nginx模块开发与架构解析
### Nginx 特点：
1. 更快
   * 在正常情况下，单次请求会得到更快的响应
   * 在高峰时期，Nginx比其他Web服务器更快的响应请求
2. 高扩展性
3. 高可靠性
4. 低内存消耗
5. 单机支持10W以上的并发
6. 热部署
   
## Linux下安装Nginx(基于源代码安装)

### 安装前必备环境
   1. 安装gcc： `yum install -y gcc` gcc编译工具
   2. 安装gcc-c++：`yum install -y gcc-c++` G++编译器
   3. 安装pcre库：`yum install -y pcre pcre-devel` 该库的作用是正则表达式支持。在http模块中如果需要使用正则表达式，那么就需要安装，否则，不需要安装
   4. 安装zlib库：`yum install -y zlib zlib-devel` 该库的作用是对HTTP包的内容做gzip压缩
   5. SSL支持：`yum install -y openssl openssl-devel`
   
### 开始安装
   1. 获取源代码：`wget http://nginx.org/download/nginx-1.14.1.tar.gz`
   2. 解压：`tar -zxvf nginx-1.14.1.tar.gz` 后并进入到此目录
   3. 简单安装，所有配置的参数都使用默认值，只需要执行
      * `./configure`
      * `make`
      * `make install`
   4. 安装完成，可在 /usr/local/nginx 下看到安装的nginx
   
## Nginx基本操作
1. 启动nginx：直接在 `/usr/local/nginx/sbin`目录下执行  `./nginx` 即可，使用默认的 `../conf/nginx.conf` 文件
2. 如果需要指定启动的nginx.conf 文件 那么需要使用 -c 参数：`./nginx -c /tmp/nginx.conf` 这时，会读取/tmp目录下的配置文件来启动
3. 使用 -d 参数指定全局配置，如：`./nginx -g "pid /var/nginx/test.pid"` 会把pid文件写到 `/var/nginx` 下
4. 停止nginx：`./nginx -s stop`
5. 优雅的停止服务 `./nginx -s quit`
6. 重新读取配置项并生效 `./nginx -s reload` 该操作会首先检查配置项是否有误，然后发送一个优雅的关闭信号，在重新启动
7. 日志文件回滚 `./nginx -s reopen` 可以重新打开日志文件，这样可以先把当前日志文件改名或转到其他目录中进行备份

## Nginx配置
1. 语法格式：配置项名 配置项值1 配置项值2 ... ;
2. 配置项单位格式：K(k):KB 或 M(m):MB y(year) d(day) m(秒)
3. 使用变量：在使用变量前加入 $ 如 $remote_addr
4. Nginx服务的基本配置：Nginx在运行时必须加入几个核心模块和一个事件模块，其他所有模块都依赖此基本配置项
   1. 用于调试，定位的配置项
      * 是否以守护进程运行nginx：`daemon on(default) | off;`
      * 是否以master/worker方式工作：`master_process on(default) | off;` 如果参数是off，那么nginx就不会fork worker进程来处理，而是由master来处理
      * error日志设置：`error_log /path/file level;` default:error_log logs/error.log error;
   2. 正常运行必备的配置项
      * 定义环境变量：`env VAR|VAR=VALUE` eg: env TESTPATH=/tmp/
      * 嵌入其他配置文件：`include mime.types;` 将其他配置文件引入到此配置文件中
      * pid文件路径：`pid path/file;` eg: pid logs/nginx_test.pid 保存master进程文件的存放路径
      * Nginx worker 进程运行的用户以及用户组： `user username [groupname];`
         1. eg1: user nobody nobody; 设置到nobody组下的nobody
         2. eg2: user nobody;  设置到nobody组下的nobody，代表用户名和用户组相同
      * 指定Nginx worker进程可以打开的最大句柄描述符： `worker_rlimit_nofile limit;`
      * 限制信号队列：`worker_rlimit_sigpending limit;` 设置每个用户发往Nginx的信号队列的大小
   3. 优化性能配置项
      * Nginx worker进程个数：worker_process 1; 默认为1，此参数和具体的业务相关，如果是较多的阻塞调用，那么可以将其调大一些，否则，将其设置为和Cpu核数相同的个数是比较好的选择
      * 绑定Nginx worker进程到指定的CPU内核：`worker_cpu_affinity cpumask[cpumask...];`
   4. 事件类配置项
      * `use [kqueue|rtsig|epoll|/dev/null|select|poll|eventport];` Nginx会自动选择最合适的事件模型
      * `worker_connections number;` 定义每个进行可以同时处理的最大连接数
      
## 使用Nginx配置一个静态Web服务器
1. 虚拟主机与请求的分发
   * 监听端口：`listen 80;` listen参数决定Nginx如何监听服务的端口,下面列出了可用参数表
      1. default：将所在的server块作为Web服务默认的server块，如果没有设置，那么Nginx会以nginx.conf中配置的第一个server作为默认的
      2. default_server：同default
      3. backlog=num：表示TCP中backlog队列的大小，默认为-1，代表不设置。在TCP建立三次握手过程中，进程还没有开始监听句柄，这时backlog就会放至新连接，如果队列已满，那么就会建立连接失败。
      4. rcvbuf=size：设置监听句柄的SO_RCVBUF
      5. sndbuf=size：设置监听句柄的SO_SNDBUF
      6. accept_filter：设置accept过滤器
      7. deferred：设置该参数（适用于大并发下的场景），若用户已经建立了TCP连接，内核也不会分配和调度worker进程来处理，而是要等到真正的请求（内核已经在网卡中收到数据包）到来才会唤醒worker进程来处理。
      8. bind：绑定当前端口/地址对，如127.0.0.1:8000。只有同时对一个端口监听多个地址才生效。
      9. ssl：在当前端口建立的链接必须是基于SSL协议
   * server_name
      1. 语法：server_name name[...];
      2. 默认：`server_name "";`
      3. 实例：
         * 跟多个主机：`server_name www.testweb.com、download.testweb.com;` 。
         处理请求的步骤和方式：在开始处理一个HTTP请求时，Nginx会取出header中的Host，与每个server_name中进行匹配，由此决定由哪个server来处理。如果一个Host匹配多个server，那么会有如下规则：
            * 1.首先选择完全匹配的server_name，如：www.testweb.com
            * 2.选择通配符在前面的server_name，如：*.testweb.com
            * 3.选择通配符在后面的server_name，如：www.testweb.*
            * 4.使用正则表达式匹配的server_name，如：~^\.testweb\.com$
            * 5.如果都不匹配：那么会按照server中加入了default的server来处理，如果没有，那么就是nginx.conf中配置的第一个server来处理。
   * server_name_hash_bucket_size：为了提高快速找到server-name的能力，Nginx使用散列桶来存储server_name，该值设置了每个散列桶占用的内存大小。
      1. 语法：`server_name_hash_bucket_size size;`
      2. 默认：`server_name_hash_bucket_size 32|64|128`
      3. 配置块：http,server,location
   * `server_name_hash_max_size`：该值主要是影响散列表的冲突率，值越大，消耗的内存就越多，key的冲突率会降低，检索速度就越快。值越小，消耗的内存就越小，冲突会增加
   * 重定向主机名称处理：`server_name_in_redirect on(default)|off`
   * location配置：location会尝试将用户请求中的URI来匹配上面的表达式
      1. 语法：`location [=|~|~*|^~|@] /uri/ {...}`
      2. 匹配规则
         * = 表示把URI作为字符串，做完全匹配：`location = / {}` 表示只有用户的请求为/时，才使用该配置
         * ~ 表示匹配URI时对大小写敏感
         * ~* 表示对大小写不敏感
         * ^~ 表示匹配前半部分即可：`location ^~ /images/ {}` 表示以/images/开始的请求都会匹配上
         * @ 代表Nginx内部的请求之间重定向，不直接处理请求
         * 匹配正则表达式：`location ~* \.(gif|jpg|jpeg)$ {}` 表示匹配gif，jpg，jpeg结尾的请求
         * location请求是有顺序的，当一个请求被多个匹配时，那么这个会交给第一个匹配的来处理。
         * 没匹配到任何请求，一般会加一个 `location / {}` 代表匹配所有请求
2. 文件路径的定义
   * 以root方式设置资源路径
      1. 语法：`root path;`
      2. 默认：root html;
      3. eg：定义资源的Http请求的根目录 
         ```
         location /download/ {
         # 如果有一个请求的URI是/download/index/test.html，那么Web服务器将返回/opt/web/html/download/index/test.html
            root /opt/web/html
         }
         ```
   * 以alias方式设置路径
      1. 语法： `alias path;`
      ```
      # 用户访问/conf/nginx.conf，实际是想访问/usr/local/nginx/nginx.conf
      location /conf {
        alias /usr/local/nginx/conf/;
        # 等同于root设置
        # root /usr/local/nginx/; 
      }
      # 两者的区别
      # location /conf{}
      # 使用alias path;时：location后的/conf会被丢弃掉，如访问/conf/nginx.conf，那么映射成path/nginx.conf
      # 使用root  path;时：location后的/conf会被加上，如访问/conf/nginx.conf，那么映射成path/conf/nginx.conf
      ```
      
   * index file... ：访问首页，后面可以跟多个文件，按照顺序查找
   * error_page code
      ```
      error_page 404 /404.html;
      error_page 502 503 504 /50x.html;
      error_page 403 http://example.com/forbidden.html;
      error_page 403 @fallback; # 交由fallback处理
      location @fallback(
        proxy_pass http://backend;
      )
      ```
3. 内存及磁盘资源的分配
4. 网络连接的设置
5. mime类型的设置
6. 对客户端请求的限制
7. 文件操作的优化
8. 对客户端请求的特殊处理