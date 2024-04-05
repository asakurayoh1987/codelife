# Node.js调试不完全指南

## 一、通过vscode调试

### 1. JavaScript Debug Terminal（便捷）

在需要调试在代码文件中添加好断点，打开`Terminal`，如下图中选择新建一个`JavaScript Debug Terminal`，然后在命令中直接通过`node xxx.js`执行脚本即可进入debug模式

![image-20240322110220034](https://oss.kuyinyun.com/11W2MYCO/rescloud1/4749fd993a904c419a0c10d55fdb4e8e.png)

![image-20240322110331386](https://oss.kuyinyun.com/11W2MYCO/rescloud1/24f80a4f21cb4bbbab430c8d10df55f6.png)

也可以通过快捷键打开命令面板（macOS：cmd + shift + p），直接打开`JavaScript Debug Terminal`

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/263a57c2ffd645df90a40327e66dd0d1.png" alt="image-20240322110455704" style="zoom:50%;" />

#### 注意事项

如果安装了`Bun`的vscode插件，会导致在`JavaScript Debug Terminal`在执行命令时进入不了debug模式，目前官方未给出解决方案，所以如非必要调试时先禁用此插件，或者选择后文介绍的其它调试方式

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/bc1a6334456442b89c2155e3d6ac170f.png" alt="image-20240322112854871" style="zoom:50%;" />

### 2. Auto Attach（便捷——三种模式可选）

通过命令面板打开`Auto Attach`的开关后，在vscode的集成终端中直接运行node.js脚本，会自动进入调试模式

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/a57a95f5656e48ea9178b61879943abe.png" alt="image-20240325152308845" style="zoom:50%;" />

开启后就可以在vscode的底部看到这样的状态展示，点击后可以切换模式：

![image-20240325152413112](https://oss.kuyinyun.com/11W2MYCO/rescloud1/de710a15e3f64a3db280d00c1813ce67.png)

三种模式的说明如下：

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/0ab3da5d12d848ec8b5c252473a639b7.png" alt="image-20240325152441564" />

这里我们选择`smart`模式，然后在vscode的集成终端中执行`PORT=4000 node app.js`，即自动进入调试模式

![image-20240325152944328](https://oss.kuyinyun.com/11W2MYCO/rescloud1/a97be4417f584c139286123134943597.png)

### 3. 通过 Run And Debug 面板（通用——控制粒度精细）

#### 3.1 Launch模式

通过以下步骤可以添加一个类型为“launch”的调试配置项，所谓Lauch模式就是以创建并启程进程的方式来开启调试

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/313700002ccf4f3b894a5a0da70e7784.png" alt="image-20240322113208962" style="zoom:50%;" />

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/d49c759e740d4e749e707a5ea12a86e5.png" alt="image-20240322113250346" style="zoom:50%;" />

![image-20240322113350596](https://oss.kuyinyun.com/11W2MYCO/rescloud1/d1b1fc942f354d4db00f7d968c7e6699.png)

添加之后，默认调试的就是当前打开的文件，比如截图中的`app.js`，此时可以通过`Run And Debug`面板中选择配置项（即这里的“Launch Program”，可以有多个配置项）并启动调试，也可以直接按“F5”来启动调试

![image-20240322113432105](https://oss.kuyinyun.com/11W2MYCO/rescloud1/144cf81f187a43839ee29a840a8141f5.png)

![image-20240322113534026](https://oss.kuyinyun.com/11W2MYCO/rescloud1/7281aded123d49d4854a393b8bd36cba.png)

#### 3.2 Attach模式

与Launch不同，该模式的调试对象是针对运行中的Node.js进程，当使用指定的命令参数以调试模式启动Node.js进行时，会暴露一个用于调试器客户端连接的端口，当调试器客户端连接后，代码运行至断点处则会进入调试界面

```bash
PORT=4000 node --inspect app.js
```

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/c0afb860fcde4fbda0fd4458fba6328a.png" alt="image-20240325105227002" />

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/74cb0e2f91c347eab59dae0d7dda7749.png" alt="image-20240325105448611" style="zoom:50%;" />

这种方式启动应用后，当调试器客户端未连接时，进程的运行不受影响，调试完成后断开连接进程依旧正常运行

![image-20240325110059012](https://oss.kuyinyun.com/11W2MYCO/rescloud1/0c95ba9456d94104b060c9bc4f34a0b5.png)

##### 关于`--inspect-brk`参数

当通过`PORT=4000 node --inspect-brk app.js  `来启动应用时，应用会在代码的首行进行断点，也就是说应用并没有真的启动起来

![image-20240325110431062](https://oss.kuyinyun.com/11W2MYCO/rescloud1/fd671b39a75d4ddab179ff02daefd1ac.png)

此时如果调试器的客户端连接时就会发现断点命中代码的第一行

#### 3.3 Attach by Process ID

适用于当未使用`--inspect`或`--inspect-brk`启动Node.js应用的场景，此时只知道应用的进程ID号（进程ID如何获取？）

![image-20240325111530642](https://oss.kuyinyun.com/11W2MYCO/rescloud1/6b2a5a0a930f424fb2c7912d68ffdf95.png)

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/2eef144e305a402e8e88213952b884f7.png" alt="image-20240325111618710" style="zoom:50%;" />

![image-20240325111707475](https://oss.kuyinyun.com/11W2MYCO/rescloud1/5913816a49d14cd08f3926ef1d11cf2a.png)

##### 如何获取PID

```bash
# 通过ps查看进程信息
ps -ef | grep 'app.js'

# 通过查看占用端口的进程信息
lsof -i:4000

# 通过在执行脚本时同时打印出来PID
PORT=4000 node app.js & echo $!

# 或者
PORT=4000 nohup node app.js & echo $!
```

需要注意一点的是，对于上面的后两种方式，Node.js进程会以后台的方式运行，通过`Ctrl + C`并不会结束进程，端口依旧被占用，此时如果再执行`PORT=4000 node app.js`则会报错，提示端口被占用，此时可以通过`fg`的方式将该进程切至前台，再通过`Ctrl + C`来结束进程 ，从而释放端口

#### 3.4 Attach to Remote

这里以测试环境服务器为例（服务器地址：`172.31.114.120`），在服务器上执行如下脚本，以调试模式启动Node.js应用

```bash
# 默认情况下是9229端口
node --inspect app.js
```

根据官方的建议，出于安全考虑，选择通过ssh tunnel的方式来进行远程调试，本地执行如下脚本创建一个ssh tunnel

```bash
# ssh -L 本地端口:本地地址:远程端口 远程服务器账号@远程服务器地址
ssh -L 9229:localhost:9229 root@172.31.114.120
```

ssh tunnel创建成功后在在vscode中进行如下配置

![image-20240325143954044](https://oss.kuyinyun.com/11W2MYCO/rescloud1/1024a66caff24932a9d9a1891867affc.png)

主要分为两个部分，图中标注的4个部分意义如下：

1. address，因为我们创建了ssh tunnel，所以本地的地址设置为`localhost`
2. localRoot，指本地代码的位置，不配置并不影响调试，只是代码只能以只读的方式进行调试
3. port，本地端口号
4. remoteRoot，远端的代码目录

#### 参考资料

1. [Node.js官网](https://nodejs.org/en/learn/getting-started/debugging#enabling-remote-debugging-scenarios)
2. [vscode官网](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_launch-configuration-attributes)

## 二、 通过Chrome

### 1. 本地调试

```bash
# 以调试模式启动应用
PORT=4000 node --inspect app.js
```

打开chrome，访问`chrome://inspect`，可以看到如下界面

![image-20240325161220178](https://oss.kuyinyun.com/11W2MYCO/rescloud1/b0e2be71c30d408f8bdcf3f8a6e51254.png)

点击`Configure...`，配置如下：

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/3ebbc88236cf44e4a180d852958ce833.png" alt="image-20240325161250795" style="zoom:50%;" />

此时就能看到待调试的项目，点击`inspect`

<img src="https://oss.kuyinyun.com/11W2MYCO/rescloud1/0416ae9e713b4a6c9243a461427511cc.png" alt="image-20240325161311351" style="zoom:50%;" />

设置好断点后发起请求即可命中断点

![image-20240325161456771](https://oss.kuyinyun.com/11W2MYCO/rescloud1/1edb20de67e24d6b8ef5c3e56f08c8bb.png)

### 2. 远程调试

前置操作与[3.4 Attach to Remote](#3.4 Attach to Remote)一致，但通过访问`chrome://inspect`看不到可调试的项目，我们假设在创建ssh tunnel时使用的是如下命令：

```bash
ssh -L 9221:localhost:9229 root@172.31.114.120
```

则本地在浏览器访问：`http://127.0.0.1:9221/json`，可以得到这样的结果

![image-20240325163700590](https://oss.kuyinyun.com/11W2MYCO/rescloud1/6c7dd1d2bdb348929ea6458b5ca0ec5a.png)

其中`devtoolsFrontendUrl`的值就是远程调试的页面地址，直接访问得到

![image-20240325163830080](https://oss.kuyinyun.com/11W2MYCO/rescloud1/ba5ac97b4cb148b0b06f32753274fc22.png)

### 参考资料

- [揭秘浏览器远程调试技术](https://fed.taobao.org/blog/taofed/do71ct/chrome-remote-debugging-technics/)

