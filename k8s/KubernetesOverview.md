# Kubernetes Overview

## 主要组件
Kubernets属于主从的分布式集群架构，包含Master和Node（slave）节点：

### Master
Master 作为控制节点，调度管理整个系统，包含以下组件：

#### API Server 
  作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。它维护的REST对象将持久化到etcd(一个分布式强一致性的key/value存储)。
#### Scheduler 
  负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上。

#### Controller Manager
  负责执行各种控制器，目前有两类：
- Endpoint Controller：定期关联service和pod(关联信息由endpoint对象维护)，保证service到pod的映射总是最新的。
- Replication Controller：定期关联replicationController和pod，保证replicationController定义的副本数量与实际运行pod的数量总是一致的。 

另外 Controller-manager 定期收集 node 健康数据。

### Node
Node 是运行业务容器的节点，包含以下组件：

#### Kubelet
  负责管控docker容器，如启动/停止、监控运行状态等。它会定期从etcd获取分配到本机的pod，并根据pod信息启动或停止相应的容器。同时，它也会接收apiserver的HTTP请求，汇报pod的运行状态。Kubelet 进程启动时会连接 apiserver 自注册，加入到k8s集群。同时也负责Volume（CVI）和网络（CNI）的管理；

#### Kube Proxy
  kube-proxy 运行在每个节点上，监听 API Server 中服务对象的变化，通过管理 iptables 来实现网络的转发。
  kube-proxy 有两种实现 service 的方案：userspace 和 iptables
- userspace 是在用户空间监听一个端口，所有的 service 都转发到这个端口，然后 kube-proxy 在内部应用层对其进行转发。因为是在用户空间进行转发，所以效率也不高
- iptables 完全实现 iptables 来实现 service，是目前默认的方式，也是推荐的方式，效率很高（只有内核中 netfilter 一些损耗）。

### 其他
etcd：分布式key-value store 存放集群状态信息和配置信息
kubectl  客户端命令行工具
kube-dns负责为整个集群提供DNS服务
Ingress Controller为服务提供外网入口
Heapster提供资源监控
Dashboard提供GUI
Federation提供跨可用区的集群

## Kubernetes的核心技术概念和API对象:
API对象是Kubernetes集群中的管理操作单元。Kubernetes集群系统每支持一项新功能，引入一项新技术，一定会新引入对应的API对象，支持对该功能的管理操作。例如副本集Replica Set对应的API对象是RS。

### Pod：
最小部署，管理单位。
生命周期由 repliction manager 管理。
每个pod有一个根容器pause，pod中的其他业务容器使用net=container的方式，造成的结果是一个pod内的容器共用一个相同的 IP，使用pod名作为容器间通信的主机名，共用pause容器挂载的Volume。简化了同一pod内容器之间的网络通信和文件共享。 Overlayer Network（例如flannel）保证不同宿主机上的pod容器也可以通过pod IP直接通信。
不建议在k8s的一个pod中运行相同应用的多个实例。
Pod是Kubernetes集群中所有业务类型的基础，可以看作运行在K8集群中的小机器人，不同类型的业务就需要不同类型的小机器人去执行。目前Kubernetes中的业务主要可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）；分别对应的小机器人控制器为Deployment、Job、DaemonSet和StatefulSet

### 复制控制器（Replication Controller，RC）
RC是Kubernetes集群中最早的保证Pod高可用的API对象。通过监控运行中的Pod来保证集群中运行指定数目的Pod副本。指定的数目可以是多个也可以是1个；少于指定数目，RC就会启动运行新的Pod副本；多于指定数目，RC就会杀死多余的Pod副本。即使在指定数目为1的情况下，通过RC运行Pod也比直接运行Pod更明智，因为RC也可以发挥它高可用的能力，保证永远有1个Pod在运行。RC是Kubernetes较早期的技术概念，只适用于长期伺服型的业务类型，比如控制小机器人提供高可用的Web服务。新版本中被 Deployment 取代。

