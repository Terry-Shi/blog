```shell
// 安装 dnsmasq
brew install dnsmasq

// 如果目录 /usr/local/sbin 不存在的话 需要创建+设置权限
mkdir /usr/local/sbin
sudo chmod 777 /usr/local/sbin
brew link dnsmasq

// 上面这步会创建 symbolic link 
// ls -sf /usr/local/Cellar/dnsmasq/2.75/sbin/dnsmasq  /usr/local/sbin/dnsmasq

// 复制配置文件

cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
// 启动dnsmasq
sudo brew services start dnsmasq
```
#### 参考
- 用Mac打造自己的DNS服务器 http://www.jianshu.com/p/3dd22d7d86b2
