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
