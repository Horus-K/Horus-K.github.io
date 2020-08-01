---
title: nginx详细编译参数
date: 2020-07-24 13:54:18
tags: nginx
---

在nginx源码通过`./configure`获得所有编译参数

<!--more-->

```bash
--prefix= 指向安装目录
--sbin-path 指向（执行）程序文件（nginx）
--conf-path= 指向配置文件（nginx.conf）
--error-log-path= 指向错误日志目录
--pid-path= 指向pid文件（nginx.pid）
--lock-path= 指向lock文件（nginx.lock）（安装文件锁定，防止安装文件被别人利用，或自己误操作。）
--user= 指定程序运行时的非特权用户
--group= 指定程序运行时的非特权用户组
--builddir= 指向编译目录
--with-rtsig_module 启用rtsig模块支持（实时信号）
--with-select_module 启用select模块支持（一种轮询模式,不推荐在高载环境下使用）禁用：--without-select_module
--with-poll_module 启用poll模块支持（功能与select相同，与select特性相同，为一种轮询模式,不推荐在高载环境下使用）
--with-file-aio 启用file aio支持（一种APL文件传输格式）
--with-ipv6 启用ipv6支持
--with-http_ssl_module 启用ngx_http_ssl_module支持（使支持https请求，需已安装openssl）
--with-http_realip_module 启用ngx_http_realip_module支持（这个模块允许从请求标头更改客户端的IP地址值，默认为关）
--with-http_addition_module 启用ngx_http_addition_module支持（作为一个输出过滤器，支持不完全缓冲，分部分响应请求）
--with-http_xslt_module 启用ngx_http_xslt_module支持（过滤转换XML请求）
--with-http_image_filter_module 启用ngx_http_image_filter_module支持（传输JPEG/GIF/PNG 图片的一个过滤器）（默认为不启用。gd库要用到）
--with-http_geoip_module 启用ngx_http_geoip_module支持（该模块创建基于与MaxMind GeoIP二进制文件相配的客户端IP地址的ngx_http_geoip_module变量）
--with-http_sub_module 启用ngx_http_sub_module支持（允许用一些其他文本替换nginx响应中的一些文本）
--with-http_dav_module 启用ngx_http_dav_module支持（增加PUT,DELETE,MKCOL：创建集合,COPY和MOVE方法）默认情况下为关闭，需编译开启
--with-http_flv_module 启用ngx_http_flv_module支持（提供寻求内存使用基于时间的偏移量文件）
--with-http_gzip_static_module 启用ngx_http_gzip_static_module支持（在线实时压缩输出数据流）
--with-http_random_index_module 启用ngx_http_random_index_module支持（从目录中随机挑选一个目录索引）
--with-http_secure_link_module 启用ngx_http_secure_link_module支持（计算和检查要求所需的安全链接网址）
--with-http_degradation_module  启用ngx_http_degradation_module支持（允许在内存不足的情况下返回204或444码）
--with-http_stub_status_module 启用ngx_http_stub_status_module支持（获取nginx自上次启动以来的工作状态）
--without-http_charset_module 禁用ngx_http_charset_module支持（重新编码web页面，但只能是一个方向--服务器端到客户端，并且只有一个字节的编码可以被重新编码）
--without-http_gzip_module 禁用ngx_http_gzip_module支持（该模块同-with-http_gzip_static_module功能一样）
--without-http_ssi_module 禁用ngx_http_ssi_module支持（该模块提供了一个在输入端处理处理服务器包含文件（SSI）的过滤器，目前支持SSI命令的列表是不完整的）
--without-http_userid_module 禁用ngx_http_userid_module支持（该模块用来处理用来确定客户端后续请求的cookies）
--without-http_access_module 禁用ngx_http_access_module支持（该模块提供了一个简单的基于主机的访问控制。允许/拒绝基于ip地址）
--without-http_auth_basic_module禁用ngx_http_auth_basic_module（该模块是可以使用用户名和密码基于http基本认证方法来保护你的站点或其部分内容）
--without-http_autoindex_module 禁用disable ngx_http_autoindex_module支持（该模块用于自动生成目录列表，只在ngx_http_index_module模块未找到索引文件时发出请求。）
--without-http_geo_module 禁用ngx_http_geo_module支持（创建一些变量，其值依赖于客户端的IP地址）
--without-http_map_module 禁用ngx_http_map_module支持（使用任意的键/值对设置配置变量）
--without-http_split_clients_module 禁用ngx_http_split_clients_module支持（该模块用来基于某些条件划分用户。条件如：ip地址、报头、cookies等等）
--without-http_referer_module 禁用disable ngx_http_referer_module支持（该模块用来过滤请求，拒绝报头中Referer值不正确的请求）
--without-http_rewrite_module 禁用ngx_http_rewrite_module支持（该模块允许使用正则表达式改变URI，并且根据变量来转向以及选择配置。如果在server级别设置该选项，那么他们将在 location之前生效。如果在location还有更进一步的重写规则，location部分的规则依然会被执行。如果这个URI重写是因为location部分的规则造成的，那么 location部分会再次被执行作为新的URI。 这个循环会执行10次，然后Nginx会返回一个500错误。）
--without-http_proxy_module 禁用ngx_http_proxy_module支持（有关代理服务器）
--without-http_fastcgi_module 禁用ngx_http_fastcgi_module支持（该模块允许Nginx 与FastCGI 进程交互，并通过传递参数来控制FastCGI 进程工作。 ）FastCGI一个常驻型的公共网关接口。
--without-http_uwsgi_module 禁用ngx_http_uwsgi_module支持（该模块用来医用uwsgi协议，uWSGI服务器相关）
--without-http_scgi_module 禁用ngx_http_scgi_module支持（该模块用来启用SCGI协议支持，SCGI协议是CGI协议的替代。它是一种应用程序与HTTP服务接口标准。它有些像FastCGI但他的设计 更容易实现。）
--without-http_memcached_module 禁用ngx_http_memcached_module支持（该模块用来提供简单的缓存，以提高系统效率）
-without-http_limit_zone_module 禁用ngx_http_limit_zone_module支持（该模块可以针对条件，进行会话的并发连接数控制）
--without-http_limit_req_module 禁用ngx_http_limit_req_module支持（该模块允许你对于一个地址进行请求数量的限制用一个给定的session或一个特定的事件）
--without-http_empty_gif_module 禁用ngx_http_empty_gif_module支持（该模块在内存中常驻了一个1*1的透明GIF图像，可以被非常快速的调用）
--without-http_browser_module 禁用ngx_http_browser_module支持（该模块用来创建依赖于请求报头的值。如果浏览器为modern ，则$modern_browser等于modern_browser_value指令分配的值；如 果浏览器为old，则$ancient_browser等于 ancient_browser_value指令分配的值；如果浏览器为 MSIE中的任意版本，则 $msie等于1）
--without-http_upstream_ip_hash_module 禁用ngx_http_upstream_ip_hash_module支持（该模块用于简单的负载均衡）
--with-http_perl_module 启用ngx_http_perl_module支持（该模块使nginx可以直接使用perl或通过ssi调用perl）
--with-perl_modules_path= 设定perl模块路径
--with-perl= 设定perl库文件路径
--http-log-path= 设定access log路径
--http-client-body-temp-path= 设定http客户端请求临时文件路径
--http-proxy-temp-path= 设定http代理临时文件路径
--http-fastcgi-temp-path= 设定http fastcgi临时文件路径
--http-uwsgi-temp-path= 设定http uwsgi临时文件路径
--http-scgi-temp-path= 设定http scgi临时文件路径
-without-http 禁用http server功能
--without-http-cache 禁用http cache功能
--with-mail 启用POP3/IMAP4/SMTP代理模块支持
--with-mail_ssl_module 启用ngx_mail_ssl_module支持
--without-mail_pop3_module 禁用pop3协议（POP3即邮局协议的第3个版本,它是规定个人计算机如何连接到互联网上的邮件服务器进行收发邮件的协议。是因特网电子邮件的第一个离线协议标 准,POP3协议允许用户从服务器上把邮件存储到本地主机上,同时根据客户端的操作删除或保存在邮件服务器上的邮件。POP3协议是TCP/IP协议族中的一员，主要用于 支持使用客户端远程管理在服务器上的电子邮件）
--without-mail_imap_module 禁用imap协议（一种邮件获取协议。它的主要作用是邮件客户端可以通过这种协议从邮件服务器上获取邮件的信息，下载邮件等。IMAP协议运行在TCP/IP协议之上， 使用的端口是143。它与POP3协议的主要区别是用户可以不用把所有的邮件全部下载，可以通过客户端直接对服务器上的邮件进行操作。）
--without-mail_smtp_module 禁用smtp协议（SMTP即简单邮件传输协议,它是一组用于由源地址到目的地址传送邮件的规则，由它来控制信件的中转方式。SMTP协议属于TCP/IP协议族，它帮助每台计算机在发送或中转信件时找到下一个目的地。）
--with-google_perftools_module 启用ngx_google_perftools_module支持（调试用，剖析程序性能瓶颈）
--with-cpp_test_module 启用ngx_cpp_test_module支持
--add-module= 启用外部模块支持
--with-cc= 指向C编译器路径
--with-cpp= 指向C预处理路径
--with-cc-opt= 设置C编译器参数（PCRE库，需要指定–with-cc-opt=”-I /usr/local/include”，如果使用select()函数则需要同时增加文件描述符数量，可以通过–with-cc- opt=”-D FD_SETSIZE=2048”指定。）
--with-ld-opt= 设置连接文件参数。（PCRE库，需要指定–with-ld-opt=”-L /usr/local/lib”。）
--with-cpu-opt= 指定编译的CPU，可用的值为: pentium, pentiumpro, pentium3, pentium4, athlon, opteron, amd64, sparc32, sparc64, ppc64
--without-pcre 禁用pcre库
--with-pcre 启用pcre库
--with-pcre= 指向pcre库文件目录
--with-pcre-opt= 在编译时为pcre库设置附加参数
--with-md5= 指向md5库文件目录（消息摘要算法第五版，用以提供消息的完整性保护）
--with-md5-opt= 在编译时为md5库设置附加参数
--with-md5-asm 使用md5汇编源
--with-sha1= 指向sha1库目录（数字签名算法，主要用于数字签名）
--with-sha1-opt= 在编译时为sha1库设置附加参数
--with-sha1-asm 使用sha1汇编源
--with-zlib= 指向zlib库目录
--with-zlib-opt= 在编译时为zlib设置附加参数
--with-zlib-asm= 为指定的CPU使用zlib汇编源进行优化，CPU类型为pentium, pentiumpro
--with-libatomic 为原子内存的更新操作的实现提供一个架构
--with-libatomic= 指向libatomic_ops安装目录
--with-openssl= 指向openssl安装目录
--with-openssl-opt 在编译时为openssl设置附加参数
--with-debug 启用debug日志
```