### 副本集（Replica Set，RS）
RS是新一代RC，提供同样的高可用能力，区别主要在于RS后来居上，能支持更多种类的匹配模式。副本集对象一般不单独使用，而是作为Deployment的理想状态参数使用。
``` yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```


### 部署(Deployment)

部署表示用户对Kubernetes集群的一次更新操作。部署是一个比RS应用模式更广的API对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。滚动升级一个服务，实际是创建一个新的RS，然后逐渐将新RS中副本数增加到理想状态，将旧RS中的副本数减小到0的复合操作；这样一个复合操作用一个RS是不太好描述的，所以用一个更通用的Deployment来描述。以Kubernetes的发展方向，未来对所有长期伺服型的的业务的管理，都会通过Deployment来管理。

### Label：
key／value形式附加到各个资源对象上，例如：pod，service，RC，Node等。便于以后分类筛选。 

### Service:
####  含义：
一个应用程序可能有多个 pod 组成，而 pod 的生命周期是短时性的，何时被创建以及何时被销毁是无法预测的。当给每个 pod 分配一个IP地址时，谁也无法保证这些IP地址是否为持久可访问的，因为这些IP地址对应的 pod 并非持久存在的。这样需要一个固定的访问入口，这就是 service 存在的原因。每一个 node 上都运行着kube-proxy，它会把指向 service 的请求转发到真正需要处理的pod上去。起到了流量转发和负载均衡的作用。

#### 实现方式 (ClusterIP, NodePort, LoadBalancer  or ExternalName）

##### ClusterIP:  
仅限k8s集群内通信 use a cluster-internal IP only，range is defined by --service-cluster-ip-range
这是Type的默认值。主要作用是方便集群内部pod到pod之间的访问（克服被访问的pod可能出问题，被部署到其他地方，IP发生变化的情况）。只有本k8s集群的pod内的app可以通过clousterIP + service port访问这个service。`spec.clusterIp:spec.ports[*].port`  clusterIP方式下处理过程：
    1. 直接访问service的clusterIP, 由于clusterIP是虚IP，会被 iptables 的 nat 策略转发到当前 node 的kube-proxy监听的端口上。
    2. 然后kube-proxy会通过内置的loadbalancer转发请求给后面的endpoint。
     两种proxy-mode: userspace and iptables 后面有解释
```Yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP # support TCP and UDP. The default is TCP.
      port: 80  # service的端口
      targetPort: 9376 # 目的地容器的端口，也可以是name of a port in the backend Pods
      nodePort：xxx # Service对外提供服务的端口
  sessionAffinity: ClientIP # the default is "None"  
#默认情况下，服务会随机转发到可用的后端。如果希望保持会话（同一个 client 永远都转发到相同的 pod），可以设置为 ClientIP。
```

##### NodePort:  
expose the service on a port on each node of the cluster ( same port on each node). You’ll be able to contact the service on any :NodePort address. 只要能访问任意一个NodeIP:NodePort 即可访问这个service，也就是说请求发起方不需要在k8s集群中。只要能访问到宿主机即可。nodePort的原理在于在每个node上开了一个端口，将向该端口的流量导入到kube-proxy，然后由kube-proxy进一步导给对应的pod。Note that this Service will be visible as both `<NodeIP>:spec.ports[*].nodePort` and `spec.clusterIp:spec.ports[*].port`.  但是不可在pod内用 clusterIp:nodePort 访问！需要注意的是，如果我们没有指定 nodePort 的值，kubernetes 会自动给我们分配一个，比如 31647（默认的取值范围是 30000-32767）。当然我们通过配置中nodePort: 31000 指定使用 31000 端口。
  这种方式的缺点是service很多时端口容易冲突。

* service 原理解析
  http://cizixs.com/2017/03/30/kubernetes-introduction-service-and-kube-proxy
* iptable 介绍
  http://www.cnblogs.com/wangkangluo1/archive/2012/04/19/2457072.html
  https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
  http://www.zsythink.net/archives/1199

##### LoadBalancer: 
on top of having a cluster-internal IP and exposing service on a NodePort also, ask the cloud provider for a load balancer which forwards to the Service exposed as a :NodePort for each Node. 通常需要第三方云提供商支持,主要有aws、azure、openstack、gce等云平台

