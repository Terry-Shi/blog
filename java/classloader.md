### Java ClassLoader

#### 定义
Java类加载就是把class文件读入内存，生成java.lang.Class对象的过程。

#### Class 加载的三个阶段
1. 载入Load
从Class文件或别的什么地方载入一段二进制流字节流，把它解释成永久代里的运行时数据结构（JDK8开始没有永久代），生成一个Class对象。
同时也会装载它的父类或父接口，并进行校验，比如编译版本不对会抛出UnsupportedClassError，依赖定义不对会抛 IncompatibleClassChangeError/ClassCircularityError等。
2. 链接Resolve
将之前载入的数据结构里的符号引用表，解析成直接引用。
中间如果遇到引用的类还没被加载，就会触发该类的加载。
可能JDK会很懒惰的在运行某个函数实际使用到该引用时才发生链接，也可能在类加载时就解析全部引用。
3. 初始化Initniazle
初始化静态变量，并执行静态初始化语句。

#### 什么时候会触发class的加载
1. ClassLoader.loadClass()
2. 前文所说的链接时触发的装载
3. Class.forName() 等java.lang.reflect反射包
4. new 构造对象
5. 初始化子类时，会同时初始化父类
6. 访问类的静态变量或静态方法(但static final的常量除外，此君在常量池里)

#### 三个系统内置的Classloader
bootstrap classloader － 引导（也称为原始）类加载器，它负责加载Java的核心类。在Sun的JVM中，在执行java的命令中使用-Xbootclasspath选项或使用 -D选项指定sun.boot.class.path系统属性值可以指定附加的类。这个加载器的是非常特殊的，它实际上不是 java.lang.ClassLoader的子类，而是由底层C++语言实现的。在JVM中无法得到这个加载器的引用。rt.jar 就是由这个加载器记载的。

extension classloader － 扩展类加载器，它负责加载JRE的扩展目录（JAVA_HOME/jre/lib/ext或者由java.ext.dirs系统属性指定的）中JAR的类 包。这为引入除Java核心类以外的新功能提供了一个标准机制。因为默认的扩展目录对所有从同一个JRE中启动的JVM都是通用的，所以放入这个目录的 JAR类包对所有的JVM和system classloader都是可见的。在这个实例上调用方法getParent()总是返回空值null，因为引导加载器bootstrap classloader不是一个真正的ClassLoader实例。

system classloader － 系统（也称为AppClassloader）类加载器，它负责在JVM被启动时，加载来自在命令java中的-classpath或者java.class.path系统属性或者 CLASSPATH操作系统属性所指定的JAR类包和类路径。总能通过静态方法ClassLoader.getSystemClassLoader()找到该类加载器。

#### JVM启动后加载过程
源码分析 https://my.oschina.net/xionghui/blog/499725
[源代码在sun.misc.Launcher中](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/sun/misc/Launcher.java/)
```java
public  Launcher() {
    // Create the extension class loader
    ClassLoader extcl;
    try {
             extcl = ExtClassLoader.getExtClassLoader();
         } catch (IOException e) {
             throw new InternalError(
                 "Could not create extension class loader", e);
         }
 
         // Now create the class loader to use to launch the application
         try {
             loader = AppClassLoader.getAppClassLoader(extcl);
         } catch (IOException e) {
             throw new InternalError(
                 "Could not create application class loader", e);
         }
 
         // Also set the context class loader for the primordial thread.
         Thread.currentThread().setContextClassLoader(loader);
         // ... 省略
```
也就是bootstrap 创建了ExtClassLoader and AppClassLoader，并设置Thread ContextClassLoader为AppClassLoader。

#### 层级关系  
三种ClassLoader存在父子关系, App ClassLoader的父类加载器是Extension ClassLoader, Extension ClassLoader的父类加载器是Bootstrap ClassLoader, 要注意的一点是,这里的父子并不是类的继承关系.

