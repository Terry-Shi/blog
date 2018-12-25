1 pom.xml 中增加本地repo
假设工程根目录下libs目录存放jar文件。
要引入的依赖为 libcephfs-10.2.6.jar 
需要将 jar 文件放到 {project.basedir}/libs/com/ceph/libcephfs/10.2.6 目录下

```xml
    <repositories>
	    <repository>
	        <id>my-local-repo</id>
	        <url>file://${project.basedir}/libs</url>
	    </repository>
    </repositories>

    <dependencies>
        <dependency>
	        <groupId>com.ceph</groupId>
	        <artifactId>libcephfs</artifactId>
	        <version>10.2.6</version>
	    </dependency>
    </dependencies>
```

相应的gradle的方案比较简单一些 
build.gradle文件中配置
``` groovy
dependencies {
    compile files('libs/libcephfs.jar')
}
```
libcephfs.jar 只需要放到工程根目录/libs下。不需要用groupId，artifactId和version拼接目录。

