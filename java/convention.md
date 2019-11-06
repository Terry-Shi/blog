## 代码规约和最佳实践
### 命名风格：
#### 一般命名风格
- 命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束。  
例如以下为错误例子：
_name/__name/$Object/name_/name$/Object$  

####  不要拼音和英文单词混用：  
例如：打折价格 
错误的名字 daZhePrices  
正确的名字 discountPrices  

#### 类命名  
类名使用 UpperCamelCase风格，首字母大写，一般遵从驼峰形式，但有一些缩写的情形可以例外：BO/DTO/JWT/RSA...  
类命名例子
- 正例：MarcoPolo/UserDO/XmlService/TcpUdpDeal/TarPromotion
- 反例：macroPolo/UserDo/XMLService/TCPUDPD/TAPromotion

#### method名，变量名使用 LowerCamelCase风格

#### 常量名全部大写，用下划线分割单词 

#### 工具类命名：  
例如 StringUtils、FileUtils、IOUtils。 格式为{操作对象}+Utils

#### 抽象类的命名
推荐抽象类命名使用 Abstratc 或 Base 开头。
例如JDK中的抽象类 java.util.AbstractList。

#### 常量定义
避免magic number魔法值的使用
`public int oneWeekTable[] = new int[7];` //这样定义错误；这个 7 究竟代表什么？
正确的定义方式是这样的：
`public static final int WEEK_TABLE_LENGTH = 7;` //这样定义正确，通过使用完整英语单词的常量名明确定义
`public int oneWeekTable[] = new int[WEEK_TABLE_LENGTH ];`

#### 代码格式
大括号的使用约定：
如果是大括号为空，则简洁地写成{}即可，不需要换行；如果是非空代码块则：

- 左大括号前不换行
- 左大括号后换行
- 右大括号前换行
- 右大括号后还有 else 等代码则不换行表示终止的右大括号后必须换行
- if / for / while 后⾯面都有空格， { 前⾯面要有空格

####   单行字符数限制  
这个并不存在唯一的标准答案，就像Martin Fowler形容小的项目组叫Two Pizza Team (the whole team can be fed by two pizzas)，一行代码的最大长度应该就是不需拖动滚动条就可以一次看到完整一行字符。按照现代笔记本的屏幕情况120个字符是个常见的标准。 eclipse中设置最大字符数后ctrl+shift+f会自动帮助换行。

####   单个方法行数限制  
这个的限制比较特别，要求方法不翻页就完全能看完比较困难，一般是可接受翻屏一次，即100行是个比较合理的数据。
但有些特例，比如数据库表对应的entity类不受此限制，因为数据库字段可能很多。

####   使用空格而不是tab来缩进

####   注释规约
类和public方法强制必须有注释。
方法内部单行注释，在被注释语句上方另起一行，使用//注释。方法内部多行注释使用/**/注释，注意与代码对照。

### 代码处理:
####  1. 单例模式需要保证线程安全
为了达到线程安全，又能提高代码执行效率，我们这里可以采用 DCL双检查锁机制（Double Check Locking） 的双检查锁机制来完成。
```java
public class MySingleton { 
    //使用 volatile 关键字保其可见性 
    private volatile static MySingleton instance = null; 
    
    private MySingleton(){} 

    public static MySingleton getInstance() { 
        try { 
            if (instance == null) {
                synchronized (MySingleton.class) { 
                    if (instance == null) {//二次检查  
                        instance = new MySingleton(); 
                    } 
               } 
            } 
        } catch (InterruptedException e) { 
            // log output
        } 
        return instance; 
    } 
}
```

####  2. Switch 语句的使用
一个 switch 块内，每个 case 要么通过 break/return 等来终止，要么注释说明程序将继续执行到哪一个 case 为止；在一个 switch 块内，都必须包含一个 default 语句并且放在最后，即使它什么代码也没有。
```java
switch(condition) {
    case ABC:
        statements;
        /*程序继续执行直到 DEF 分支*/
    case DEF:
        statements;
        break;
    case XYZ:
        statements;
        break;
    default:
        statements;
        break;
}
```

####  3. 异常处理  
不要捕获运行时异常 RuntimeException。
不要在 finally 块中使用 return。这样会覆盖try语句块中的return值。


####  5.  统一出错时的 RESTful response body

```json
{
  "code": 8090003,
  "message": "the token is expired",
  "payload": {}
}
```

```json
{
  "code": 8090004,
  "message": "Internal server error",
  "payload": {
    "root_cause": "add client failed: User already exists!; nested exception is java.lang.Exception: User already exists!"
  }
}
```

#### 谨慎使用类的数据成员
一方面多线程读写时，要注意线程安全， 另一方面要避免把类的数据成员当成方法之间的输入输出参数。

#### 日期，时间操作
从 JDK8开始 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar， DateTimeFormatter 代替 SimpleDateFormat 。

code sample:

```java
Instant expiredTime = Instant.now().plus(tokenExpireTime, ChronoUnit.MINUTES);
// 如需转换回Date型，可使用 Date.from(expiredTime)
```