##### externalName 将外部服务引入k8s内部
  Maps the service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up. This requires version 1.7 or higher of kube-dns.
  ```yaml
  kind: Service
  apiVersion: v1
  metadata:
  name: my-service
  namespace: prod
  spec:
  type: ExternalName
  externalName: my.database.example.com # 用于访问集群外地址
  ```

##### External IPs  绑定到某台Node的IP
  In the example below, my-service can be accessed by clients on 80.11.12.10:80 (externalIP:port)
```yaml
kind: Service,
apiVersion: v1,
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http,
      protocol: TCP,
      port: 80,
      targetPort: 9376
  externalIPs: 
    - 80.11.12.10
```

### Ingress 
提供服务外部访问的URL，负载均衡（L7），SSL，提供基于名称的虚拟主机等。
Ingress 有两大组件：Ingress Controller 和 Ingress

#### Ingress 定义了什么样的host和path映射到哪个service的哪个端口.
一个简单的 Ingress 例子：
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress # Ingress 属于某个namespace，也只能关联到相同namespace下的service
spec:
  rules:
  - host: foo.bar.com # 后面ingress controller 会从http request header中取得"Referer:http://foo.bar.com" 还是 “Host: foo.bar.com”?
    http:
      paths:
      - path: /testpath
        backend:
          serviceName: test
          servicePort: 80
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80    
```
表示指向 http://foo.bar.com/testpath 的请求最终会访问到 service test:80 对应的 endpoints 上。
也可以多个host：
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
```

#### Ingress Controller 
本质上说Ingress只是存储在etcd上面一些数据，我们可以通过apiserver添加删除和修改ingress资源。那么真正让整个Ingress运转起来的一个重要组件是Ingress Controller。Ingress Controler 通过与 Kubernetes API 交互，动态的去感知集群中 Ingress 规则变化，然后读取他，按照他自己模板生成一段 Nginx 配置，再写到 Nginx Pod 里，最后 reload 一下。 Ingress controllers don’t route traffic to the service, but rather to the actual service endpoints i.e. pods. 

Ingress Controler可以监视同一namespace 或者所有namespace下的ingress， 由参数watch-namespace决定。https://github.com/kubernetes/ingress/blob/13c6b0e44c9fbc0ec0856007055892c99d89ad1c/controllers/gce/main.go#L125

Ingress controller例子：
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    k8s-app: nginx-ingress-controller-outside
  name: nginx-ingress-controller-outside
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: nginx-ingress-controller-outside
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        k8s-app: nginx-ingress-controller-outside
        name: nginx-ingress-controller-outside
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
        - --default-ssl-certificate=$(POD_NAMESPACE)/ingress-laikang-cert
        - --configmap=$(POD_NAMESPACE)/nginx-ingress-controller-outside-conf
        - --kubeconfig=/etc/kubernetes/kube-config
        - --election-id=ingress-controller-outside
        - --ingress-class=outside
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
        image: 127.0.0.1:30100/miao/nginx-ingress-controller:enn-0.9.0
        imagePullPolicy: IfNotPresent
        name: nginx-ingress-controller-outside
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 18080
          name: status
          protocol: TCP
        resources:
          limits:
            cpu: "2"
            memory: 8Gi
          requests:
            cpu: 500m
            memory: 2Gi
        volumeMounts:
        - mountPath: /etc/kubernetes
          name: kubeconfig
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      volumes:
      - configMap:
          defaultMode: 420
          name: kube-config
        name: kubeconfig
