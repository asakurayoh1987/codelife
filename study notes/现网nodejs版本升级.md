# CentOS7环境下Node.js版本升级记录

## 一、升级gcc到8.3.0

```bash
# 也可使用阿里云镜像站：https://mirrors.aliyun.com/gnu/gcc/gcc-8.3.0/gcc-8.3.0.tar.gz
wget https://ftp.gnu.org/gnu/gcc/gcc-8.3.0/gcc-8.3.0.tar.gz
tar zxvf gcc-8.3.0.tar.gz

cd gcc-8.3.0
./contrib/download_prerequisites
mkdir build
cd build
../configure --prefix=/usr/local/gcc-8.3.0 --enable-checking=release --enable-languages=c,c++ --disable-multilib

# 这里可以指定实际CPU核心数 
make -j4
make install

# 修改环境变量
echo -e '\n/usr/local/gcc-8.3.0/bin\n' >> /etc/profile

# 添加动态链接库配置
echo -e '\n/usr/local/gcc-8.3.0/lib64\n' >> /etc/ld.so.conf.d/gcc.conf
ldconfig

# 验证动态链接库是否正确，打印中可以看到：libstdc++.so.6 -> libstdc++.so.6.0.25
ldconfig -v | grep c++

# 验证gcc版本
gcc --version

```



## 二、升级make 4.3

```bash
wget https://ftp.gnu.org/gnu/make/make-4.3.tar.gz
tar -zxvf make-4.3.tar.gz
cd make-4.3
./configure --prefix=/usr/local/make
make -j4
make install

cd /usr/bin
mv make make.bak
ln -sv /usr/local/make/bin/make /usr/bin/make

# 验证版本
make --version
```



## 三、升级glibc 2.28

```bash
# https://www.gnu.org/software/libc/
wget https://ftp.gnu.org/gnu/glibc/glibc-2.28.tar.gz
tar -zxvf glibc-2.28.tar.gz
cd glibc-2.28
mkdir build && cd build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make -j4
# 耗时比较久，如果有系统快照备份，可略过
make check
make install

# 如果升级后报错尝试如下方式，如果还不行，就要用之前的快照来恢复了
sln /usr/lib64/ld-2.17.so /usr/lib64/ld-linux-x86-64.so.2
sln /usr/lib64/libc-2.17.so /lib64/libc.so.6

# 如果使用locale命令时报错，则执行下面的操作
make localedata/install-locales
# 我自己环境中尝试时，仅执行上一步之后就不再乱码了
make install

# 验证 查看有“GLIBC_2.28”则表示成功
strings /lib64/libc.so.6 | grep GLIBC_
```

![image-20221031195826237](../media/image-20221031195826237.png)

## 四、安装python3

```bash
yum install epel-release
yum install -y python3
```



## 五、现网当前环境搭建（模拟）

Node.js版本v10.15.3，pm2版本3.5.0

```bash
nvm install v10.15.3
npm i -g git+https://github.com/Pana/nrm.git
npm i -g pm2@3.5.0
```



通过质效构建，然后部署到该环境中，使用pm2进行管理

![image-20220922154604868](../media/image-20220922154604868.png)



## 六、Node.js版本升级

升级之前可使用如下命令查看当前pm2使用的Node.js版本

```bash
# 查询当年pm2中使用的Node.js版本
pm2 monit
```



![image-20220922160836140](../media/image-20220922160836140.png)

按如下步骤安装新版本的Node.js以及全局依赖包

```bash
# 安装版本的Node.js
nvm install v16.17
# 设置新版本的Node.js为默认版本
nvm alias default v16.17
# 将旧版本Node.js全局安装的包在新版本中重新安装
nvm reinstall-packages v10.15.3
# 更新pm2的版本（非必须）
npm i -g pm2
```



修改pm2所使用的Node.js版本

```bash
# 保存pm2中的进程信息
pm2 save
# 更新pm2，使用新版本的Node.js
pm2 update
```



再次使用`pm2 monit`查看当前使用的Node.js版本

![image-20220922160930891](../media/image-20220922160930891.png)



## 七、现网组件梳理

### 1. Act（5078）——旧活动

​      cwd: '/data1/kuyin/act',

​      script: 'app.js',

​      env: {

​        PORT: '3001',

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/act/config',

​      },

http://172.31.114.108:3001/act/redsong/index.html#/index

http://172.31.114.108:3001/act/soundproject/index.html#/index

----

https://h5.kuyin123.com/act/soundproject/index.html



### 2. Grocery（287006）

​      cwd: '/data1/kuyin/grocery',

​      script: 'src/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/grocery/config',

​        PORT: '3004',

​      },

http://172.31.114.108:3004/grocery/share/index.html

---

