

docker ps -a
docker inspect 容器名
docker diff 容器名  #显示容器启动后文件系统的改变情况
输出例子：
C /.wh..wh.plnk
A /.wh..wh.plnk/101.715484
D /bin
A /basket
A /basket/bash
A /basket/cat
A /basket/chacl
A /basket/chgrp
A /basket/chmod
...

Docker uses a union file system (UFS) for containers, 

docker logs 容器名 # 查看容器内产生的控制台输出
docker rm 容器名
docker rm -v $(docker ps -aq -f status=exited) # remove all your stopped containers

创建自己的镜像
1. from another image
2. from docker file