```

参考：[原理+代码分析1] (http://shareinto.github.io/2017/04/13/KubernetesIngress%EF%BC%881%EF%BC%89/)
when a HTTP request arrives in the cluster from the external traffic i.e. traffic from outside the K8 cluster. People wanted to know how do all the pieces such as service, ingress and DNS work together inside the cluster. 
http://containerops.org/2017/01/30/kubernetes-services-and-ingress-under-x-ray/
ingress config
https://docs.giantswarm.io/guides/advanced-ingress-configuration/
https://stackoverflow.com/questions/45079988/kubernetes-ingress-vs-load-balancer

Http header “Host” https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html  https://stackoverflow.com/questions/43156023/what-is-http-host-header

#### 解释service各属性含义。

| 字段                         | 是否必需 | 含义                                | 备注                                       |
| -------------------------- | ---- | --------------------------------- | ---------------------------------------- |
| Port                       | 是    | service对外暴露的端口号                   | /                                        |
| Protocol                   | 否    | 访问service端口的传输层协议                 | 有两种选择：TCP和UDP，如不指定，默认是TCP                |
| Labels                     | 否    | service对象的labels                  | /                                        |
| Selector                   | 否    | service对象的label selector          | /                                        |
| CreateExternalLoadBalancer | 否    | 用于指定是否在service的外部IP上创建一个基于云的负载均衡器 | 要求工作节点上运行在IaaS上且IaaS支持创建负载均衡器            |
| PublicIPs                  | 否    | 用于为service分配外部可访问IP的IP池           | /                                        |
| ContainerPort              | 否    | service后端pod中容器对外暴露的端口            | /                                        |
| PortalIP                   | 否    | service入口的IP地址                    | 用户不自己指定则由kube-apiserver从portal_net中随机选取一个 |
| SessionAffinity            | 否    | Used to maintain session affinity | 必须是`ClientIP`或`None`中的一个，默认值是`None`      |



### 集群外访问pod
hostNetwork, hostPort, NodePort, LoadBalancer and Ingress features of Kubernetes.
http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/
1. hostNetwork: true
   直接使用宿主机网络
```yaml
spec:
  hostNetwork: true
  containers:
    - name: influxdb
      image: influxdb
```
2. hostPort: 
   等于docker run -p 8080:8080
```yaml
spec:
  containers:
    - name: influxdb
      image: influxdb
      ports:
        - containerPort: 8086
          hostPort: 8086
```
3. NodePort
```yaml
kind: Service
apiVersion: v1
metadata:
  name: influxdb
spec:
  type: NodePort
  ports:
    - port: 8086
      nodePort: 30000
  selector:
    name: influxdb
