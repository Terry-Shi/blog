### Spring Boot的优点
* 约定优于配置，减少XML配置量
* 快速搭建APP
* 易于改变默认设置
* 提供大量便利工具 (e.g. embedded servers, security, metrics, health checks, externalized configuration).

### 从需求出发，需要Web framework提供什么功能？
* 客户端请求的URL与服务端处理类和函数的映射关系 - ModelAttribute("")
* HTML Form数据或JSON与JavaBean之间转换 - @Valid BindingResult
* HTTP response的映射 - a）目的地 ViewName
                       b）输出数据
* Validator输入数据验证 - @Valid BindingResult
* 出错处理
* 数据库存取
* security／权限管理
* Static Resources 的管理such as css/js/jpg...
   例如Spring 4.1提供升级后避免客户端旧js干扰的机制
* 提供监控数据
* 简单日志设置
* 简便的UT功能

### 不希望出现的情况:引入的新complexity超过了原有的complexity
* 学习曲线陡峭
* 大量用不到的功能
* 未能屏蔽实现细节

### 参考资料
- http://projects.spring.io/spring-boot
- MVC工程，RestFul工程例子 http://kielczewski.eu/2014/04/developing-restful-web-service-with-spring-boot/