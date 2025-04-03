 刚刚我们谈到nginx不同的worker进程间需要共享信息的时候,需要通过共享内存;我们也谈到了共享内存上可以使用**链表**或者**红黑树**这样的数据结构;但是每一个红黑树上有许多节点;每一个节点你都需要分配内存去存放;那么怎么样把一整块共享内存切割成一小块给红黑树上的每一个节点使用尼?

　　下面我们来看下Slab内存分配管理是怎么样应用于共享内存上的;首先我们来看下**Slab**内存管理是怎么样的一种形式;
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03cd9de0b6c0de6fc68c38add06b461c.png)
它首先会把整块的共享内存分为很多**页面**;那么每个页面例如**4k**;会切分为很多**slot**;比如**32字节是一种slot**;**64字节又是一种slot;128字节又是一种slot**;那么这些slot是以乘2的方式向上增长的;如果现在有一个51字节需要分配的内存会放到哪里尼?会放于小于它最大的一个slot的一个环节;比如说64字节;所以上图中slot就是指向不同大小的块;所以这样的一种数据结构尼 它有一个特点;会有内存的浪费的;就像我们刚刚所说的;51字节它会用64字节来存放;那么其它的13字节就浪费了;那么最多会有多少内存消耗尼?会有两倍;这种使用的方式叫做`Bestfit`;**Bestfit这种分配内存的方式有什么好处尼?它适合小对象;如果我们要分配的对象的内存非常小,比如小于一个页面的大小,就非常合适;因为它很少有碎片,那么每分配一块内存,就会沿着还未分配的空白的地方继续使用就可以了;当一个页面使用满以后,我再拿一个空白的页面继续给此类slot大小的内存继续使用就可以.那么有时候我分配在某段内存上的数据结构它是固定的,甚至需要初始化;那么这样的话,原先的数据结构都还在;当我重复使用的话,也避免了初始化;Slab内存管理中,我们怎么做数据的监控和统计尼?**

　　那么tng上有一个模块叫做`slab_stat`;`slab_stat`可以帮我们看不同的slot;
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf461a69a086a3606d7a181f1ecde3af.png)
比如说:8字节 16 字节 。。。。等等;

　　一共目前分配了多少,使用了多少,有多少个请求在访问,失败了多少次,这个对我们来监控Slab是非常有用处的;

　　下面我们来看下怎么样在openresty的场景下去使用tng上的slab_stat这个模块;

　　首先我们打开tengine的页面   http://tengine.taobao.org/document/ngx_slab_stat.html

　　但是会发现在这个模块上没有github的地址;也就是说它没有作为一个独立的模块提供出来;那这个时候该怎么办尼?

　　那么tengine怎么下载下来?从download里;

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5e61b091748ed11478e782b8f068c63a.png)

```bash
$ wget http://tengine.taobao.org/download/tengine-2.3.3.tar.gz
$ tar -zxvf tengine-2.3.3.tar.gz
$ cd tengine-2.3.3/
$ ls
AUTHORS.te  auto  CHANGES  CHANGES.cn  CHANGES.te  conf  configure  contrib  docs  html  LICENSE  man  modules  packages  README.markdown  src  tests  THANKS.te

```
 modules 里有个`ngx_slab_stat`
再进入查看

```bash
$ cd modules/
$ ls -l
总用量 4
drwxrwxr-x 2 root root   20 3月  29 2021 mod_config
drwxrwxr-x 4 root root  192 3月  29 2021 mod_dubbo
drwxrwxr-x 2 root root   50 3月  29 2021 ngx_backtrace_module
drwxrwxr-x 3 root root  127 3月  29 2021 ngx_debug_pool
drwxrwxr-x 3 root root  100 3月  29 2021 ngx_debug_timer
drwxrwxr-x 2 root root   52 3月  29 2021 ngx_http_concat_module
drwxrwxr-x 2 root root   59 3月  29 2021 ngx_http_footer_filter_module
drwxrwxr-x 9 root root  151 3月  29 2021 ngx_http_lua_module
drwxrwxr-x 3 root root   68 3月  29 2021 ngx_http_proxy_connect_module
drwxrwxr-x 2 root root   79 3月  29 2021 ngx_http_reqstat_module
drwxrwxr-x 2 root root   51 3月  29 2021 ngx_http_slice_module
drwxrwxr-x 2 root root   54 3月  29 2021 ngx_http_sysguard_module
drwxrwxr-x 2 root root 4096 3月  29 2021 ngx_http_tfs_module
drwxrwxr-x 2 root root   57 3月  29 2021 ngx_http_trim_filter_module
drwxrwxr-x 2 root root  100 3月  29 2021 ngx_http_upstream_check_module
drwxrwxr-x 2 root root   70 3月  29 2021 ngx_http_upstream_consistent_hash_module
drwxrwxr-x 2 root root   62 3月  29 2021 ngx_http_upstream_dynamic_module
drwxrwxr-x 2 root root  131 3月  29 2021 ngx_http_upstream_dyups_module
drwxrwxr-x 2 root root  134 3月  29 2021 ngx_http_upstream_keepalive_module
drwxrwxr-x 2 root root   69 3月  29 2021 ngx_http_upstream_session_sticky_module
drwxrwxr-x 2 root root   61 3月  29 2021 ngx_http_upstream_vnswrr_module
drwxrwxr-x 2 root root   56 3月  29 2021 ngx_http_user_agent_module
drwxrwxr-x 2 root root  287 3月  29 2021 ngx_multi_upstream_module
drwxrwxr-x 3 root root  121 3月  29 2021 ngx_slab_stat
```