```
3. Ingress


#### Service discovery：
Kubernetes主要支持两种 service 发现机制：环境变量和DNS
- 环境变量：
  当一个pod在一个工作节点上运行时，kubelet就会向它注入一组所有处于激活状态的service的环境变量，更具体地说是kubelet创建pod的参数里有service环境变量，然后如有需要，这些环境变量被注入pod内的容器中。这些环境变量是诸如{SVCNAME}_SERVICE_HOST和{SVCNAME}_SERVICE_PORT这样的变量，其中{SVCNAME}部分将service名字全部替换成大写且将破折号（-）替换成下划线（_）。假设service redis-master被分配了一个IP地址10.0.0.11并暴露一个TCP端口6379，那么将会生成以下这些环境变量。

REDIS_MASTER_SERVICE_HOST=10.0.0.11 
REDIS_MASTER_SERVICE_PORT=6379 
REDIS_MASTER_PORT=tcp://10.0.0.11:6379 
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379 
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp 
REDIS_MASTER_PORT_6379_TCP_PORT=6379 
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11

客户端只需连接REDIS_MASTER_SERVICE_HOST（在这个例子中是10.0.0.11，更为一般的是$MYAPP_SERVICE_HOST）上的REDIS_MASTER_SERVICE_PORT 
（在这个例子中是6379，更为一般的是$MYAPP_SERVICE_PORT）即可访问该service。

环境变量的注入过程其实蕴含着一个先后次序，即：任何想要访问service的pod都需要在service已经存在后创建，不然与service相关的环境变量就无法注入该pod的docker容器中。但如果使用DNS就不会有这样的限制，下文将会对service DNS作详细解释。
- DNS：
  Kubernetes集群现在支持增加一个可选的组件——DNS服务器。这个DNS服务器使用Kubernetes的watchAPI，不间断地监测新service的创建并为每个service新建一个DNS记录。如果DNS在整个集群范围内都可用，那么所有pod都能够自动解析service的域名。例如，假设你有一个名为my-service的service，且该service处在namespace my-ns中，这样就形成了一条DNS记录：my-service.my-ns。在namespace my-ns中的pod只要查找域名my-service即可发现该service，而处在其他namespace的的pod则必须指定完整的namespace前缀my-service，即：my-service.my-ns。DNS域名解析的结果就是一个service的入口IP地址（portal IP）。

参考：
- 官网 https://kubernetes.io/docs/concepts/services-networking/service/
- 什么是service以及为什么需要service https://www.zybuluo.com/dujun/note/60321
- https://sysdig.com/blog/understanding-how-kubernetes-services-dns-work/
- [外网访问service -Ingress](https://mritd.me/2017/03/04/how-to-use-nginx-ingress/)

### 任务（Job）
Job是Kubernetes用来控制批处理型任务的API对象。批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job管理的Pod根据用户的设置把任务成功完成就自动退出了。成功完成的标志根据不同的spec.completions策略而不同：单Pod型任务有一个Pod成功就标志完成；定数成功型任务保证有N个任务全部成功；工作队列型任务根据应用确认的全局成功而标志成功。


### 存储 - 数据卷Volumes:
[官方网站](https://kubernetes.io/docs/concepts/storage/volumes/)
[Kubernetes持久化存储1——PV示例](http://www.cnblogs.com/zhenyuyaodidiao/p/6626023.html)
[Kubernetes持久化存储2——探究实验](http://www.cnblogs.com/zhenyuyaodidiao/p/6632600.html)
[running-stateful-applications-kubernetes-volumes/](https://thenewstack.io/strategies-running-stateful-applications-kubernetes-volumes/)

Kubernetes volumes may be classified into host-based storage and non-host-based storage types. Host-based storage is similar to Docker volumes, where a portion of the host’s storage becomes available to the pod. Once a pod is terminated, the volume gets automatically deleted. Non-host-based storage doesn’t rely on a specific node. Instead, a storage volume is created from an external storage service.Volumes based on this storage type would be available even after the pods are deleted.
要使用卷，需要为 pod 指定为卷（spec.volumes 字段）以及将它挂载到容器的位置（spec.containers.volumeMounts 字段）。

#### emptyDir 临时空间
如果Pod配置了emptyDir类型Volume， Pod 被分配到Node上时候，会创建emptyDir，只要Pod运行在Node上，emptyDir都会存在（容器挂掉不会导致emptyDir丢失数据），但是如果Pod从Node上被删除（Pod被删除，或者Pod发生迁移），emptyDir也会被删除，并且永久丢失。
emptyDir is one of volume types in Kubernetes, it share the lifetime with Pod which means that it exists as long as the Pod is running on that node. Containers in same pod share the emptyDir, its quite helpful when you want to access the same files across Pod. when a pod is removed, the data will be lost.

[POD中的多个容器共享存储](http://www.stratoscale.com/blog/kubernetes/kubernetes-how-to-share-disk-storage-between-containers-in-a-pod/)

#### hostPath 对应host机器的某目录／文件
hostPath 卷将主机节点的文件系统中的文件或目录挂载到集群中。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```

#### NFS  
允许一块现有的网络硬盘在同一个pod内的容器间共享。This type allows us to mount a Network File Share (NFS), which can be very useful for both persisting the data and sharing it across the infrastructure

#### Glusterfs

GlusterFS is a scalable network filesystem

#### RBD(Rados Block Device)
rbd卷可以将Rados Block Device设备映射到pod中。当Pod被移除时，rbd卷的内容还存在着，仅仅是卷被卸载掉而已。也就是说，rbd卷可以其上的数据一起，再次被映射，数据也可以在pod之间传递。
RBD volumes can only be mounted by a single consumer in read-write mode - no simultaneous writers allowed.
```json
"volumes": [
            {
                "name": "rbdpd",
                "rbd": {
                    "monitors": [
        						"10.16.154.78:6789",
						        "10.16.154.82:6789",
        						"10.16.154.83:6789"
    				 ],
                    "pool": "kube",
                    "image": "foo",
                    "user": "admin",
                    "keyring": "/etc/ceph/keyring",
                    "fsType": "ext4",
                    "readOnly": true
                }
            }
        ]
```

