Spring MVC 处理静态文件

错误现象：部署项目后程序加载或用浏览器访问时出现类似的警告，2011-01-19 10:52:51,646 WARN [org.springframework.web.servlet.PageNotFound] -<No mapping found for HTTP request with URI [/sandDemo001/images/1.jpg] in DispatcherServlet with name 'spring'>

原因分析：
web.xml下对spring的DispatcherServlet请求url映射的配置，原配置如下：

	 <servlet>
	    <servlet-name>spring</servlet-name>
	    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	    <load-on-startup>1</load-on-startup>
	 </servlet>
	 <servlet-mapping>
	        <servlet-name>spring</servlet-name>
	        <url-pattern>/</url-pattern>
	 </servlet-mapping>

分析原因：<servlet-mapping>的<url-pattern>/</url-pattern>把所有的请求都交给spring去处理了，而所有available的请求url都是在Constroller里使用类似@RequestMapping(value = "/login/{user}", method = RequestMethod.GET)这样的注解配置的，这样的话对js/css/jpg/gif等静态资源的访问就会得不到。


方案一：激活Tomcat的defaultServlet来处理静态文件

		<servlet-mapping>   
		    <servlet-name>default</servlet-name>  
		    <url-pattern>*.jpg</url-pattern>     
		</servlet-mapping>    
		<servlet-mapping>       
		    <servlet-name>default</servlet-name>    
		    <url-pattern>*.js</url-pattern>    
		</servlet-mapping>    
		<servlet-mapping>        
		    <servlet-name>default</servlet-name>       
		    <url-pattern>*.css</url-pattern>      
		</servlet-mapping>

要配置多个，每种文件配置一个   
要写在DispatcherServlet的前面， 让 defaultServlet先拦截请求，这样请求就不会进入Spring了。

Tomcat, Jetty, JBoss, and GlassFish 自带的默认Servlet的名字 -- "default"

Google App Engine 自带的 默认Servlet的名字 -- "_ah_default"

Resin 自带的 默认Servlet的名字 -- "resin-file"

WebLogic 自带的 默认Servlet的名字  -- "FileServlet"

WebSphere  自带的 默认Servlet的名字 -- "SimpleFileServlet" 


方案2
使用<mvc:default-servlet-handler/>
会把"/**" url,注册到SimpleUrlHandlerMapping的urlMap中,把对静态资源的访问由HandlerMapping转到org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler处理并返回.
DefaultServletHttpRequestHandler使用就是各个Servlet容器自己的默认Servlet.




#### 补充说明：多个HandlerMapping的执行顺序问题：
DefaultAnnotationHandlerMapping的order属性值是：0

<mvc:resources/ >自动注册的 SimpleUrlHandlerMapping的order属性值是： 2147483646
 
<mvc:default-servlet-handler/>自动注册 的SimpleUrlHandlerMapping 的order属性值是： 2147483647
 
spring会先执行order值比较小的。当访问一个a.jpg图片文件时，先通过 DefaultAnnotationHandlerMapping 来找处理器，一定是找不到的，因为我们没有叫a.jpg的Action。然后再按order值升序找，由于最后一个 SimpleUrlHandlerMapping 是匹配 "/**"的，所以一定会匹配上，就可以响应图片。



参考:

http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-config-static-resources

http://elf8848.iteye.com/blog/875830

http://www.blogjava.net/Steven-bot/archive/2012/08/14/361333.html