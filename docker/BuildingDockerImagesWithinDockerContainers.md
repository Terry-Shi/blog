### Building Docker Images within Docker Containers

1. 假设各个角色名称为 **Host ---> 外层容器--->内层容器**

2. 外层容器必须加参数 `--privileged` 启动，例如
   `docker run --privileged --name some-docker -d docker`
   这里使用[官方 docker 镜像](https://hub.docker.com/_/docker/) 作为例子。也可以自己实现一个内含docker命令文件的镜像。

3. 进入外层容器，用下面命令启动内层 docker daemon 
   `nohup  dockerd  --storage-driver=vfs  --iptables=false -H tcp://0.0.0.0:2375 -H  unix:///var/run/docker.sock&`

   参数说明 
   1. `--storage-driver=vfs` this is the only driver which is guaranteed to work regardless of underlying filesystem 如果不加会出现以下错误信息
      `/var/lib/docker/aufs/mnt/e4e6356aac9c82bd7ba4b5fb5ff520de3f1ceef7cafc8f5ff92ad664a4bacc00-init:invalid argument`

   2. `--iptables=false` 外层容器没有iptable，实际也不是必要的

4. 内层 docker daemon 启动后可以执行各种 `docker build` 命令，`docker run hello-world` 也可以运行成功。

5. 补充：

   1. 网上有[一些建议](https://www.develves.net/blogs/asd/2016-05-27-alternative-to-docker-in-docker/)用文件映射的方式 `docker run -v /var/run/docker.sock:/var/run/docker.sock -ti docker`  

   2. 需要代理时加
      ```shell
      export https_proxy="http://IP:port"
      export http_proxy="http://IP:port"
      ```
6. 自定义 Dockerfile 包含 docker 命令文件的例子：
