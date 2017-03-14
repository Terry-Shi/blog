“Java程序员修炼之道“ NIO2 学习笔记

### 第二章 NIO2 （Java7 新功能）

#### 主要功能与改善
- 以Path代表一个文件或目录，用Files帮助类实现文件的create，delete，copy，move。
- 增强文件属性的读取，不仅仅是size，lastModifiedTime，isDir 还有是否为SymbolicLink，*nix下的读写权限等某操作系统特有的属性
- 提供遍历目录树的功能
- 监控目录下的任何变化
- 改善随机读写API
- 改善异步读写的API

 #### Java6  IO的缺点／功能缺失

| Java 6          | Java 7                 |
| --------------- | ---------------------- |
| 没有遍历目录树的API     | Files.walkFileTree()   |
| 不能判断symbol link | Files.isSymbolicLink() |
| 不能读属性           | Files.readAttributes() |
| 没有copy文件的API    | Files.copy()           |



#### 2.1 Java I/O—a history 

#### 2.2  Path 

- Path 是一个关键类，它代表了一个目录，文件或symbolic，例如C:\workspace\java7developer 或者/usr/bin/zip，甚至zip archive 


- PATH 相关API
```Java
Path listing =Paths.get("/usr/bin/zip");
listing.getFileName()
listing.getNameCount()
listing.getParent()
listing.getRoot()
listing.subpath(0,2) 
```

```Java
// 找到 symobolic 对应的真正地址
Path normalizedPath = Paths.get("./Listing_2_1.java").normalize();
Path realPath = Paths.get("/usr/logs/log1.txt").toRealPath();
```

##### 2.2.5

- 新的I/O类可以完全替代老的I/O类（Path替代了File）
- 新旧API之间可以转换
```Java
File file = new File("../Listing_2_1.java");
Path listing = file.toPath();
listing.toAbsolutePath();
file = listing.toFile();
```

#### 2.3 Dealing with directories and directory trees 
目标：遍历目录，过滤，进行递归操作

##### 2.3.1 Finding files in a directory （不递归）

```java
Path dir = Paths.get("C:\\workspace\\java7developer");
try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir, "*.properties")) {
    for (Path entry: stream) {
        System.out.println(entry.getFileName());
    }
} catch (IOException e) {
    System.out.println(e.getMessage());
}
```
##### 2.3.2  Walking the directory tree 递归遍历一个目录树

```java
Path startingDir = Paths.get("C:\\workspace\\java7developer\\src");
Files.walkFileTree(startingDir, new FindJavaVisitor());
```

#### 2.4 Filesystem I/O with NIO.2 
`Files` 是重要的帮助类（helper class），可以create, move，copy，delete文件或目录。

##### 2.4.1 Creating and deleting files

```java
Path target = Paths.get("D:\\Backup\\MyStuff.txt");
Path file = Files.createFile(target);

Files.delete(target);
```
##### 2.4.2 Copying and moving files

```java
Path source = Paths.get("C:\\My Documents\\Stuff.txt");
Path target = Paths.get("D:\\Backup\\MyStuff.txt");
Files.copy(source, target);
// specify options with the copy operation，其他选项还有 COPY_ATTRIBUTES ATOMIC_MOVE ...
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
// 移动文件
Files.move(source, target, REPLACE_EXISTING, COPY_ATTRIBUTES);
```

##### 2.4.3 File attributes
- BASIC FILE ATTRIBUTE SUPPORT
  - last modified time?
  - What is its size?
  - Is it a symbolic link?
  - Is it a directory?

```java
Path zip = Paths.get("/usr/bin/zip");
System.out.println(Files.getLastModifiedTime(zip));
System.out.println(Files.size(zip));
System.out.println(Files.isSymbolicLink(zip));
System.out.println(Files.isDirectory(zip));
System.out.println(Files.readAttributes(zip, "*")); // bulk read
```
- SPECIFIC FILE ATTRIBUTE SUPPORT (filesystem-specific file attributes)
  TODO：

- SYMBOLIC LINKS

  ```java
  Files.readAttributes(target,
                       BasicFileAttributes.class,
  					LinkOption.NOFOLLOW_LINKS);
  ```
##### 2.4.4 Reading and writing data quickly

一行一行读文件

```java
Path logFile = Paths.get("/tmp/app.log");
try (BufferedReader reader = Files.newBufferedReader(logFile, StandardCharsets.UTF_8)) {
    String line;
    while ((line = reader.readLine()) != null) {
     ...
    }
}
```
写文件
```java
Path logFile = Paths.get("/tmp/app.log");
try (BufferedWriter writer =
     Files.newBufferedWrite(logFile, StandardCharsets.UTF_8,
                            StandardOpenOption.WRITE)) {
  writer.write("Hello World!");
}
```



