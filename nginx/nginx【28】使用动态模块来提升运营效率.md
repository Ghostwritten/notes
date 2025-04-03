接下来,我们来查看Nginx的动态模块,动态模块可以帮助我们在使用Nginx的时候,在升级Nginx的时候帮助我们减少编译环节;下面我们来看下动态模块在编译及使用的流程;

　　我们再用一个例子,给大家演示下;
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed5a305d652860582ead6577da78e9d6.png)
们在使用动态模块之前,先来看下在不适用动态模块的方法里,我们是怎么样使用Nginx；

　　首先我们在下载完nginx的源代码,提供了一个叫config的脚本,以及在源代码中介绍的auto目录;这里都在帮助nginx在建立编译系统;那么nginx源代码中提供了很多官方模块,但我们也可能添加许多的第三方模块,不管是官方模块还是第三方模块,这些模块的源码都会和Nginx的框架源码放到一起,进行编译,最后编译出一个nginx可执行文件;那么这是不使用动态模块的一种方式;

　　那么使用了动态模块尼？

　　我们在编译的时候,指定了某些模块,使用动态模块的方式去编译;那么最后尼,除了生成nginx的二进制可执行文件,还会生成一个动态库,也就是我们指定了模块的那个动态库;

　　那么这里我简单介绍下,动态库和静态库主要有 什么区别?

　　静态库会直接把所有的源代码编译进最终的二进制可执行文件中;

　　而动态库尼,在nginx二进制可执行文件里,只保留了它的位置或者说地址;那么在我们需要这个动态库里的功能时尼,由nginx的可执行文件去掉用这个动态库,再去完成这样的功能,所以这里的好处就表现为仅仅需要修改某一个模块或者升级这个模块功能时,特别是当我们的nginx编译了大量的第三方模块,那么这个时候我可以仅仅重新编译这个动态库,而不用去替换我的二进制可执行文件,因为这里很有可能会漏了或者多编译一些nginx模块或者参数使用了错误,而我编译出新的动态库以后,我只要去替换掉这个动态库,然后用ngixn -s reload 一遍;那么我就可以使用新的模块功能了;

　　具体使用的时候,主要为6个步骤;

 - (1):首先,要在nginx的源代码中加入`configure`加入动态模块的时候必须指明这个模块是使用动态模块的的方式编译进nginx中;这里有一个潜台词,不是所有的nginx模块都可以以动态模块的方式加入到nginx中;只有一些模块才可以以动态模块的方式加入;
 - (2):开始执行`make`,编译出`binary`;
 - (3):到第三步的时候,也就是说我们开始启动nginx了;启动nginx的时候尼我们去读`ngx_module`里的数组;
 - (4):读到模块数组中尼,我们发现了使用了一个动态模块,接下来我们会看到一个nginx的conf中加入的一个配置项,这个配置项叫load_module配置;指明了这个　　　
   动态模块所在的路径,
 - (5):那么接下来我们就可以在nginx的进程中打开这个动态库加入模块数组,
 - (6):最后再进行一个初始化的过程(基于模块数组进程初始化);

　　这是动态模块的一个工作流程;

　　接下来我们做一个简单的演示:
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/10e21e88932a13913a3d8c9e0c3dfbd9.png)
 在nginx的源代码目录中:

　　我们先看下哪些模块是支持动态模块的;

　　命令:.`/configure --help|more`

