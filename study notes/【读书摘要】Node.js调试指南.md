# Node.js调试指南

## 第1章 CPU

> Node.js 是单线程异步 I/O 模型，适合 I/O 密集型的场景，这也意味着 Node.js 并不适合 CPU 密集型计算的场景，因为 Node.js 不是收到一个请求就启动一个线程，假如有一个计算任务长时间地占用 CPU，整个应用就会“卡住”，无法处理其他请求

### 1.1 理解perf与火焰图（FlameGraph）

常见的操作系统的性能分析工具：

- Linux：perf、eBPF、SystemTap和ktap
- Solaris：illumos、FreeBSD和DTrace
- Mac OS X：DTrace和Instruments
- Windows:Xperf.exe.

#### 1.1.1 perf

- 环境搭建

  - 当前的系统是MacOS M2

  - 通过multipass安装了Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic aarch64)

  - 进入Ubuntu虚拟机：`multipass shell`（这里我设置了默认虚拟机，因为可以创建多个）

  - 更新安装源：

    - deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy main restricted universe multiverse
      deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy-updates main restricted universe multiverse
      deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse
      deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy-backports main restricted universe multiverse

      deb [arch=arm64] https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
      deb [arch=arm64] https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
      deb [arch=arm64] https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
      deb [arch=arm64] https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse

    - `sudo apt update && sudo apt upgrade`

    - 关于当前Ubuntu的版本信息可以通过命令查看：`lsb_release -a`，上面源中的`jammy`就是取自这条命令执行结果中的`Codename`

  - 安装perf：

    - `sudo apt-get install linux-tools-generic`
    - 安装成功之后可以通过`perf --version`来验证是否安装成功

  - 下载生成火焰图的工具

    - `git clone https://github.com/brendangregg/FlameGraph.git ~/FlameGraph`

- 测试代码，这里基于koa搭建一个服务，源码如下，通过pnpm安装下koa即可

  ```javascript
  const crypto = require('node:crypto');
  const Koa = require('koa');
  
  const app = new Koa();
  const users = {};
  
  app.use(ctx => {
    const path = ctx.path;
    if (path === '/newUser') {
      const name = ctx.query.username || 'test';
      const password = ctx.query.password || 'test';
  
      const salt = crypto.randomBytes(128).toString('base64');
      const hash = crypto
        .pbkdf2Sync(password, salt, 10000, 64, 'sha512')
        .toString('hex');
  
      users[name] = { salt, hash };
      ctx.status = 204;
    }
  
    if (path === '/auth') {
      const name = ctx.query.username || 'test';
      const password = ctx.query.password || 'test';
  
      if (!users[name]) {
        ctx.status = 400;
        return;
      }
  
      const hash = crypto
        .pbkdf2Sync(password, users[name].salt, 10000, 64, 'sha512')
        .toString('hex');
  
      if (users[name].hash === hash) {
        ctx.status = 204;
      } else {
        ctx.status = 403;
      }
    }
  });
  
  app.listen(3000, () => console.log('server running on port: 3000'));
  ```

- 生成数据

  - ```bash
    # 生成性能数据
    node --perf-basic-prof app.js &
    ```

  - 性能数据的位置：`cat /tmp/perf-50362.map`，其中的50362即为PID

- 压测

  - 工具：可以通过`sudo apt install apache2-utils`安装apache的ab，也可以使用`pnpm add -g autocannon`来安装autocannon

  - ```bash
    # 1. 创建一个用户，PS：记住，这里不要打开网络代理
    curl 'http://localhost:3000/newUser?username=admin&password=123456'
    
    # 2. 打开一个新的终端窗口，准备好性能采样脚本，这里的50362即为当前node.js服务的PID
    # -F表示采样频率，-p表示采样的PID，-g表示启用call-graph记录，sleep 30表示记录30s
    sudo perf record -F99 -p50362 -g -- sleep 30;
    
    # 3. 在原终端窗口中执行如下命令，开启压测，并立刻回到上一步的终端窗口中执行采样脚本
    ab -k -c 10 -n 2000 "http://localhost:3000/auth?username=admin&password=123456"
    
    # 4. 修改map文件为root权限，不然会报错
    sudo chown root /tmp/perf-50362.map
    
    # 5. 写入trace信息，perf record会将记录的信息保存到当前目录的perf.data文件中，使用perf script来将perf.data中的信息写入perf.stacks中
    sudo perf script > perf.stacks
    
    # 6. 生成火焰图，使用之前clone的FlameGraph
    ~/FlameGraph/stackcollapse-perf.pl --kernel < ~/test/perf.stacks | ~/FlameGraph/flamegraph.pl --color=js --hash > ~/test/flamegraph.svg;
    ```

#### 1.1.2 火焰图

