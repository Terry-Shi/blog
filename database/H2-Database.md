# H2 嵌入式数据库最佳选择 #

## How to start a H2 server ##
**Embedded mode**

`Class.forName("org.h2.Driver");
Connection conn = DriverManager.getConnection("jdbc:h2:E:\\research\\workspace\\H2Test\\db\\DBName", "sa", "");`
这个模式下其他进程无法访问这个数据库。

**Server mode (aka TCP server mode)**

1.Command Line:

start server: `java -cp h2*.jar org.h2.tools.Server`

启动参数很多，可以通过 `java -cp h2*.jar org.h2.tools.Server -?` 查看。

> 一个实际例子
> `nohup java -cp h2*.jar org.h2.tools.Server -tcp -tcpPort 9092 -webAllowOthers -tcpAllowOthers -web -baseDir /opt/mount/health_check_center/db_file/`
> 
> JDBC连接：`jdbc:h2:tcp://C0004717.itcs.hp.com:9092/test;AUTO_SERVER=TRUE"`
> 
> web browser访问 `http://C0004717.itcs.hp.com:8082`

stop server: `java org.h2.tools.Server -tcpShutdown tcp://localhost:9092`


2.within a java Application

	// start the TCP Server:
	
	`Server server = Server.createTcpServer(args).start();`
	
	// stop the TCP Server over server object:(when start/stop in same JVM process)
	
	`server.stop();`
	
	// stop the TCP Server over TCP:
	
	`org.h2.tools.Server.shutdownTcpServer("tcp://localhost:9094");`


## How to connect to a database ##

**• Embedded mode (don't need server)**

`jdbc:h2:mem:DBName;DB_CLOSE_DELAY=-1`

参数`DB_CLOSE_DELAY`是要求最后一个正在连接的连接断开后，不要关闭DB，因为下一个case可能还会有新连接进来。

**• Server mode (remote connections using JDBC or ODBC over TCP/IP)**

`jdbc:h2:tcp://localhost/~/DBName`

**• Mixed mode (local and remote connections at the same time)**

`jdbc:h2:file:~/.h2/DBName;AUTO_SERVER=TRUE` 

AUTO_SERVER=TRUE表示：第一个打开数据库文件的进程，将自觉兼任H2 Server的角色，并把自己的地址和端口写到lock文件里， 第二个连进来的进程，发现lock文件存在，就会读取并通过网络方式来访问H2。第一个连进来的进程因为是直接访问数据库文件，性能会较好一些。

**In all modes, both persistent and in-memory databases are supported**

使用内存模式的方法：
`Connection conn = DriverManager.getConnection("jdbc:h2:tcp://localhost/mem:DBName", "sa", "");`

## How to open a H2 console ##
The H2 Console is a small web app, let you connect to DB in browser. (It's not just use to connect H2, could be use with any database which support JDBC)

![H2 Console](http://www.h2database.com/html/images/console-2.png)

启动命令 `java -jar h2*.jar -web -webPort 8090 -browser`
这时可以通过http://localhost:8090 访问console。


*By default, if the database specified in the URL does not yet exist, a new (empty) database is created automatically. The user that created the database automatically becomes the administrator of this database.


### Ref 
[H2 website tutorial](http://www.h2database.com/html/tutorial.html)

[spring side wiki](https://github.com/springside/springside4/wiki/H2Database)
