# MongoDB安装

## 一、安装

```bash
brew tap mongodb/brew
brew update
brew install mongodb-community@7.0
```

## 二、相关文件位置

| Intel Processor                                              | Apple Silicon Processor      |                                 |
| :----------------------------------------------------------- | :--------------------------- | ------------------------------- |
| [configuration file](https://www.mongodb.com/docs/manual/reference/configuration-options/) | `/usr/local/etc/mongod.conf` | `/opt/homebrew/etc/mongod.conf` |
| [`log directory`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-systemLog.path) | `/usr/local/var/log/mongodb` | `/opt/homebrew/var/log/mongodb` |
| [`data directory`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.dbPath) | `/usr/local/var/mongodb`     | `/opt/homebrew/var/mongodb`     |

可以通过`brew --prefix`来查看安装的位置

## 三、启动

- 作为macOS的service启动和停止

  ```bash
  # 启动
  brew services start mongodb-community@7.0
  # 停止
  brew services stop mongodb-community@7.0
  ```

  

- 手动作为后台进程启动和关闭

  ```bash
  # Mac Intel芯片
  mongod --config /usr/local/etc/mongod.conf --fork
  # Mac M系列芯片
  mongod --config /opt/homebrew/etc/mongod.conf --fork
  # 关闭 通过mongosh连接mongod，然后执行shutdown命令
  ```

配置文件都位于mongod.conf文件中

- 安装之后查看服务是否存在

  ```bash
  # 作为macOS的service启动时
  brew services list
  
  # 作为后台进程手动启动时
  ps aux | grep -v grep | grep mongod
  ```

还可以查看日志文件mongo.log来确认服务是否正常启动

## 四、连接并使用mongodb

```bash
# 通过shell
mongosh

# 通过mongodb database tools提供的一系列命令可以进行部分操作，更多命令查看：https://www.mongodb.com/docs/database-tools/
mongotop
mongodump
mongoimport
```



