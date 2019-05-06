# 获取镜像
docker pull ubuntu:16.04

# 运行
docker run -it -rm ubuntu:16.04 bash

# 列出镜像
docker images
docker image ls

# 镜像体积
docker system df

# 构建镜像
Dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
在所在Dockerfile目录运行
docker build -t nginx:v3 .

docker run --name webserver -d -p 80:80 nginx
docker container ls -a
docker container stop
docker container rm

docker build https://github.com/liuhenwuji/docs.git#note/docker

docker build -t ubuntu:myip -f D:\docs\note\docker\myip .
docker build -t nginx:myweb -f D:\docs\note\docker\myweb .

docker run --name myweb -d -p 81:80 nginx:myweb

# 进入容器 bash
docker exec -it 69d1 bash

# 导出导入容器
windows:
docker export b228a3abaf8a > d:\ubuntu.tar
docker import d:\ubuntu.tar test/ubuntu:v1.0

linux:
docker export 7691a814370e > ubuntu.tar
cat ubuntu.tar | docker import - test/ubuntu:v1.0
docker import http://example.com/exampleimage.tgz example/imagerepo