#### 双亲委派机制delegation mode
系统内置的加载器采用委派的模式加载
所谓双亲委派机制，就是先从parent loader开始查找，找不到了才用自己的findClass()函数去查找，兼顾了效率：避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次，和安全：避免子类乱加载。
a class is loaded in Java, when its needed. Suppose you have an application specific class called Abc.class, first request of loading this class will come to Application ClassLoader which will delegate to its parent Extension ClassLoader which further delegates to Bootstrap class loader. Bootstrap will look for that class in rt.jar and since that class is not there, request comes to Extension class loader which looks on jre/lib/ext directory and tries to locate this class there, if class is found there than Extension class loader will load that class and Application class loader will never load that class but if its not loaded by extension class-loader then Application class loader loads it from [Classpath in Java](http://java67.blogspot.sg/2012/08/what-is-path-and-classpath-in-java-difference.html). 

```java
// ClassLoader.loadClass()代码说明
// 检查类是否已被装载过
Class c = findLoadedClass(name);
if (c == null ) {
     // 指定类未被装载过
     try {
         if (parent != null ) {
             // 如果父类加载器不为空， 则委派给父类加载
             c = parent.loadClass(name, false );
         } else {
             // 如果父类加载器为空， 则委派给启动类加载加载
             c = findBootstrapClass0(name);
         }
     } catch (ClassNotFoundException e) {
         // 启动类加载器或父类加载器抛出异常后， 当前类加载器将其
         // 捕获， 并通过findClass方法， 由自身加载
         c = findClass(name);
     }
}
```

#### 可见性原则
1. According to this principle, Classes loaded by parent classloaders are visible to child classloaders but not vice versa
2. For classes loaded by ClassLoader X, classes A, B and X are visible, but not class Y. Sibling classloaders cannot see each other’s classes.
#### 唯一性原则
According to this principle, when a classloader loads a class, the child classloaders in the hierarchy will never reload the class.

#### Class.forName(以jdbc driver为例) 
JDK7为例 java.sql.DriverManager 源码
```Java
      // 一段典型的 JDBC 代码
      //STEP 1: Register JDBC driver
      Class.forName("com.mysql.jdbc.Driver");

      //STEP 2: Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL, USER, PASS);

      //STEP 3: Execute a query
      System.out.println("Creating statement...");
      stmt = conn.createStatement();
      String sql;
      sql = "SELECT id, first, last, age FROM Employees";
      ResultSet rs = stmt.executeQuery(sql);
      // 省略后面代码
```
Connection java.sql.DriverManager.getConnection(String url, String user, String password)
使用Reflection.getCallerClass()获得加载器

#### Thread ContextClassLoader
http://www.blogjava.net/jackjhy/archive/2008/05/30/204163.html
每一个Thread有一个相关联系的Context ClassLoader，默认是父线程的classloader
有的时候委派模式不适用，此时需要ContextClassLoader

##### Log4j in J2EE
http://archive.oreilly.com/pub/a/onjava/2003/04/02/log4j_ejb.html
log4j source code: http://grepcode.com/file/repo1.maven.org/maven2/log4j/log4j/1.2.17/org/apache/log4j/helpers/Loader.java/


#### 自定义loader或者说“非JDK内置”的Loader载入类时如果要违背委派模式
自定义的加载器怎么判断一个类应该是自己加载还是交给上层loader加载？按package名
例子：javax.xml.parsers.DocumentBuilderFactory类中的 newInstance()方法用来生成一个新的 DocumentBuilderFactory的实例。这里的实例的真正的类是继承自 javax.xml.parsers.DocumentBuilderFactory，由 SPI 的实现所提供的。如在 Apache Xerces 中，实现的类是 org.apache.xerces.jaxp.DocumentBuilderFactoryImpl。而问题在于，SPI 的接口是 Java 核心库的一部分，是由引导类加载器来加载的；SPI 实现的 Java 类一般是由系统类加载器来加载的。引导类加载器是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。它也不能代理给系统类加载器，因为它是系统类加载器的祖先类加载器。也就是说，类加载器的代理模式无法解决这个问题。 

#### classloader 的父子关系是怎么设定的？
通过java.lang.ClassLoader 的构造方法可知，如果外部传入parent参数 则设置this.parent = parent
如果不传，则默认使用system classloader。 源代码如下：
```Java
private ClassLoader(Void unused, ClassLoader parent) {
        this.parent = parent;
        //...
}

protected ClassLoader() {
        //getSystemClassLoader()
        this(checkCreateClassLoader(), getSystemClassLoader());
}
```

#### 定义加载器和初始加载器的定义？

#### 类的访问权限(保护域)是如何指定的
http://www.importnew.com/17093.html

#### 参考资料
- 王森  《Java深度历险》相应章节
- Core java 相应章节
- ClassLoader, JavaAgent, Aspectj Weaving一站式扫盲http://calvin1978.blogcn.com/articles/classloader-javaagent.html
- Java类加载机制深度分析https://my.oschina.net/xianggao/blog/70826
- http://archive.oreilly.com/pub/a/onjava/2005/01/26/classloading.html
- http://stackoverflow.com/questions/2424604/what-is-a-java-classloader
- http://stackoverflow.com/questions/1974705/log4j-and-the-thread-context-classloader
- http://stackoverflow.com/questions/1771679/difference-between-threads-context-class-loader-and-normal-classloader