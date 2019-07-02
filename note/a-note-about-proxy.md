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

## Shell

设置环境变量, http/s proxy, 且以命令别名 alias 的形式来做.

在`.bashrc`文件加上下列内容

```bash
proxy_addr="http://127.0.0.1:1080"

# proxy
alias proxy='
echo -e "export http_proxy=$proxy_addr; \n\
export HTTP_PROXY=$proxy_addr; \n\
export https_proxy=$proxy_addr; \n\
export HTTPS_PROXY=$proxy_addr; \n\
export all_proxy=$proxy_addr; \n\
export ALL_PROXY=$proxy_addr; \n" \
    >> ~/.profile \
    && source ~/.profile'

# unproxy
alias unproxy="sed -i '/export/{/1080/d}' ~/.profile && \
source ~/.profile && \
unset http_proxy HTTP_PROXY https_proxy HTTPS_PROXY all_proxy ALL_PROXY && \
echo 'unset proxy' "
```

使用`source`命令使修改生效后, 即可使用`proxy`或者`unproxy`方便地为当前 session 设置代理或者取消代理了.

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
\*nix 环境下

```bash
Host github.com
    User                    git
    ProxyCommand            nc -x localhost:1080 %h %p
```

Windows 环境下

```bash
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/git_id_rsa
    ProxyCommand connect -H http://127.0.0.1:1080 %h %p
```

## Docker

在 daemon.json 配置,不赘述

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