```bash
$ ./configure --help

  --help                             print this message

  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --modules-path=PATH                set modules path
  --conf-path=PATH                   set nginx.conf pathname
  --error-log-path=PATH              set error log pathname
  --pid-path=PATH                    set nginx.pid pathname
  --lock-path=PATH                   set nginx.lock pathname

  --user=USER                        set non-privileged user for
                                     worker processes
  --group=GROUP                      set non-privileged group for
                                     worker processes

  --build=NAME                       set build name
  --builddir=DIR                     set build directory

  --with-select_module               enable select module
  --without-select_module            disable select module
  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module

  --with-threads                     enable thread pool support

  --with-file-aio                    enable file AIO support

  --with-http_ssl_module             enable ngx_http_ssl_module
  --with-http_v2_module              enable ngx_http_v2_module
  --with-http_realip_module          enable ngx_http_realip_module
  --with-http_addition_module        enable ngx_http_addition_module
  --with-http_xslt_module            enable ngx_http_xslt_module
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
  --with-http_image_filter_module    enable ngx_http_image_filter_module
  --with-http_image_filter_module=dynamic
                                     enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-http_sub_module             enable ngx_http_sub_module
  --with-http_dav_module             enable ngx_http_dav_module
  --with-http_flv_module             enable ngx_http_flv_module
  --with-http_mp4_module             enable ngx_http_mp4_module
  --with-http_gunzip_module          enable ngx_http_gunzip_module
  --with-http_gzip_static_module     enable ngx_http_gzip_static_module
  --with-http_auth_request_module    enable ngx_http_auth_request_module
  --with-http_random_index_module    enable ngx_http_random_index_module
  --with-http_secure_link_module     enable ngx_http_secure_link_module
  --with-http_degradation_module     enable ngx_http_degradation_module
  --with-http_slice_module           enable ngx_http_slice_module
  --with-http_stub_status_module     enable ngx_http_stub_status_module

  --without-http_charset_module      disable ngx_http_charset_module
  --without-http_gzip_module         disable ngx_http_gzip_module
  --without-http_ssi_module          disable ngx_http_ssi_module
  --without-http_userid_module       disable ngx_http_userid_module
  --without-http_access_module       disable ngx_http_access_module
  --without-http_auth_basic_module   disable ngx_http_auth_basic_module
  --without-http_mirror_module       disable ngx_http_mirror_module
  --without-http_autoindex_module    disable ngx_http_autoindex_module
  --without-http_geo_module          disable ngx_http_geo_module
  --without-http_map_module          disable ngx_http_map_module
  --without-http_split_clients_module disable ngx_http_split_clients_module
  --without-http_referer_module      disable ngx_http_referer_module
  --without-http_rewrite_module      disable ngx_http_rewrite_module
  --without-http_proxy_module        disable ngx_http_proxy_module
  --without-http_fastcgi_module      disable ngx_http_fastcgi_module
  --without-http_uwsgi_module        disable ngx_http_uwsgi_module
  --without-http_scgi_module         disable ngx_http_scgi_module
  --without-http_grpc_module         disable ngx_http_grpc_module
  --without-http_memcached_module    disable ngx_http_memcached_module
  --without-http_limit_conn_module   disable ngx_http_limit_conn_module
  --without-http_limit_req_module    disable ngx_http_limit_req_module
  --without-http_empty_gif_module    disable ngx_http_empty_gif_module
  --without-http_browser_module      disable ngx_http_browser_module
  --without-http_upstream_hash_module
                                     disable ngx_http_upstream_hash_module
  --without-http_upstream_ip_hash_module
                                     disable ngx_http_upstream_ip_hash_module
  --without-http_upstream_least_conn_module
                                     disable ngx_http_upstream_least_conn_module
  --without-http_upstream_random_module
                                     disable ngx_http_upstream_random_module
  --without-http_upstream_keepalive_module
                                     disable ngx_http_upstream_keepalive_module
  --without-http_upstream_zone_module
                                     disable ngx_http_upstream_zone_module

  --with-http_perl_module            enable ngx_http_perl_module
  --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname

  --http-log-path=PATH               set http access log pathname
  --http-client-body-temp-path=PATH  set path to store
                                     http client request body temporary files
  --http-proxy-temp-path=PATH        set path to store
                                     http proxy temporary files
  --http-fastcgi-temp-path=PATH      set path to store
                                     http fastcgi temporary files
  --http-uwsgi-temp-path=PATH        set path to store
                                     http uwsgi temporary files
  --http-scgi-temp-path=PATH         set path to store
                                     http scgi temporary files

  --without-http                     disable HTTP server
  --without-http-cache               disable HTTP cache

  --with-mail                        enable POP3/IMAP4/SMTP proxy module
  --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
  --with-mail_ssl_module             enable ngx_mail_ssl_module
  --without-mail_pop3_module         disable ngx_mail_pop3_module
  --without-mail_imap_module         disable ngx_mail_imap_module
  --without-mail_smtp_module         disable ngx_mail_smtp_module

  --with-stream                      enable TCP/UDP proxy module
  --with-stream=dynamic              enable dynamic TCP/UDP proxy module
  --with-stream_ssl_module           enable ngx_stream_ssl_module
  --with-stream_realip_module        enable ngx_stream_realip_module
  --with-stream_geoip_module         enable ngx_stream_geoip_module
  --with-stream_geoip_module=dynamic enable dynamic ngx_stream_geoip_module
  --with-stream_ssl_preread_module   enable ngx_stream_ssl_preread_module
  --without-stream_limit_conn_module disable ngx_stream_limit_conn_module
  --without-stream_access_module     disable ngx_stream_access_module
  --without-stream_geo_module        disable ngx_stream_geo_module
  --without-stream_map_module        disable ngx_stream_map_module
  --without-stream_split_clients_module
                                     disable ngx_stream_split_clients_module
  --without-stream_return_module     disable ngx_stream_return_module
  --without-stream_set_module        disable ngx_stream_set_module
  --without-stream_upstream_hash_module
                                     disable ngx_stream_upstream_hash_module
  --without-stream_upstream_least_conn_module
                                     disable ngx_stream_upstream_least_conn_module
  --without-stream_upstream_random_module
                                     disable ngx_stream_upstream_random_module
  --without-stream_upstream_zone_module
                                     disable ngx_stream_upstream_zone_module

  --with-google_perftools_module     enable ngx_google_perftools_module
  --with-cpp_test_module             enable ngx_cpp_test_module

  --add-module=PATH                  enable external module
  --add-dynamic-module=PATH          enable dynamic external module

  --with-compat                      dynamic modules compatibility

  --with-cc=PATH                     set C compiler pathname
  --with-cpp=PATH                    set C preprocessor pathname
  --with-cc-opt=OPTIONS              set additional C compiler options
  --with-ld-opt=OPTIONS              set additional linker options
  --with-cpu-opt=CPU                 build for the specified CPU, valid values:
                                     pentium, pentiumpro, pentium3, pentium4,
                                     athlon, opteron, sparc32, sparc64, ppc64

  --without-pcre                     disable PCRE library usage
  --with-pcre                        force PCRE library usage
  --with-pcre=DIR                    set path to PCRE library sources
  --with-pcre-opt=OPTIONS            set additional build options for PCRE
  --with-pcre-jit                    build PCRE with JIT compilation support

  --with-zlib=DIR                    set path to zlib library sources
  --with-zlib-opt=OPTIONS            set additional build options for zlib
  --with-zlib-asm=CPU                use zlib assembler sources optimized
                                     for the specified CPU, valid values:
                                     pentium, pentiumpro

  --with-libatomic                   force libatomic_ops library usage
  --with-libatomic=DIR               set path to libatomic_ops library sources

  --with-openssl=DIR                 set path to OpenSSL library sources
  --with-openssl-opt=OPTIONS         set additional build options for OpenSSL

  --with-debug                       enable debug logging
```

