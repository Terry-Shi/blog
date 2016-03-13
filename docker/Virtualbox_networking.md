### Virtualbox网络模式
1. not attached: 有网卡但是网线没接上
2. Network Address Translation (NAT)  默认模式:  虚拟机-> 宿主机 ok；访问Internet ok，外部无法访问虚拟机内部。 效果等价于virtualbox建立了虚拟路由器，DHCP server，让后让虚拟机连上。
3. NAT Network
4. Bridged networking：它是通过主机网卡，架设了一条桥，直接连入到网络中了。因此，它使得虚拟机能被分配到一个网络中独立的IP，看上去和宿主机的网络地位平等，所有网络功能完全和在网络中的真实机器一样。
网桥模式下的虚拟机，你把它认为是真实计算机就行了。这个模式的缺点是需要绑定到宿主机的网卡上，如果你在宿主机WIFI上网情况下配置好了，第二天又用网线＋有线网卡上网就会发现虚拟机上不了网了。
5. Internal networking:  只在特定选中的虚拟机中连通， 宿主机无法与之通信
6. Host-only networking：宿主机和一批虚拟机之间可连通。
7. Generic networking 很少使用


### 参考资料
- https://blogs.oracle.com/fatbloke/entry/networking_in_virtualbox1
- http://www.cnblogs.com/adforce/archive/2013/10/11/3363373.html
- 官方文档 https://www.virtualbox.org/manual/ch06.html

