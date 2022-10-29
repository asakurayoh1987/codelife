# PM2功能列表整理

## 一、功能或特性

1. `--cron <cron_pattern>`，指定强制重启的cron表达式，可以运用于按天分割日志的场景
2. 通过`--`分隔来传递参数，比如`pm2 start api.js -- arg1 arg2`
3. `--update-env`选项来更新环境变量，`NODE_ENV=production pm2 restart app --update-env`
4. 



### 二、常用命令

1. `pm2 [list|ls|status]`查看当前应用状态
2. `pm2 [restart|reload|stop|delete] app_name`重启、重载、停止、删除应用
3. `pm2 monit`查看实时的dashboard
4. `pm2 startup`用于服务器重启时自动启动pm2并拉起应用
5. `pm2 save`用于在进行`pm2 update`前保存进程信息
6. `pm2 describe app_id`查看指定进程id的所有信息（等同于`pm2 show app_id`）
7. `pm2 reset [app|all]`重置应用的重启次数
8. `pm2 [start|stop|restart|reload|delete] ecosystem.config.js` [`--only appname|"app1,app2"`]



## 三、升级pm2

```bash
pm2 save
npm i -g pm2@latest
pm2 update
```

## 四、重启策略

1. cron方式（使用5位，分时日月周）

   ```bash
   pm2 start app.js --cron-restart="0 0 * * *"
   
   # 关闭cron restart功能
   pm2 restart app_name --cron-restart 0
   ```

   

2. watch方式，文件修改则重启

   ```bash
   pm2 start app.js --watch
   
   # 关闭watch必须使用如下方式
   pm2 stop app_name --watch
   ```

   

   使用配置文件的方式可以指定`watch_delay`以及`ignore_watch`

3. 内存增涨到最大值（每30秒检测一次），单位可以是：K\M\G

   ```bash
   # 同配置文件中的max_memory_restart: '300M'
   pm2 start app.js --max-memory-restart 300M
   ```

   

4. 不自动重启，比如执行shell脚本等只需要执行一次的场景

   ```bash
   pm2 start 'ls -l' --no-autostart --attach
   ```

   

5. 指定exit code来跳过重启

   ```bash
   # 当exit code为0时不进行重启，此种属于正常关闭
   pm2 start app.js --stop-exit-codes 0
   ```

   

6. 指数级延迟重启

   ```bash
   pm2 start app.js --exp-backoff-restart-delay=100
   ```

   第一次重启延迟100ms，第二次150ms，第三次225ms，第n次为100*1.5^(n-1)ms，直到最大的15000ms，当近30s没有异常重启，则该值被重置

## 五、日志

日志模块：pm2-logrotate，自动分割日志，并可指定使用的硬盘空间上限

```bash
pm2 install pm2-logrotate
```

## 六、关于配置文件

### 6.1 interpreter

**用于指定解释器的绝对路径**，默认为node，如果升级node版本，可以通过指定`interpreter`来处理



