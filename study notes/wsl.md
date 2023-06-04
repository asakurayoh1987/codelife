# WSL2开发环境搭建

## wsl2安装

参照[这里](https://docs.microsoft.com/en-au/windows/wsl/install-win10)

## 手动安装ubuntu



## 配置安装源镜像

```bash
# 备份源文件
cp /etc/apt/sources.list sources.list_backup
# 清空源文件下内容，写入下面的镜像源信息
sudo vim /etc/apt/sources.list
```

deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

```bash
# 保存文件后使用下述命令来更新安装源
 sudo apt-get update && sudo apt upgrade
```

## 设置临时的proxy

```bash
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*');
export https_proxy=http://${hostip}:7890 http_proxy=http://${hostip}:7890 all_proxy=socks5://${hostip}:7891
```

## 安装基础依赖

```bash
sudo apt-get install build-essential curl file git -y
```

## 安装zsh

[官方安装说明](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)

```bash
sudo apt install zsh -y
sudo apt-get install powerline fonts-powerline -y

chsh -s /bin/zsh
```

* 安装Oh My Zsh

参见[这里](https://github.com/ohmyzsh/ohmyzsh)

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

* 安装zsh-powerline-theme

参见[这里](https://github.com/jeremyFreeAgent/oh-my-zsh-powerline-theme)

* 安装插件[参照](https://gist.github.com/n1snt/454b879b8f0b7995740ae04c5fb5b7df)
	* [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
	* [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
	* [autojump](https://github.com/wting/autojump)
	* [git open](https://github.com/paulirish/git-open)
* 配置插件

```bash
# 编辑zsh的配置文件，配置如下信息
vim ~/.zshrc
```

```bash
# 主题相关的配置
ZSH_THEME="powerline"
POWERLINE_RIGHT_A="mixed"
POWERLINE_HIDE_USER_NAME="true"
POWERLINE_HIDE_HOST_NAME="true"
POWERLINE_PATH="short"
POWERLINE_RIGHT_A="mixed"
HIST_STAMPS="yyyy-mm-dd"

# 插件
plugins=(git zsh-syntax-highlighting zsh-autosuggestions git-open)

# 网络代理

export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*');
alias setproxy='export https_proxy=http://${hostip}:7890 http_proxy=http://${hostip}:7890 all_proxy=socks5://${hostip}:7891'
alias unsetproxy='unset https_proxy http_proxy all_proxy'
```

## 安装node

* 安装nvm

参见[这里](https://github.com/nvm-sh/nvm)

* 安装node

```bash
nvm install v10.15.3
```

* 设置镜像源

```bash
npm config set registry https://registry.npm.taobao.org
```

* 安装 nrm

```bash
npm i -g nrm
nrm use taobao
```

* 设置讯飞npm私服

```bash
 nrm add iflytek http://maven.iflytek.com:18081/respository/npm-public
```

## git配置

```bash
git config --global user.name "ycyu"
git config --global user.email ycyu@iflytek.com
git config --global core.editor vim
git config --global core.autocrlf false
```

## 以import方式导入时用户为root

* 修改方法：

Get-ItemProperty Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss\*\ DistributionName | Where-Object -Property DistributionName -eq <UbuntuCustom> | Set-ItemProperty -Name DefaultUid -Value 1000

[参考](https://github.com/microsoft/WSL/issues/3974#issuecomment-504225491)

## 局域网访问你本机WSL启动的应用

* https://docs.microsoft.com/en-us/windows/wsl/compare-versions#accessing-a-wsl-2-distribution-from-your-local-area-network-lan
