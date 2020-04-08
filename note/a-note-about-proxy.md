# 常见开发环境代理设置

[^_^]: # (url:a-note-about-proxy)
[^_^]: # (tag:note,#tech)
[^_^]: # (excerpt:记开发环境中常见的代理需求该如何设置)

## Sys

### Windows

首先是主要的代理软件,其他应用的代理设置依赖于这一软件

1. SS
   大名鼎鼎,不赘述.
2. Fiddler
   用于解开 Windows UWP 应用的网络封闭
3. CFW
   Clash For Windows. 新兴的代理软件,另具备解开 UWP 网络隔离的功能.一般使用场景下可代替上述 SS 和 Fiddler.

## Shell

设置环境变量, http/s proxy, 将设置写入 `~/.bash_proxy` 文件中.
在`~/.bashrc`读取`~/.bash_proxy`文件.

在`.bashrc`文件加上下列内容,即可在终端使用`proxy`和`unproxy`命令来开启关闭代理

```bash
# GLOBAL PROXY FOR BASH
export PORXY_ADDR="http://your_proxy:1080"
# if ~/.bash_proxy not exist , then create it.
if [ ! -f ~/.bash_proxy ]; then
    touch ~/.bash_proxy
fi
# execute ~/.bash_proxy
. ~/.bash_proxy

function proxy() {
    # export all_proxy="$PORXY_ADDR"
    # export http_proxy="$PORXY_ADDR"
    # export https_proxy="$PORXY_ADDR"
    # export ALL_PROXY="$PORXY_ADDR"
    # export HTTP_PROXY="$PORXY_ADDR"
    # export HTTPS_PROXY="$PORXY_ADDR"
    echo -e "export {all_proxy,http_proxy,https_proxy,ALL_PROXY,HTTP_PROXY,HTTPS_PROXY}=\"$PORXY_ADDR\";" | tee ~/.bash_proxy >/dev/null
    # apply
    . ~/.bash_proxy
    # declare
    echo "current proxy status: using $PORXY_ADDR, proxying"
}

function unproxy() {
    # unset all_proxy http_proxy https_proxy ALL_PROXY HTTP_PROXY HTTPS_PROXY
    echo -e "unset all_proxy http_proxy https_proxy ALL_PROXY HTTP_PROXY HTTPS_PROXY" | tee ~/.bash_proxy >/dev/null
    # apply
    . ~/.bash_proxy
    # declare
    echo "current proxy status:  direct connect, not proxying"
}

```

使用`source`命令使修改生效后, 即可使用`proxy`或者`unproxy`方便地设置代理或者取消代理了.
但很多软件如 Docker,Git,APT 等皆需要特殊设置.

## PowerShell & CMD

1. 环境变量

   开启代理脚本

   ```bat
   @echo off
   SET proxy_addr=http://127.0.0.1:1080
   SETX>nul http_proxy "%proxy_addr%"
   SETX>nul https_proxy "%proxy_addr%"
   SETX>nul all_proxy "%proxy_addr%"

   SET http_proxy=%proxy_addr%
   SET https_proxy=%proxy_addr%
   SET all_proxy=%proxy_addr%
   echo From now on using %proxy_addr% as proxy server
   SET proxy_addr=
   @echo on
   ```

   关闭代理脚本

   ```bat
   @echo off
   SETX>nul http_proxy ""
   SETX>nul https_proxy ""
   SETX>nul all_proxy ""

   SET http_proxy=
   SET https_proxy=
   SET all_proxy=
   echo From now on directly connect to network
   @echo on
   ```

2. netsh 设置,部分情况下对 powershell 有作用

   检查代理设置

   ```powershell
   netsh winhttp show proxy
   ```

   设置代理

   ```powershell
   netsh winhttp set proxy "127.0.0.1:1080"
   ```

   还原代理设置

   ```powershell
   netsh winhttp reset proxy
   ```

## Git

1. 使用环境变量

   在\*nix 和在 windows 下设置方式不同,见上文.

2. 使用 git 配置

   ```bash
   proxy_addr="http://127.0.0.1:1080"
   git config --global http.proxy $proxy_addr
   # if you proxy is socks5
   git config --global http.proxy socks5://localhost:1080
   ```

对 ssh 进行配置
\*nix 环境下,请安装 nmap(需要里面的 ncat 命令)

```bash
Host github.com gitlab.com
    User git
    ProxyCommand ncat --proxy 127.0.0.1:1080 %h %p
```

Windows 环境下,Git 请安装至无空格的路径下(如 C:/Programs/Git)

```bash
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/git_id_rsa
    ProxyCommand C:/Programs/Git/mingw64/bin/connect.exe -H http://127.0.0.1:1080 %h %p
```

## Docker

Systemd 管理下则配置 docker.service,不赘述.
若是 WSL2 下的 Docker,是 SysV 脚本启动的 Docker. 则需要修改 `/etc/default/docker` 文件.

## WSL2

关于 WSL2 的代理设置,可参看:
[https://blog.nediiii.com/wsl2-note/](https://blog.nediiii.com/wsl2-note/)

## NodeJS

```bash
# set proxy
yarn config set proxy http://127.0.0.1:1080
yarn config set https-proxy http://127.0.0.1:1080
# unset proxy
yarn config delete proxy
yarn config delete https-proxy
# set mirrors
yarn config set registry http://registry.npm.taobao.org/
# unset mirrors
yarn config set registry https://registry.npmjs.org/

# set proxy
npm config set proxy http://127.0.0.1:1080
npm config set https-proxy http://127.0.0.1:1080
# unset proxy
npm config delete proxy
npm config delete https-proxy
# set mirrors
npm config set registry http://registry.npm.taobao.org/
# unset mirrors
npm config set registry https://registry.npmjs.org/
```

## Golang

https://github.com/golang/go/wiki/GoGetProxyConfig

主要是对 git 进行代理设置,上文已提到,不赘述
建议开启 go mod,并修改 GOPROXY.

```bash
go env -w GO111MODULE=on GOPROXY=https://goproxy.io
```
