# ubuntu18.04环境下docker设置指南
[^_^]: # (url:ubuntu-docker-setup)
[^_^]: # (tag:#tech,note,setup,docker)
[^_^]: # (excerpt:在ubuntu18.04下安装docker和docker-compose)

## 安装docker

依照官方文档的介绍，目前有两种主流的方式安装docker,其中脚本安装方式应该是对手动命令安装的一种封装，总之两种安装方式结果都一致

### 使用`docker-install`脚本进行安装

> 详情参阅[https://github.com/docker/docker-install](https://github.com/docker/docker-install)

这个安装脚本的目的是为了在受支持的linux发行版（主流发行版如ubuntu,centos等）中快速安装最新docker-ce。建议不要在生产系统中依赖此脚本进行部署工作。  
*经笔者测试，此脚本在`AWS`的`Amazon Linux`中并不能正常工作。*

命令如下：

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

### 依照官方文档进行安装

> 详情参阅[https://docs.docker.com/install/](https://docs.docker.com/install/)

命令如下：

```bash
# 卸载旧版本的docker, 如果是新系统，可跳过此步骤
sudo apt-get remove docker docker-engine docker.io containerd runc

# 更新软件源
sudo apt-get update

# 允许apt使用https协议
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# 添加Docker的官方GPG秘钥
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 验证秘钥
sudo apt-key fingerprint 0EBFCD88

# 设置软件仓库， 此命令设置为软件仓库为稳定版 stable
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"

# 更新软件源
sudo apt-get update

# 正式安装docker
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

新建`docker`用户组，并将当前用户添加到`docker`组中，这样可以避免当前用户使用docker命令时必须加sudo才能

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
# 随后重启或者注销，以使用户组设置生效
```

设置docker开机自启

```bash
sudo systemctl enable docker
```

disable docker start on boot

```bash
sudo systemctl disable docker
```

## 设置docker国内镜像

从以下镜像中选择一个镜像，笔者使用阿里云镜像

- [Docker 中国官方镜像加速](https://www.docker-cn.com/registry-mirror)
- [阿里云加速(需注册账号)](https://cr.console.aliyun.com/cn-hangzhou/mirrors)
- [DaoCloud 加速器](https://www.daocloud.io/mirror)
- [七牛云镜像加速](https://kirk-enterprise.github.io/hub-docs/#/user-guide/mirror)

将下列命令的`https://replace.with.your.mirror.com`替换成你选择的镜像地址，按顺序执行下列命令

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://replace.with.your.mirror.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 安装docker-compose

依照官方文档的介绍，目前主流的方式是直接下载对应平台的compose二进制文件
命令如下：

```bash
curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 测试docker与docker-compose

运行下列命令：验证docker与docker-compose 是否安装成功

```bash
docker --version
docker-compose --version
```

如正确返回版本号，docker安装到此结束。