```bash
$ cd ngx_slab_stat/
$ ls -l
总用量 44
-rw-rw-r-- 1 root root   204 3月  29 2021 config
-rw-rw-r-- 1 root root  6627 3月  29 2021 ngx_http_slab_stat_module.c
-rw-rw-r-- 1 root root  3465 3月  29 2021 README.cn
-rw-rw-r-- 1 root root  3501 3月  29 2021 README.md
-rw-rw-r-- 1 root root 21430 3月  29 2021 slab_stat.patch
drwxrwxr-x 2 root root    20 3月  29 2021 t
```
可以看到这是一个标准的nginx第三方模块;因为每一个nginx第三方模块会通过一个`ngx_http_slab_stat_module.c`文件定义好我们之前所说的ngx_module_t这么一个结构体;

　　以及处理哪些配置项 ,提供哪些变量;并有一个`config`来帮助他编译到我们的目标nginx中 ;现在我们再回到 `openresty`中,让`openresty`编译的时候,把`tengine`的`slab_stat`模块编译进去;然后再使用openresty上的share_direct去分配共享内存;再用slab_stat查看我们共享内存的使用情况;

　　我们执行configure的时候,可以添加一个参数叫 `--add-module=`;--add-module=这个命令尼 可以将一个目录下具备config的这样一个配置项或者目录添加到我们的nginx中,它的方式尼 也就是将这个模块的nginx源码能够使我们的`./configure` 识别到;所以这里的configure和我们的官方configure是一样的,那么我们现在开始编译openresty;

```bash
$ cd /root/nginx/openresty-1.19.3.2
$ ./configure --add-module=../tengine-2.3.3/modules/ngx_slab_stat/
$ make
```
如果之前已经安装了openresty或者(nginx)

执行编译make 好以后不需要make install

(1):备份原来的nginx 可执行文件

```bash
cd /usr/local/openresty/nginx/sbin

cp nginx nginx.old
```

(2):将编译好的 nginx 可执行二进制文件复制到原始 nginx 的 sbin 目录

```bash
cp /home/web/openresty-1.13.6.2/build/nginx-1.13.6/objs/nginx /usr/local/openresty/nginx/sbin/ -f
```

(3):验证是否成功安装 ngx_slab_stat

```bash
cd /usr/local/openresty/nginx/sbin

./nginx -V
```
现在openresty已经安装完成了,这个openresty中,已经包含了`ngx_slab_stat`模块;

　　我们以上一节中`lua_shared_dict` 那段代码的例子,来讲解下`slab_stat`是怎么使用的?

　　首先我们用 `lua_shared_dict` dogs 10m; 分配了一个10m的共享内存,名字叫`dogs`;

　　使用的话,我们有一个set的url下;设置了一个`key-value`:Jim-8;设置在我们的共享内存下;并返回给用户一个字符串叫STORED;

　　当收到get这个url请求时,会从dogs 共享内存中取出Jim的值;也就是这里设置的8返回给用户;

　　然后我们又增加了一个location;这个`location`叫做`slab_stat`;它里面的内容尼就是`tengine`中`slab_stat`提供的slab_stat配置项,这个配置项就叫做`slab_stat`;

　　它会返回我们slab的所有的统计状况;

nginx.conf 配置文件代码:

```bash
　lua_shared_dict dogs 10m;
server {
    listen 8080;
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;
    location = /slab_stat{
     slab_stat;
    }
    location /set{
            content_by_lua_block{
                    local dogs=ngx.shared.dogs
                    dogs:set("Jim",8)
                    ngx.say("STORED");
            }
    }

    location /get {
            content_by_lua_block{
                    local dogs=ngx.shared.dogs
                    ngx.say(dogs:get("Jim"))
            }
    }
    location / {
        #proxy_set_header Host host;             #proxy_set_header X-Real-IPremote_addr;
        #proxy_set_header X-Forwarded-For proxy_add_x_forwarded_for;             #proxy_cache my_cache;             #proxy_cache_keyhosturiis_args&args;
        #proxy_cache_valid 200 304 302 1d;
        #proxy_pass http://local;
    }
```
配置项写完,把ngixn启动看看它的执行效果;

　　每一个slot及其slot对应的大小;分配了多少个,使用了多少个,失败了多少个;

　　所谓分配就是10m是一个非常大的共享内存,它会划分很多个页面;对于比较小的比如32字节,一个页面可以有128个;这里127可用,已经使用了一个;

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df85efa4e3d7d6e3dec826aa7567eff9.png)
总结:以上我们介绍了Slab内存的使用方法:
**slab使用了Bestfit思想,它也是Linux操作系统经常使用的内存分配方式;**

　　**那么通常我们在使用共享内存时,都需要使用slab_stat去分配相应的内存给对象,再使用上层的数据结构来维护这些数据对象;**
