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
