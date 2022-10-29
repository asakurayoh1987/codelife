# 生产环境Node.js版本升级

## 步骤

### 1. 安装新版本的node.js

```bash
nvm install v12.16.2 --reinstall-packages-from=v10.16.3 && nvm alias default v12.16.2
```

### 2. 保存pm2进程信息

新版本node安装后，通过`pm2 monit`可以看到当前运行的进程中node依旧是老版本，在更新pm2之前最好先保存一下进程信息

```bash
pm2 save
```

### 3. 升级pm2

```bash
pm2 update
```

升级之后再通过`pm2 monit`查看，可以发现应用的node版本已经更新为最新的版本了

