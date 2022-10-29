# Nginx使用

## 一、缓存

所属模块[ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)

[参考文档](https://www.nginx.com/blog/nginx-caching-guide/)

### 相关配置介绍

#### 1. proxy_cache_path

[官方介绍](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path)

```bash
Syntax:	proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [min_free=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default:	—
Context:	http
```

**关键配置：**

- path 设置缓存文件存放的位置，缓存是以文件形式保存，文件名为配置项`proxy_cache_key`指定的字符串md5后的结果

- levels 设置缓存文件的层级结构，在单个层级下放置大量文件会将低文件的访问速度，可指定1到3级，每一级之间用“:”分隔，每一级的值为1或2，举例说明：
	-	不配置levels时，缓存文件目录结构为：cache/9047f12b3b863bb857eab0cee61a8ef2 
	- levels=1时，缓存文件目录结构为：cache/f/842469aec9f643c2de850a256ad2059f
	- levels=1:2时，缓存文件目录结构为：cache/0/0d/884ecc1703b98d7fec584d2f588da0d0
	- levels=2:2:2时，缓存文件目录结构为：cache/09/81/3a/b2908768d005735285b596a11a3a8109
	**实际使用时，发现个性levels并reload后，只有新的缓存目录会按levels指定的值，老的缓存依旧使用旧的格式，通过停止nginx后再启动可更新，但这种方式相当于服务中断了**

- keys_zone 设置一个用于存储缓存的key以及一个元数据（比如使用情况计数）的共享内存区域，将key保存在内存中可以使nginx快速获知某个请求是否命中缓存，从而避免访问磁盘，1MB约可存储8000个key

- max_size 缓存文件可使用的磁盘空间上限，如果不指定则会占满可用的磁盘空间，指定上限后，当缓存大小达到上限时，会有一个缓存管理的进程按照LRU算法来清理缓存，使之降回上限之下

- inactive 用于指定多久不被访问就从缓存中清除该项（也是通过缓存管理进程完成的），其默认值为10分钟。该时间与缓存的过期时间无关，比如cache-control:max-age=120，过期时间是两分钟，但即使一直没访问，如果inactive没到期，缓存是不会删除的，如果在此期间又被访问了，此时因为已经超过2分钟，nginx会去server端获取新的文件并重置inactive的时间

- use_temp_path nginx会将要缓存的内容先写入一个临时区域，然后再拷贝到指定的缓存存放位置，此项配置设置为off时是告诉nginx直接将要缓存的内容写入缓存存放位置，这样可以避免数据在文件系统中不必要的拷贝操作

```bash
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
```

#### 2. proxy_cache

配置为proxy_cache_path中keys_zone指定的值，从而激活该缓存配置，可以在sever或location级别指定缓存配置，在server级指定的proxy_cache可以被location级的同名配置覆盖

```bash
proxy_cache my_cache;
```

#### 3. proxy_cache_use_stale

用于指定当无法从server端获取新的内容，继续使用缓存的内容，比如如下配置：

```bash
location / {
    # 当server端返回错误、超时或5xx的返回码时，并且缓存中有相应的内容，则直接返回缓存中的内容
    proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
}
```

#### 4. proxy_cache_revalidate

```bash
proxy_cache_revalidate on;
```

相当于开启协商缓存的功能，开启后，nginx会带着If-Modified-Since头去请求server

#### 5. proxy_cache_min_uses

```bash
proxy_cache_min_uses 3;
```

设置触发nginx缓存的最小请求次数，比如上述表示至少请求3次才会被缓存

#### 6. proxy_cache_background_update

```bash
proxy_cache_background_update on;
```

此项与proxy_cache_use_stale配合使用，当开启时，如果客户端请求了一个过期的资源或nginx正在向server端获取最新的资源，则直接将旧的内容先返回，更新内容的操作放在后台进行，直到新的内容下载完成后缓存才会更新

#### 7. proxy_cache_lock

```bash
proxy_cache_lock on;
```

当此项开启时，如果有多个请求，并且缓存未命中，则只有第一个请求会走到sever端，其它请求会等待这个请求完成并从缓存中获取内容并返回

#### 8. 缓存状态标识

```bash
add_header X-Cache-Status $upstream_cache_status;
```

会在响应头X-Cache-Status中反应缓存命中状态

- MISS
- BYPASS
- EXPIRED
- STALE
- UPDATING
- REVALIDATED
- HIT

#### 9. nginx如何决定是否缓存

默认情况下，nginx的缓存行为会遵循server端返回的Cache-Control响应头，当此响应头设置为private、no-cache、no-store或设置有set-cookie响应头时，nginx都不会缓存，并且nginx仅缓存get及head请求，如果proxy_buffering设置为off，也不会开启缓存功能，此配置项默认是on

#### 10. 无视Cache-Control响应头

```bash
location /images/ {
    proxy_cache my_cache;
    proxy_ignore_headers Cache-Control;
    proxy_cache_valid any 30m;
    # ...
}
```

proxy_ignore_headers可用于无视特定的响应头，比如这里的Cache-Control，此时需要同时配置proxy_cache_valid，因为nginx不会缓存没有过期时间内容

proxy_ignore_headers也可用于处理Set-Cookies头

#### 11. proxy_cache_methods

```bash
proxy_cache_methods GET HEAD POST;
```

nginx默认只缓存get、head请求，但可以通过此配置来缓存post请求

#### 12. proxy_cache_bypass

```bash
location / {
    proxy_cache_bypass $cookie_nocache $arg_nocache;
    # ...
}
```

缓存穿透功能，当cookie中设置了nocache或url参数中指定了nocache=true时，穿透缓存，直接请求源站

#### 13. proxy_no_cache

```bash
location / {
    proxy_no_cache $arg_nocache;
    # ...
}
```

用于指定当次请求不进行缓存，这个要与proxy_cache_bypass配合使用，比如绕过缓存请求源站，同时不缓存该次源站的响应内容

该配置项的参数意义与proxy_cache_bypass一致

#### 14. proxy_cache_key

默认情况下，缓存的key为nginx变量组合：$scheme$proxy_host$request_uri的md5码，但可以通过proxy_cache_key来重新指定，比如指定为`proxy_cache_key $proxy_host$request_uri$cookie_jessionid;`来将cookie中的jessionid作为用于生成key的一部分

**注意$proxy_host并不是指实际的host name，而是指proxy_pass中配置的host+端口号，比如：**

```bash
server {
    # ...
    location / {
        proxy_cache my_cache;
        proxy_pass http://my_upstream;
    }
}
```

链接`http://www.example.org/my_image.jpg`对应的$scheme$proxy_host$request_uri是`http://my_upstream:80/my_image.jpg`

## 二、变量

[官方文档](https://nginx.org/en/docs/varindex.html)