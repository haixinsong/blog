# SSH 使用笔记

[^_^]: # (url:ssh-note)
[^_^]: # (tag:note,#tech)
[^_^]: # (excerpt:SSH 配置文件优化工作流)

## SSH 配置文件位置

Linux 下:

    # 用户配置
    ~/.ssh/config
    # 系统配置
    /etc/ssh/ssh_config

Windows10(Ver.2004)下:

    # 用户配置
    %USERPROFILE%\.ssh\config
    # 系统配置
    C:\ProgramData\ssh\ssh_config

## 配置应用优先级

命令配置 `>` 用户配置 `>` 系统配置

即 :

执行命令时指定的配置 `优于` 用户配置文件中的配置 `优于` 系统配置文件的配置.

这与绝大多数的软件配置的优先级的逻辑一致.

## 配置文件格式

例如:

    # ~/.ssh/config
    Host example                       # 关键词
        HostName a.example.com         # 主机地址(ip/域名)
        User root                      # 用户名
        # IdentityFile ~/.ssh/id_ecdsa # 证书
        # Port 22                      # 指定端口

此后可以使用`ssh example`来连接 a.example.com, ssh example 等价于 ssh root@a.example.com

为 git 配置

    Host github.com gitlab.com         # 你也可以加上其他支持git的网站,空格分隔
        HostName %h                    # Tokens,指代匹配到的Host值, 详见下面的Tokens
        User git                       # git,此处不可修改,需要使用git用户
        IdentityFile ~/.ssh/git_id_rsa # 注意需要提前将你的rsa证书配置到github等网站上

以上配置为 github 和 gitlab 两个网站设置了自动使用 git_id_rsa 证书进行验证.

此后可以使用 `git clone git@github.com:your_name/your_repository.git` 来 clone 你的仓库. 不需要再输入密码.

### Tokens

    Arguments to some keywords can make use of tokens, which are expanded
         at runtime:

               %%    A literal ‘%’.
               %C    Hash of %l%h%p%r.
               %d    Local user's home directory.
               %h    The remote hostname.
               %i    The local user ID.
               %L    The local hostname.
               %l    The local hostname, including the domain name.
               %n    The original remote hostname, as given on the command line.
               %p    The remote port.
               %r    The remote username.
               %T    The local tun(4) or tap(4) network interface assigned if
                     tunnel forwarding was requested, or "NONE" otherwise.
               %u    The local username.

         Match exec accepts the tokens %%, %h, %i, %L, %l, %n, %p, %r, and %u.

         CertificateFile accepts the tokens %%, %d, %h, %i, %l, %r, and %u.

         ControlPath accepts the tokens %%, %C, %h, %i, %L, %l, %n, %p, %r, and
         %u.

         Hostname accepts the tokens %% and %h.

         IdentityAgent and IdentityFile accept the tokens %%, %d, %h, %i, %l,
         %r, and %u.

         LocalCommand accepts the tokens %%, %C, %d, %h, %i, %l, %n, %p, %r, %T,
         and %u.

         ProxyCommand accepts the tokens %%, %h, %n, %p, and %r.

         RemoteCommand accepts the tokens %%, %C, %d, %h, %i, %l, %n, %p, %r,
         and %u.

## 配置读取逻辑

一般对 SSH 的配置都写在用户配置中,如 linux 下就是 `~/.ssh/config`

用以下配置文件来作说明:

    Host example                       # 关键词
        HostName a.example.com         # 主机地址(ip/域名)
        User root                      # 用户名
        # IdentityFile ~/.ssh/id_ecdsa # 证书
        Port 22                        # 指定端口

    Host *                             # 快捷名称
        HostName a.example.com         # 主机地址(ip/域名)
        User root                      # 用户名
        IdentityFile ~/.ssh/id_ecdsa   # 证书
        Port 23                        # 指定端口

SSH 会读取配置文件,并匹配所有能匹配的项, 组合其中的所有配置项. 对于重复的配置项, 使用最先读取到的配置值.

如 `ssh example` 会匹配 `example` 也会匹配 `*` , 那么组合起来的配置是:

    HostName a.example.com         # 重复项, 使用最先读取的配置, 也就是example块的配置
        User root                      # 重复项, 使用最先读取的配置, 也就是example块的配置
        IdentityFile ~/.ssh/id_ecdsa   # 组合配置, 使用*块的配置
        Port 22                        # 重复项, 使用最先读取的配置, 也就是example块的配置

## SSH 使用代理

Windows 下, 安装 Git 到某个没有路径里不包含空格的位置(如`C:\Programs\Git`)去. 需要里面的 `Git\mingw64\bin\connect.exe`文件

Linux 下, 安装 `nmap` ,需要里面的 `ncat`命令.

配置 SSH 的 config 文件, 对需要代理的主机设置 `ProxyCommand` 配置, 如:

    Host example
        HostName example.com
        User root
        Port 22
        ProxyCommand ncat --proxy 127.0.0.1:1080 %h %p                                                       # 注意把127.0.0.1:1080改成你的代理地址, 不可加http://前缀
        # ProxyCommand C:/Programs/Git/mingw64/bin/connect.exe -H http://127.0.0.1:1080 %h %p                # windows下的配置格式,注意connect.exe的路径要正确

也可为将使用代理设为默认项, 仅对个别主机特殊, 如下:

    Host example
        HostName example.com
        User root
        Port 22
        ProxyCommand none # 设置不使用ProxyCommand

    Host hk
        HostName 47.244.176.45
        User root
        Port 22
        IdentityFile ~/.ssh/INT_ALI_HK.pem
        # 没有设置ProxyCommand, 则会组合*块的设置, 从而使用ProxyCommand

    # 注意需要将这个块放置到配置文件的最下方, 确保其他主机的ProxyComand先被读取到(避免某些主机的特殊配置被覆盖).
    Host *
        ProxyCommand ncat --proxy 127.0.0.1:1080 %h %p

### 为 Git 设置 SSH 的代理,可以使 git clone git 协议格式的地址时使用代理而加快速度. 因为国内直连 github/gitlab 都很慢.

如果是 Git clone https 协议的地址 , 那么需要手动为 git 设置 http 代理才可以使用代理.可参见: /a-note-about-proxy/

## 开启 ssh-agent ( Windows 上把`OpenSSH Authentication Agent`服务打开就 OK,建议设置成自动启动)

    eval `ssh-agent`

## 将密钥加入 agent 管理

    ssh-add ~/.ssh/id_rsa

## 在.ssh/config 里面把想要被 agent 纳入管理的站点加上

    ForwardAgent yes

## 查看密钥

    ssh-add -l