##### 2.4.5 File change notification

- 文件变化监视 java.nio.file.WatchService http://blog.csdn.net/chuchus/article/details/43672967
- 使用 Watch Service API 觀察檔案系統 http://stevenitlife.blogspot.com/2014/06/watch-service-api.html

```java
public static void main(String[] args) {
		new TestWatchService().run();
	}

	public void run() {
		final Path path = Paths.get("/tmp");
		WatchService watchService;
		try {
			watchService = FileSystems.getDefault().newWatchService();

			path.register(watchService, StandardWatchEventKinds.ENTRY_CREATE, StandardWatchEventKinds.ENTRY_MODIFY,
					StandardWatchEventKinds.ENTRY_DELETE);

			while (true) {
				final WatchKey key = watchService.take();

				for (WatchEvent<?> watchEvent : key.pollEvents()) {
					final WatchEvent<Path> watchEventPath = (WatchEvent<Path>) watchEvent;
					final Path filename = watchEventPath.context();

					System.out.println("kind: " + watchEvent.kind().toString());
					System.out.println("filename: " + filename.toString());
				}
				key.reset();
				break;
			}

			watchService.close();
			System.out.println("exit");
		} catch (IOException | InterruptedException e) {
			e.printStackTrace();
		}
	}
	/*
	 * 例如创建文件abc.txt在 /tmp目录下的输出: 
	 * kind: ENTRY_CREATE 
	 * filename: abc.txt 
	 * exit
	 */
```


##### 2.4.6 SeekableByteChannel
Java7中引入了SeekableByteChannel接口，允许我们定位到文件的任意位置进行读写。**注意这里的写，不是新增式的插入，而是覆盖**，当然在文件末尾的写，是新增。

```java
public class SeekableMain {
    
    private static final int BUFFER_SIZE = 64;

    public static void main(String[] args) throws IOException {
        Path path = Paths.get("path_to_file");
        System.out.println("--------------------------------");
        readAndDisplay(path);
        System.out.println("--------------------------------");
        writeAndDisplayAll(path);
    }
    
    /*
     * SeekableByteChannelとByteBufferを用いて任意の位置から指定バイトの文字を読み込み出力する。
     */
    private static void readAndDisplay(final Path path) throws IOException {
        try (SeekableByteChannel channel = getSeekableByteChannel(path)) {
            ByteBuffer buffer = ByteBuffer.allocate(BUFFER_SIZE);
            // beautifulを出力
            display(channel, buffer, 9, 9);
            buffer.clear();
            // 「when I woke up」を出力
            display(channel, buffer, 53, 14);
        }
    }
    
    private static void display(SeekableByteChannel channel, ByteBuffer buffer, int start, int length) throws IOException {
        channel.position(start);
        channel.read(buffer);
        for (int i = 0; i < length; i++) {
            System.out.print((char)buffer.get(i));
        }
        System.out.println();
    }
    
    /*
     * 指定位置に特定の文字列を書き込む。
     */
    private static void writeAndDisplayAll(final Path path) throws IOException {
        // 書き込みモードでチャネルオープン
        try (SeekableByteChannel channel = getSeekableByteChannel(path, StandardOpenOption.WRITE)) {
            // classをgrassに修正
            String modifier = "grass";
            channel.position(124);
            channel.write(ByteBuffer.wrap(modifier.getBytes()));
        }
        // 追記モードでチャネルオープン
        try (SeekableByteChannel channel = getSeekableByteChannel(path, StandardOpenOption.APPEND)) {
            // 最終行に1文追加
            String appended = " Clouds were moving fast in the sky.";
            channel.write(ByteBuffer.wrap(appended.getBytes()));
        }
        // BufferedReaderで全ファイル内容を出力する。
        try (BufferedReader reader = Files.newBufferedReader(path, Charset.defaultCharset())) {
            String line = null;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }
    }
    
    private static SeekableByteChannel getSeekableByteChannel(Path path, StandardOpenOption... opts) throws IOException {
        return Files.newByteChannel(path, opts);
    }

}
```

执行前

```
It was a beautiful Sunday morning.
The rain was gone when I woke up. So I opened the window to look out.
I saw a dew on the class and heard the birds sing.
```

执行后

```
--------------------------------
beautiful
when I woke up
--------------------------------
It was a beautiful Sunday morning.
The rain was gone when I woke up. So I opened the window to look out.
I saw a dew on the grass and heard the birds sing. Clouds were moving fast in the sky.
```

#### 2.5 Asynchronous I/O operations

TODO:


### 参考
* NIO.2 入门，第 2 部分: 文件系统 API https://www.ibm.com/developerworks/cn/java/j-nio2-2/index.html
* Java NIO 系列教程 http://blog.csdn.net/tzs_1041218129/article/details/54917857