# 构建自己的kms-server镜像

[^_^]: # (url:build-your-own-kms-server-image)
[^_^]: # (tag:docker,docker-image,kms,#tech)
[^_^]: # (excerpt:如何借助开源技术构建自己的kms-server镜像用于测试激活Office和Windows等微软系软件)

> 现如今的生态，不可避免的需要使用Windows，那么Windows的系统激活以及Office的激活也就是一大问题  
> 因笔者对Windows的使用场景大多在虚拟机里做各种测试，使用频度不高，重装是家常便饭，遂kms激活便是我的一个很不错的解决方案  

## 开源的kms服务

在Github上搜索kms，按Star数排序，第一名的项目便是[Wind4/vlmcsd](https://github.com/Wind4/vlmcsd)。

![github-search-kms](/home/nediiii/CodeProjects/blog/img/post/build-your-own-kms-server-image/github-search-kms.png)

## 创建Dockerfile

1. 为了减小镜像大小，使用AlpineLinux作为基础镜像,这里使用的是笔者根据[官方alpine镜像](https://hub.docker.com/_/alpine)修改后的alpine镜像,详情可参阅[nediiii/alpine](https://hub.docker.com/r/nediiii/alpine)，当然使用官方镜像也可以。

2. 为了避免手动复制文件，也尽量避免增加镜像大小，这里采用多阶段构建的方式。这里有两点考虑

    1. 如果手动下载`vlmcsd`项目，再添加进镜像里，则每次`vlmcsd`项目更新之后必须手动下载更新后的`vlmcsd`项目，替换原有文件，再次构建镜像。这样必然很繁琐。
    2. 如果直接在构建中下载`vlmcsd`项目，需要安装`curl`软件，且需要清理`vlmcsd`项目的其他未使用的文件。如果采用多阶段构建，只需取得我们需要的文件即可，不会产生其他文件，也不需要进行清理工作。

Dockerfile文件如下：

```Dockerfile
# pre builder 构建阶段一 获取vlmcsd项目
FROM nediiii/alpine as builder

LABEL maintainer="varnediiii@gmail.com"

RUN apk add --no-cache curl

# download latest vlmcsd releases 下载最新的release
RUN wget $(curl -s https://api.github.com/repos/Wind4/vlmcsd/releases/latest | grep binaries.tar.gz |tail -n 1| cut -d '"' -f 4)

RUN tar -zxvf binaries.tar.gz

# main image 构建阶段二 从阶段一复制程序 并配置镜像启动参数
FROM nediiii/alpine

LABEL maintainer="varnediiii@gmail.com"

# copy the application 复制程序
COPY --from=builder /binaries/Linux/intel/static/vlmcsd-x64-musl-static .

EXPOSE 1688

# -L: listen ip:port , -e: log to stdout , -D: run in foreground
CMD ./vlmcsd-x64-musl-static -L 0.0.0.0:1688 -e -D

```

## 构建镜像

进入`Dockerfile`所在目录，运行下列命令

```bash
docker build . -t nediiii/kms-server
```

一切顺利的情况下，运行`docker images`命令即可看到上面构建的`nediiii/kms-server`镜像了。

## 测试镜像功能

1. 以守护进程的形式运行`nediiii/kms-server`，并映射1688端口，命令如下：

    ```bash
    docker run -d -p 1688:1688 nediiii/kms-server
    ```

    ![docker-images](/home/nediiii/CodeProjects/blog/img/post/build-your-own-kms-server-image/docker-images.png)

2. 虚拟机安装Windows，用以下命令测试激活

    密钥参阅：<https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj612867(v=ws.11)>

    - 卸载密钥：  
    `slmgr -upk`  

    - 输入密钥：  
    `slmgr -ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`  

    - 设置kms服务器地址：  
    `slmgr -skms ip`

    - 激活：  
    `slmgr -ato`

    - 检查状态：  
    `slmgr -dlv`

3. 安装Office（Volumes版）,用以下命令测试激活

    可参阅另一相关文章：[Office 2019 部署](/deploy-office/#toc2)  
    命令参阅：<https://docs.microsoft.com/en-us/deployoffice/vlactivation/tools-to-manage-volume-activation-of-office>  
    密钥参阅：<https://docs.microsoft.com/en-us/DeployOffice/vlactivation/gvlks>  

    - 检查状态：  
    `cscript ospp.vbs /dstatus`  
    - 卸载密钥：  
    `cscript ospp.vbs /unpkey:XXXXX`  
    - 输入密钥：  
    `cscript ospp.vbs /inpkey:XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`  
    - 设置kms服务器地址：  
    `cscript ospp.vbs /sethst:ip`  
    - 激活：  
    `cscript ospp.vbs /act`  

笔者对构建的镜像做过测试，激活Windows和Office（Volumes版）都没有问题。  
镜像地址：<https://hub.docker.com/r/nediiii/kms-server>
