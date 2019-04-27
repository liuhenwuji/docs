docker run -d --hostname my-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3.7.12-management

#进入容器shell
docker exec -it 0d10476afd44  /bin/bash

#映射本地路径 $PWD是当前路径 
docker run -p 80:80 -d -v $PWD/html:/usr/share/nginx/html nginx

# docker registry
docker search whalesay
docker pull docker/whalesay
docker run docker/whalesay cowsay Docker hello!
//产生自己的docker镜像
docker tag docker/whalesay my/whalesay
docker images
docker login
docker push my/whalesay

uname -s //Linux
uname -m //x86_64


#redis
docker run -d -p 6379:6379 redis:4.0.8