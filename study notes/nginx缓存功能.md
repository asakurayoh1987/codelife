调研前端nginx缓存方案，结论及具体实施如下。

 

一、前端缓存需求

1.      能够缓存根HTML文件，避免频繁请求代理层渲染，降低代理层压力，提升性能

2.      能够穿透根HTML缓存，直连代理层渲染以获取最新的HTML，便于现网部署后能够及时进行现网验证

3.      能够控制是否缓存穿透HTML缓存获取的最新的HTML，避免现网部署失败导致影响现网用户

二、Nginx能力及配置

1.      Nginx社区版本提供proxy_cache指令，原生支持文件缓存 

2.      Nginx 社区版本提供proxy_cache_bypass指令，用以控制是否穿透缓存

3.      Nginx社区版本提供proxy_no_cache指令，用以控制是否缓存定义的请求

 

综合如上三个执行，Nginx社区版本可以实现如下功能：

1.      添加proxy_cache指令用以缓存根HTML文件

2.      添加proxy_cache_bypass指令，便于控制指定请求穿透缓存，进行现网部署后验证

3.      添加proxy_no_cache指令，便于控制穿透缓存后的请求的返回是否缓存，保证现网验证通过之后再更新原缓存内容

 

例如，采用如下配置：

proxy_cache_path /usr/local/nginx/cache levels=1:2 keys_zone=cdn_cache:10m max_size=10g inactive=60m use_temp_path=off;

 

location ~* /refactor/.*/friend/ {

                        proxy_cache cdn_cache;
    
                        #proxy_ignore_headers X-Accel-Expires Expires Cache-Control Set-Cookie;
    
                        proxy_cache_valid 200 24h;
    
                        proxy_cache_revalidate on;
    
                        proxy_cache_min_uses 1;
    
                        proxy_cache_use_stale error timeout updating http_500 http_502
    
                                http_503 http_504;
    
                        proxy_cache_background_update on;
    
                        proxy_cache_lock on;
    
                        proxy_cache_key "$scheme$proxy_host$uri";
    
                        proxy_no_cache $cookie_nocache $arg_nocache $http_nocache;
    
                        proxy_cache_bypass $cookie_bypass $arg_bypass $http_bypass;
    
                        add_header X-Cache-Status $upstream_cache_status;

 


                        rewrite /refactor/(.*)/friend/(.*)$ /friend/$2 break;
    
                        proxy_pass http://192.168.57.233:4002;
    
                        proxy_set_header Host $host;
    
                        proxy_set_header X-Real-IP $remote_addr;
    
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
                        access_log logs/refactor.access.log main;
    
                        error_log logs/refactor.error.log error;
    
                }

则：

1.      初次访问地址http://61.191.24.229:18000/refactor/iring.diyring.cc/friend/c5da5439ba307802 时，未命中缓存，直接请求代理层渲染根HTML，对应Response Header中X-Cache-Status: MISS；

2.      再次访问地址http://61.191.24.229:18000/refactor/iring.diyring.cc/friend/c5da5439ba307802 时，命中缓存，直接返回Nginx本地缓存中的文件，对应Response Header中X-Cache-Status: HIT；

3.      再次访问地址http://61.191.24.229:18000/refactor/iring.diyring.cc/friend/c5da5439ba307802?bypass=true&nocache=true 时，穿过缓存请求代理层渲染新的HTML，且不更新Nginx原有的缓存数据，对应Response Header中X-Cache-Status: BYPASS；

4.      再次访问地址http://61.191.24.229:18000/refactor/iring.diyring.cc/friend/c5da5439ba307802?bypass=true时，穿过缓存请求代理层渲染新的HTML，且更新Nginx原有的缓存数据，对应Response Header中X-Cache-Status: BYPASS；

以上，基本满足前端缓存需求。

 

注：安全性问题，

1.      proxy_cache_bypass和proxy_no_cache对应的key可以添加在URL中、header中、cookies中，让外界无法知晓，无法调用；

2.      proxy_cache_bypass和proxy_no_cache对应的key名称可以自行定义，保证穿透缓存的链接外界无法知晓，无法调用；

 

其余参数说明：

1.      proxy_ignore_headers 可以不配置，主要用于控制即使请求头中添加了【Cache-Control: no-cache】等不走缓存的头，仍然走nginx缓存；

2.      proxy_cache_valid 实测此配置必须要有，用于强制缓存过期时间；

3.      proxy_cache_revalidate 性能优化配置，

4.      proxy_cache_min_uses 性能优化配置，控制某个请求至少请求多少次之后才会被缓存；

5.      proxy_cache_use_stale 性能优化配置，控制哪些情况下可以使用过期的缓存；

6.      proxy_cache_background_update 性能优化配置，允许启动后台子请求来更新过期的缓存项，同时将过时的缓存响应返回给客户端；

7.      proxy_cache_lock 性能优化配置，多个客户端同时请求同一个未缓存的资源时，用于控制只发送一个请求到后端，其余请求等待该请求返回并从缓存中获取数据；

8.      proxy_cache_key 必须配置，缓存的key，酷音H5可以使用 【$scheme$proxy_host$uri】；

9.      add_header 非必须，用于添加返回头，本示例用户添加【X-Cache-Status: MISS】头；

 

三、问题及其他

1.      Nginx原生的清理缓存的proxy_cache_purge指令为Nginx Plus的功能，Nginx社区版本不支持；

2.      现网部署成功后，全量更新/清理缓存比较麻烦，需要系统提供另外的功能；