#### Cephfs
  cephfs卷可以将已经存在的CephFS卷映射到pod中。与rbd卷相同，当pod被移除时，cephfs卷的内容还存在着，仅仅是卷被卸载掉而已。另外一点不同的是，CephFS可以同时以可读写的方式映射给多个用户。

```yaml
volumes:
  - name: cephfs
    cephfs:
      monitors:
      - 10.16.154.78:6789
      - 10.16.154.82:6789
      - 10.16.154.83:6789
      # by default the path is /, but you can override and mount a specific path of the filesystem by using the path attribute
      # path: /some/path/in/side/cephfs 
      user: admin
      secretFile: "/etc/ceph/admin.secret"
      readOnly: true
```

#### GitRepo 
this option clones a Git repo into an a new and empty folder

#### Secret
Kubemetes提供了Secret来处理敏感数据，比如密码、Token和密钥，相比于直接将敏感数据配置在Pod的定义或者镜像中，Secret提供了更加安全的机制（Base64加密），防止数据泄露。Secret的创建是独立于Pod的，以数据卷的形式挂载到Pod中，Secret的数据将以文件的形式保存，容器通过读取文件可以获取需要的数据。yaml示例如下：

```Yaml
// 定义
apiVersion: v1
kind: Secret
metadata:
  name: mysecrets
data:
  password: dmFsdWUtMg0K
  username: someuser
```
```yaml
//使用
        volumeMounts:
          # must match the name of the volume name below
          - name: my-passwords
         readOnly: true
            # mount path within the container
            mountPath: /etc/my-passwords-volume
    # volumes to be mounted to containers
      volumes:
      - name: my-passwords
        secrect:
          secrectName: mysecrects
```

#### 持久化卷PersistentVolume, 卷声明PersistentVolumeClaim and StorageClass
  对于持久化的Volume，PersistentVolume (PV)和PersistentVolumeClaim (PVC)提供了更方便的管理卷的方法：PV提供网络存储资源，而PVC请求存储资源。这样，设置持久化的工作流包括配置底层文件系统或者云数据卷、创建PV、最后PVC来将pod跟数据卷关联起来。PV和PVC可以将pod和数据卷解耦，pod不需要知道确切的文件系统或者支持它的持久化引擎。
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
PV是由管理员添加的的一个存储的描述，是一个全局资源，包含存储的类型（ceph，GlusterFS，NFS），存储的大小和访问模式等。它的生命周期独立于Pod，例如当使用它的Pod销毁时对PV没有影响。
PVC描述对PV的一个请求。请求信息包含存储大小，访问模式等。

PV
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /tmp
    server: 172.17.0.2
```

PVC:
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```
PVC可以直接挂载到Pod中：
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: dockerfile/nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

### 后台支撑服务集 DaemonSet

DaemonSet能够让所有（或者一些特定）的Node节点运行同一个pod。当节点加入到kubernetes集群中，pod会被（DaemonSet）调度到该节点上运行，当节点从kubernetes集群中被移除，被（DaemonSet）调度的pod会被移除，如果删除DaemonSet，所有跟这个DaemonSet相关的pods都会被删除。典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持Kubernetes集群运行的服务。

  在使用kubernetes来运行应用时，很多时候我们需要在一个区域（zone）或者所有Node上运行同一个守护进程（pod），例如如下场景：

- 每个Node上运行一个分布式存储的守护进程，例如glusterd，ceph
- 运行日志采集器在每个Node上，例如fluentd，logstash
- 运行监控的采集端在每个Node，例如prometheus node exporter，collectd等
  在简单的情况下，一个DaemonSet可以覆盖所有的Node，来实现Only-One-Pod-Per-Node这种情形；在有的情况下，我们对不同的计算几点进行着色，或者把kubernetes的集群节点分为多个zone，DaemonSet也可以在每个zone上实现Only-One-Pod-Per-Node。

