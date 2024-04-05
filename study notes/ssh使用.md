# ssh使用

## 1. 在远程服务器上建立反向代理

1. 在本地机器上，使用 SSH 命令启动一个反向隧道，将远程服务器的流量转发到本地的 SOCKS5 代理

   ```sh
   ssh -R 7890:localhost:7890 user@remote_host
   ```

2. 在远程服务器上，通过设置环境变量或使用代理工具将网络请求通过 SOCKS5 代理转发

   ```bash
   export http_proxy="socks5://localhost:7890"
   export https_proxy="socks5://localhost:7890"
   ```

## 2. 让本地网络请求通过远程服务器转化

1. 在本地机器的 1090 端口启动一个 SOCKS 代理，将所有通过该端口的流量通过远程服务器转发。

   ```bash
   ssh -D 1090 -g root@172.31.114.120
   ```

2. 本地可通过环境变量设置代理（如上一例）或在浏览器中可以通过SwitchyOmega插件将对应的网络请求转发到本地的1090端口即可