我们可以看到支持动态模块的这些模块会有一个新增的选项;当我们加--with的时候 可以在最后面加上一个=`dynamic`;

　　下面以`--with-http_image_filter_module=dynamic`这个动态模块为例进行演示;

```bash
./configure --prefix=/usr/local/nginx --with-http_image_filter_module=dynamic
```
执行`make`以后将  `ngx_http_image_filter_module.so`复制到 编译好的nginx目录中新建的`modules` 目录中;
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dcad4562a02580e9cf31fc085f242605.png)
　配置文件中:

首先添加:load_module 模块和对应的路径;

```bash
load_module modules/ngx_http_image_filter_module.so;
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/04c81c9754bc36bf47b6708d7c76d65e.png)
然后对我们访问图片的目录 我们加上 image_filter resize 15 10;  即15*10这个像素;

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1cf03ced6cf41fdc6b1765cd1cf431d3.png)
添加图片文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48701bf6145ae7763170c8ca9350296d.png)
访问这个文件图片:
`--with-http_image_filter_module=dynamic` 这个模块指实时的把这个图片压缩成更小的一些图片;
下面我们以动态模块的方式看看是否能将其变成更小的图片

重新启动nginx,进行访问
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1a67dde24cd7bbcfd61f89c6af3bcc5.png)
图片变小了,说明image_filte模块已经生效了;

使用了动态模块,不需要删除nginx 二进制文件,进行热升级了,可以减少我们出错的效率,但是并非所有的模块都支持动态模块的加载;