一般采用：

```sh
./configure \
--prefix=/Data/apps/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-pcre \
--with-stream \
--with-stream_ssl_module \
--with-stream_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_auth_request_module \
--with-threads \
--with-stream \
--with-http_slice_module \
--with-mail \
--with-mail_ssl_module \
--with-file-aio 
```

别人的：

```sh
本参数基于Nginx-1.15.2安装包

一 路径参数

1.1.1 指定Nginx安装路径

--prefix=/usr/local/nginx-1.15.2
1.1.2 设置Nginx可执行文件路径默认路径在 --prefix指定的路径下的bin

--sbin-path=PATH
1.1.3 设置Nginx模块路径

--modules-path=PATH 
1.1.4 设置在nginx.conf配置文件的路径 默认路径：--prefix指定的路径下的conf

 --conf-path=PATH
1.1.5 设置主错误，警告等文件日志路径 默认在：--prefix指定的路径下的logs 安装完成后随配置文件改变

--error-log-path=PATH
1.1.6 设置Nginx Pid文件路径  默认：--prefix指定的路径下的logs/nginx.pid安装完成后随配置文件改变

 --pid-path=PATH 
1.1.7 设置Nginx锁文件路径     nginx.lock是nginx保证只有1个服务运行的锁文件, nginx启动时会自动创建

--lock-path=PATH 
1.1.8 设置编译的名字

--build=NAME 
1.1.9 设置编译路径

 --builddir=DIR
1.2.0  设置perl模块路径 和设置perl库文件路径

  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname
1.2.1 设置日志路径 可在配置文件设置

--http-log-path=path
1.2.2 设置客户端请求临时文件路径 可在配置文件设置

--http-client-body-temp-path=PATH
1.2.3 设置http proxy临时文件路径 可在配置文件设置

--http-proxy-temp-path=PATH
1.2.4 设置http fastcgi临时文件路径 可在配置文件设置

--http-fastcgi-temp-path=PATH
1.2.5 定义了一个目录来存储临时文件 收到uwsgi服务器数据 可在配置文件设置

--http-uwsgi-temp-path=PATH
1.2.6 定义了一个目录来存储临时文件 收到SCGI服务器数据 可在配置文件设置

--http-scgi-temp-path=PATH
 1.2.7 使用外部模块路径

--add-module=path
1.2.8 使外部动态模块路径

--add-dynamic-module=path
1.2.9 设置C编译器路径

--with-cc=PATH 
1.3.0 设置C预处理路径

--with-cc-opt=OPTIONS 
1.3.1  设置C编译器参数 设置其他参数,将被添加到CFLAGS变量

--with-cc-opt=parameters
1.3.2 设置连接文件参数 --with-ld-opt="-L /usr/local/lib"应该被指定。

--with-ld-opt=parameters
1.3.3 每个指定的CPU使建筑:pentium,pentiumpro,pentium3,pentium4,athlon,opteron,sparc32,sparc64,ppc64。

--with-cpu-opt=cpu
1.3.4 不使用pcre库文件，pcre是包括 perl 兼容的正规表达式库

--without-pcre
1.3.4 启用pcre

--with-pcre
1.3.4 设置pcre库路径

--with-pcre=path
1.3.4 设置pcre运行参数

--with-pcre-opt=parameters
1.3.4 即时编译pcre

--with-pcre-jit
1.3.5 设置zlib库路径

--with-zlib=path
--with-zlib-opt=parameters
--with-zlib-asm=cpu                  #允许使用zlib汇编优化的来源 为指定的cpu之一:pentium,pentiumpro
1.3.6 libatomic 

1
2
--with-libatomic                     #启用
--with-libatomic=path                 #libatomic路径
1.3.7 设置openssl

--with-openssl=path                  #设置路径OpenSSL库来源。
--with-openssl-opt=parameters        #OpenSSL设置额外的构建选项。
1.3.8使调试日志

 --with-debug
二 设置用户系列

2.1.1 指定nginx运行用户

--user=USER
2.1.2 指定Nginx运行组

--user=USER
三 模块设置
3.1.1 允许或不允许开启SELECT模式

  --with-select_module               enable select module
  --without-select_module            disable select module
3.1.2 启用或禁用编译模块,允许服务器工作 与poll()方法。 这个模块是建立自动如果平台不出现 支持更适当的方法如kqueue,epoll或/dev/poll.

  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module
3.1.3 允许的线程池

 --with-threads                     enable thread pool support
3.1.4 允许使用的异步文件I / O(AIO)在FreeBSD和Linux。

 --with-file-aio                    enable file AIO support
3.1.5 启用 ngx_http_ssl_module 支持（使支持 https 请求，需已安装openssl）

--with-http_ssl_module 
3.1.6 供支持HTTP / 2

--with-http_v2_module
3.1. 7 启用 ngx_http_realip_module 支持（这个模块允许从请求标头更改客户端的 IP 地址值，默认为关）

--with-http_v2_module
3.1.8 启用 ngx_http_addition_module 支持（作为一个输出过滤器，支持不完全缓冲，分部分响应请求）

--with-http_realip_module 
3.1.9 启用 ngx_http_addition_module 支持（作为一个输出过滤器，支持不完全缓冲，分部分响应请求）

--with-http_realip_module
3.2.0 在响应之前或者之后追加文本内容。 这个模块不是建立在默认情况下是启用

--with-http_addition_module 
3.2.1 这个模块是一个过滤器，它可以通过XSLT模板转换XML应答 默认用 这个模块需要libxml2和libxslt库。支持

  --with-http_xslt_module            enable ngx_http_xslt_module
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
3.2.2  http_image_filter_module是nginx提供的集成图片处理模块，支持nginx-0.7.54以后的版本，在网站访问量不是很高磁盘有限不想生成多余的图片文件的前提下可，就可以用它实时缩放图片，旋转图片，验证图片有效性以及获取图片宽高以及图片类型信息 默认用

  --with-http_image_filter_module            enable ngx_http_image_filter_module
  --with-http_image_filter_module=dynamic    enable dynamic ngx_http_image_filter_module
3.2.3 取值于客户端IP地址,使用预编译MaxMind数据库 可以禁用指定国家访问

  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
3.2.4 启用 ngx_http_sub_module 支持（允许用一些其他文本替换nginx 响应中的一些文本）

--with-http_sub_module
3.2.5 启用ngx_http_dav_module支持（增加PUT,DELETE,MKCOL：创建集合,COPY 和 MOVE 方法为文件和目录指定权限，限制不同类型的用户对于页面有不同的操作权限）默认情况下为关闭，需编译开启

--with-http_dav_module
3.2.6 启用 ngx_http_flv_module 支持（提供寻求内存使用基于时间的偏移量文件,这个模块支持对FLV（flash）文件的拖动播放）

--with-http_flv_module
3.2.7 模块提供pseudo-streaming服务器端支持 MP4文件

--with-http_mp4_module
3.2.8 解压缩响应 以“Content-Encoding: gzip” 为客户,不支持gzip编码方法

--with-http_gunzip_module
3.2.9 启用 ngx_http_gzip_static_module 支持（在线实时压缩输出数据流,这个模块在一个预压缩文件传送到开启Gzip压缩的客户端之前检查是否已经存在以“.gz”结尾的压缩文件，这样可以防止文件被重复压缩。）

--with-http_gzip_static_module 
3.3.0 实现客户端授权 基于subrequest的结果

--with-http_auth_request_module
3.3.1 从目录中选择一个随机主页

--with-http_random_index_module
3.3.2 防止盗链下载

 --with-http_secure_link_module
3.3.3 CDN系统中，向父层回源时，如果文件过大，使用slice可以让用户快速得到响应。

  --with-http_slice_module 
3.3.4 启用ngx_http_stub_status_module支持（获取nginx自上次启动以来的工作状态）

--with-http_stub_status_module
3.3.5 Nginx配置能够扩展使用Perl代码。这个选项启用这个模块（然而使用这个模块会降低性能）

  --with-http_perl_module
   --with-http_perl_module                enable ngx_http_perl_module
   --with-http_perl_module=dynamic        enable dynamic ngx_http_perl_module

3.3.6 模块开启mysql外网地址tcp/udp代理stream

  --with-stream                      enable TCP/UDP proxy module
  --with-stream=dynamic              enable dynamic TCP/UDP proxy module
3.3.7 为代理流服务器提供必要的SSL/TLS协议支持。该模块默认不会启用，需要通过–with-stream_ssl_module配置参数启用

--with-stream_ssl_module
3.3.8 处理来自client客户端的TCP/UDP session 

--with-stream_realip_module
3.3.9 使用Nginx 和 GeoIP 模块的可以来处理不同地区的访问 需要MaxMind数据库支持

--with-stream_geoip_module
--with-stream_geoip_module=dynamic
3.4.0 允许提取的信息ClientHello消息没有终止SSL / TLS

 --with-stream_ssl_preread_module
3.4.1 模块,使分析nginx工作进程使用谷歌性能工具。 模块适用于nginx开发人员

--with-google_perftools_module
3.4.2 使建筑ngx_cpp_test_module模块

--with-cpp_test_module 
四 邮件设置
4.1.1 允许POP3/IMAP4/SMTP代理模块

 --with-mail 
 --with-mail=dynamic  
4.1.2 支持建立模块,添加了SSL / TLS协议的支持邮件代理服务器 OpenSSL库支持

  --with-mail_ssl_module  
4.1.3 禁用POP3协议 在邮件代理服务器

--without-mail_pop3_module 
4.1.4 禁用IMAP协议 在邮件代理服务器

--without-mail_imap_module
4.1.5 禁用SMTP协议 在邮件代理服务器

--without-mail_smtp_module
 

五 一些禁用参数默认开启除特殊情况外关闭
5.1.1 限制连接数ngx_http_limit_conn_module模块

--without-stream_limit_conn_module
5.1.2 禁止 允许限制访问某些客户端地址的 ngx_stream_access_module模块

--without-stream_access_module 
5.1.3根据客户端IP地址创建变量述指定变量的值与客户端IP地址之间的依赖关系。默认情况下，地址从$remote_addr变量中获取

 --without-stream_geo_module
 

5.1.4  模块创建变量 值取决于其他变量的值

--without-stream_map_module
5.1.5 模块为A / B测试创建变量

--without-stream_split_clients_module
5.1.6  模块,向客户端发送一些指定值 然后关闭连接

 --without-stream_return_module
5.1.7 禁用模块实现哈希负载平衡方法

--without-stream_upstream_hash_module
5.1.8 禁用模块实现least_conn负载平衡方法

--without-stream_upstream_least_conn_module
5.1.9 禁止模块可以存储运行时状态 一群上游的共享内存区

--without-stream_upstream_random_module
5.2.0 默认开启 下面禁用方法 这个模块将在应答头中为"Content-Type"字段添加字符编码  另外,可以将数据从一个字符集转换为另一种形式。

--without-http_charset_module
5.2.1 默认开启 下面禁用方法 不使用ngx_http_gzip_module模块，文件压缩模式

 --without-http_gzip_module 
5.2.2 默认开启 下面禁用方法  那压缩响应一个HTTP服务器。 zlib库需要构建并运行该模块

--without-http_ssi_module 
5.2.3 默认开启 下面禁用方法 适合客户端识别。 收到并设置cookie可以登录使用嵌入的变量

--without-http_userid_module
5.2.4 默认开启 下面禁用方法 模块允许限制访问某些客户端地址

--without-http_access_module 
5.2.5 默认开启 下面禁用方法 验证模块,允许限制对资源的访问的用户名 和密码使用HTTP基本身份验证协议。默认开启

--without-http_auth_basic_module
5.2.6 默认开启 下面禁用方法 模块ngx_http_auth_basic_module允许使用“HTTP基本认证”协议验证用户名和密码来限制对资源的访问 默认开启

--without-http_mirror_module
5.2.7 默认开启 下面禁用方法  禁用自动索引模块

--without-http_autoindex_module
5.2.8 默认开启 下面禁用方法  

该模块允许你定义

依赖于IP地址段的变量

--without-http_geo_module 
5.2.9默认开启 下面禁用方法  禁用Map模块，该模块允许你声明map区段

 --without-http_map_module 
5.3.0 默认开启 下面禁用方法 对比测试

--without-http_split_clients_module
5.3.1  不使用HTTP server功能，如果只是做代理服务器，可以不提供http服务  和禁用HTTP缓存。

  --without-http                     disable HTTP server
  --without-http-cache               disable HTTP cache
```

