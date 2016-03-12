### 基本概念
Swarm是一个官方提供的docker集群管理工具。
1. master slave结构：master节点上运行 swarm manager 负责协调调度容器的运行，slaver节点上运行着swarm agent。还需要一个服务发现（Service Discovery）功能，一般可以用Docker hub，etcd，ZooKeeper 或者简单的静态IP list文件来实现
2. 主要功能
 - filter 过滤，
     - 1. 利用容器基本信息:例如执行 `docker run -d -p 80:80 nginx`   swarm会考虑端口冲突因素。
     - 2. 利用宿主机的key-value tag指定docker 容器在哪台宿主机启动运行： 例如 `docker run -d -e constraint:dc==b postgres`
 - STRATEGIES 启动容器时挑选宿主机的策略有三种 spread（default），binpack，random。
3. 优点：提供了一个docker 容器集群的同一交互入口，把多个docker宿主机作为资源池来利用

### 安装预备工作：
 - 安全类： 打开相应的端口
 - Swarm manager HA 为一个集群配置多个swarm manager节点达到高可用的目的
 - Discovery service HA: Hosted (not for production use), Consul,etcd and Zookeeper
 - Multiple clouds
 - Operating system selection: For example, Docker container networks require Linux kernel 3.16 or higher.
 - Performance
    - Container networks
    - Scheduling strategies：spread，binpack，random (not for production use)

### 安装Build a Swarm cluster for production
TBC:  https://docs.docker.com/swarm/install-manual/

### issue list
1. 错误信息
$ SWARM_TOKEN=$(docker run swarm create)
docker: An error occurred trying to connect: Post https://192.168.99.100:2376/v1 .22/containers/create: Service Unavailable.
See 'C:\Program Files\Docker Toolbox\docker.exe run --help'.
https://github.com/docker/toolbox/issues/78

### 参考资料
- http://www.ibm.com/developerworks/cn/cloud/library/1511_zhangyq_dockerswarm/index.html
- 官方文档 https://docs.docker.com/swarm/