### 有状态服务集（PetSet -> StatefulSet）

http://www.tuicool.com/articles/Z3yEJzz
https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

### 集群联邦（Federation）

### 节点（Node）

Kubernetes集群中的计算能力由Node提供，最初Node称为服务节点Minion，后来改名为Node。Kubernetes集群中的Node也就等同于Mesos集群中的Slave节点，是所有Pod运行所在的工作主机，可以是物理机也可以是虚拟机。不论是物理机还是虚拟机，工作主机的统一特征是上面要运行kubelet管理节点上运行的容器。

### 密钥对象（Secret）

Secret是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。使用Secret的好处是可以避免把敏感信息明文写在配置文件里。在Kubernetes集群中配置和使用服务不可避免的要用到各种敏感信息实现登录、认证等功能，例如访问AWS存储的用户名密码。为了避免将类似的敏感信息明文写在所有需要使用的配置文件中，可以将这些信息存入一个Secret对象，而在配置文件中通过Secret对象引用这些敏感信息。这种方式的好处包括：意图明确，避免重复，减少暴漏机会。

### 用户帐户（User Account）和服务帐户（Service Account）

顾名思义，用户帐户为人提供账户标识，而服务账户为计算机进程和Kubernetes集群中运行的Pod提供账户标识。用户帐户和服务帐户的一个区别是作用范围；用户帐户对应的是人的身份，人的身份与服务的namespace无关，所以用户账户是跨namespace的；而服务帐户对应的是一个运行中程序的身份，与特定namespace是相关的。

### Namespace:

名字空间为Kubernetes集群提供虚拟的隔离作用，Kubernetes集群初始有两个名字空间，分别是默认名字空间default和系统名字空间kube-system，除此以外，管理员可以可以创建新的名字空间满足需要。

### RBAC访问授权（Role-based Access Control)

Kubernetes在1.3版本中发布了alpha版的基于角色的访问控制（Role-based Access Control，RBAC）的授权模式。相对于基于属性的访问控制（Attribute-based Access Control，ABAC），RBAC主要是引入了角色（Role）和角色绑定（RoleBinding）的抽象概念。在ABAC中，Kubernetes集群中的访问策略只能跟用户直接关联；而在RBAC中，访问策略可以跟某个角色关联，具体的用户在跟一个或多个角色相关联。显然，RBAC像其他新功能一样，每次引入新功能，都会引入新的API对象，从而引入新的概念抽象，而这一新的概念抽象一定会使集群服务管理和使用更容易扩展和重用。这个功能在1.6中默认打开。



### ConfigMap

ConfigMap API用于存储已经在kubernetes中部署的配置数据，它主要聚焦：
- 为已经部署的应用提供动态的、分布式的配置管理
- 封装配置管理信息，简化kubernetes的部署管理
- 为kubernetes创建一个灵活的配置管理模型
  配置数据为key，value形式，也可直接存放文件。

关于热更新，更新 ConfigMap 后：
- 使用该 ConfigMap 挂载的 Env 不会同步更新
- 使用该 ConfigMap 挂载的 Volume 中的数据需要一段时间（实测大概10秒）才能同步更新

## 参考资料：
- [K8s 介绍](http://cizixs.com/2016/07/12/kubernetes-intro)
- [Kubernetes Overview 1](https://deis.com/blog/2016/kubernetes-overview-pt-1/)
- [Kubernetes Overview 2](https://deis.com/blog/2016/kubernetes-overview-pt-2/)
- Kubernetes权威指南  pdf
- [总体架构](https://github.com/kubernetes/kubernetes/blob/release-1.5/docs/design/architecture.md)
- [客户端命令](https://linfan1.gitbooks.io/kubernetes-chinese-docs/content/150-kubectl_exec.html)
- [Pod 调度](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)
- [kubernetes handbook](https://jimmysong.io/kubernetes-handbook)