https://h5.kuyin123.com/grocery/inventory/index.html?nh=115&cid=334f5ac04b6107fe&id=632d741ee4b0234c40f3e5c3&at=1&ta=0&fc=0&immer=1&ts=0&tc=6d4edf

https://h5.kuyin123.com/grocery/songrec/index.html?nh=115&cid=f0056b410b6b73a8&pushId=6350b767e4b0fbb4a52e01c2&at=1&ta=0&fc=0&immer=1&ts=0&tc=f7e6fc

https://www.ciqujia.cn/grocery/activity/index.html?aid=1537239650809552896#/



### 3. Hybrid(23529)——客户端H5页面

​      cwd: '/data1/kuyin/hybrid',

​      script: 'src/app.js',

​      env: {

​        NODE_ENV: 'product',

​        NODE_CONFIG_DIR: '/data1/kuyin/hybrid/config',

​        PORT: '3000',

​      },

http://172.31.114.108:3000/client/rights/renew/agreement.html

---

https://h5.kuyin123.com/client/page/ios/1cc04b158f5f73ae?supportsetring=1#/home

https://h5.kuyin123.com/client/hybrid/page/index.html

https://h5.kuyin123.com/client/rights/renew/agreement.html



### 4. RingService(656344)——试听

​      cwd: '/data1/kuyin/ringservice',

​      script: 'src/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/ringservice/config',

​      },

http://172.31.114.108:3007/listen/kyh5/kyext.js

---

https://chaos.kuyin123.com/listen/kyh5/kyext.js

https://chaos.kuyin123.com/listen/api/v1/q_a_url?cn=8692&ringno=1097634933490319360&midmb=c33e9155d39c6078cb1c7e0f1297fc21&miduc=&midtc=&urlmb=&urluc=&urltc=

---

待办事项：添加cdn访问kyext.js



### 5. RingTone（2799）——客户端振铃

​      cwd: '/data1/kuyin/ringtone',

​      script: 'src/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/ringtone/config',

​      },

http://172.31.114.108:3008/toneh5/order.js

---

https://pring.kuyin123.com/toneh5/page/aorder/c1c250db803b364f

https://pring.kuyin123.com/toneh5/order.js



### 6. TrackServer(4184)——轨迹服务

​      cwd: '/data1/kuyin/track-server',

​      script: 'dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/track-server/dist/config',

​      },

http://172.31.114.108:3009/track/test-touch.html

http://172.31.114.108:3009/track/ges/get?distance=516.0

---

https://migu.diyring.cc/track/ges/get?distance=516.0

待办事项：migu.diyring.cc是cdn的域名，会导致url不变时在一段时间内取到的轨迹数据相同



### 7. UnionOrder（260W）——统一订购组件

​      cwd: '/data1/kuyin/unionorder',

​      script: 'dist/app.js',

​      env: {

​        NODE_ENV: 'beijing',

​        NODE_CONFIG_DIR: '/data1/kuyin/unionorder/dist/config',

​      },

http://172.31.114.108:3006/union/page/miguv2



### 8. finder(515)——音乐Finder相关的H5分享页面

​      cwd: '/data1/kuyin/finder',

​      script: 'dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        PORT: '3014',

​        NODE_CONFIG_DIR: '/data1/kuyin/finder/dist/config',

​      },

http://172.31.114.108:3014/finder/demo-share/index.html

---

https://h5.kuyin123.com/finder/demo-share/index.html?shareid=6dc90abf07544b12a4809284072850b8&code=011b3lHa1IAy6E0jukFa11QT9W3b3lHK&state=123



### 9. h5-chinaunicom-aimusic-proxy（没有请求）——成都

npm start运行

http://172.31.114.108:4001/aimusic/public/index.html

---

https://miniprogram.diyring.cc/aimusic/public/index.html



### 10. h5act(3000)——未重构前的活动

​      cwd: '/data1/kuyin/h5act',

​      script: './dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5act/dist/config',

​      },

http://172.31.114.108:3011/actlm/0093b7a82da31e30/45275

http://172.31.114.108:3011/h5actcdn/guess-emoji/img/sound.40053d0b.png

---

https://game.diyring.cc/actlm/5490b6018b88a43e/321997?nh=128&at=1&ta=0&fc=0#/home



### 11. h5actssr(7500)——重构后的活动（基于Nuxt的SSR）

​      cwd: '/data1/kuyin/h5actssr',

​      script: './node_modules/nuxt/bin/nuxt.js',

​      args: 'start --env=prod',

​      env: {

​        PORT: '3022',

​      },

http://172.31.114.108:3022/act/culture2/a1bc17bd2b7b10a4/45275?type=vivo

http://172.31.114.108:3022/act/h5actres/d0bd291.modern.js

---

https://game.diyring.cc/act/culture2/a1bc17bd2b7b10a4/45275?type=vivo

