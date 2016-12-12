### 

Tomcat部署应用时，不知道你是否曾注意过这几点：
1. 如果在一个Tomcat内部署多个应用，甚至多个应用内使用了某个类似的几个不同版本，但它们之间却互不影响。这是如何做到的？
2. 如果多个应用都用到了某类似的相同版本，是否可以统一提供，不在各个应用内分别提供，占用内存呢。
3. 还有时候，在开发Web应用时，在pom.xml中添加了servlet-api的依赖，那实际应用的class加载时，会加载你的servlet-api 这个jar吗？

tomcat 中有多个classloader： commonLoader，catalinaLoader与sharedLoader
在catalina.properties 中定义了 common.loader,  server.loader and shared.loader 的对应类文件的路径，默认值为：
common.loader="${catalina.base}/lib","${catalina.base}/lib/*.jar","${catalina.home}/lib","${catalina.home}/lib/*.jar"
server.loader=
shared.loader=

但是可以简化为以下加载器结构图
参考：http://tomcat.apache.org/tomcat-8.0-doc/class-loader-howto.html
     Bootstrap
          |
       System
          |
       Common
       /     \
  Webapp1   Webapp2 ...


每一个web应用程序都有自己专用的WebappClassLoader，用于加载属于自己应用程序的资源，例如/web-inf/lib下面的jar包，classes里面的class文件





Ref:  
- Tomcat类加载器机制（Tomcat源码解析六）http://m.blog.csdn.net/article/details?id=47416007
- Tomcat classloader源码分析 [http://blog.csdn.net/dviewer/article/details/51817120](http://blog.csdn.net/dviewer/article/details/51817120)
- Tomcat 源代码分析之ClassLoader http://redhat.iteye.com/blog/1405709
官方网站 https://tomcat.apache.org/tomcat-8.0-doc/class-loader-howto.html