https://game.diyring.cc/act/find/b3e405aaae106f5c/45275/home?type=vivo

https://game.diyring.cc/act/culture3/633c6db657f8eb50/45275/?type=vivo



### 12. h5bigscreen(4434)——音乐大屏

​      cwd: '/data1/kuyin/h5bigscreen',

​      script: './dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5bigscreen/dist/config',

​      },

http://172.31.114.108:3021/dashboard/iflymusic/index.html

---

https://muse.kuyinyun.com/dashboard/iflymusic/index.html



### 13. h5iflymusic(220)——讯飞音乐官网

​      script: './node_modules/nuxt/bin/nuxt.js',

​      args: 'start --env=prod',

​      env: {

​        PORT: '3023',

​        NODE_ENV: 'production.purchase',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5iflymusic/dist/config',

​      },

http://172.31.114.108:3023/

---

https://www.iflytekmusic.com/



### 14. h5itocoo(0)——讯飞图酷官网

​      script: './node_modules/nuxt/bin/nuxt.js',

​      args: 'start --env=prod',

​      env: {

​        NODE_ENV: 'production',

​        PORT: '3018',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5itocoo/dist/config',

​      },

http://172.31.114.108:3018/

---

https://www.itocoo.com/



### 15. h5lyrics(43)——词曲家作者端

​      script: './node_modules/nuxt/bin/nuxt.js',

​      args: 'start --env=prod',

​      env: {

​        PORT: '3019',

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5lyrics/dist/config',

​      },

http://172.31.114.108:3019/

---

https://www.ciqujia.cn/



### 16. h5music(17)——讯飞音乐旧版

​      cwd: '/data1/kuyin/h5music',

​      script: './dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5music/dist/config',

​      },

http://172.31.114.108:3013/h5music/pc/index.html

http://172.31.114.108:3013/h5music/app/index.html#/

---

https://h5.kuyin123.com/h5music/index



### 17. h5purchase(42)——词曲家采买端

​      script: './node_modules/nuxt/bin/nuxt.js',

​      args: 'start --env=prod',

​      env: {

​        PORT: '3020',

​        NODE_ENV: 'production.purchase',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5purchase/dist/config',

​      },

http://172.31.114.108:3020/

---

https://buy.ciqujia.cn/



### 18. h5ring（50w）——音乐彩铃

​      cwd: '/data1/kuyin/h5ring',

​      script: './dist/app.js',

​      env: {

​        NODE_ENV: 'product',

​        NODE_CONFIG_DIR: '/data1/kuyin/h5ring/dist/config',

​      },

http://172.31.114.108:3010

http://172.31.114.108:3010/iringh5res/huawei/js/suggest.a550066a.js

---

https://iring.diyring.cc/friend/7343c29048673252#/

https://hw.diyring.cc/#/



### 19. itocoo(0)——讯飞图酷旧版（已废弃）

​      cwd: '/data1/kuyin/itocoo',

​      script: './dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/itocoo/dist/config',

​      },

http://172.31.114.108:3017/itocoo/pc/index.html#/



### 20. musician(3)——讯飞音乐人

​      cwd: '/data1/kuyin/musician',

​      script: './dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/musician/dist/config',

​      },

http://172.31.114.108:3016/musician/album/index.html

---

https://h5.kuyin123.com/musician/album/index.html



### 21.vring（4w）——视频彩铃

​      cwd: '/data1/kuyin/vring',

​      script: './dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/vring/dist/config',

​      },

http://172.31.114.108:3012/

http://172.31.114.108:3012/sdk/v1/order.js

---

https://vring.kuyin123.com/#/

https://vring.kuyin123.com/friend/a1c742e28add2f9c#/

http://h5.kuyin123.com/vring/sdk/v1/order.js

###  22.cxmusic——成都创响（废弃）

​      cwd: '/data1/kuyin/cxmusic',

​      script: 'dist/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/cxmusic/dist/config',

​      },

http://172.31.114.108:3015/cxmusic/index#/



### 23.ministatic(0)——星座彩铃（成都）

http://172.31.114.108:4002/ministatic/constellation/login.html

http://172.31.114.108:4002/ministatic/star/index.html

---

https://miniprogram.diyring.cc/ministatic/star/index.html#/select



### 24. InGrocery(1)——AI作词内网页面

​	  cwd: '/data1/kuyin/ingrocery',

​      script: 'src/app.js',

​      env: {

​        NODE_ENV: 'production',

​        NODE_CONFIG_DIR: '/data1/kuyin/ingrocery/config',

​        PORT: '3024',

​      },




## 八、质效平台上构建脚本修改

**主要涉及运行环境依赖Node.js运行时的组件，比如Union-Order**，修改构建所使用的Node.js版本



