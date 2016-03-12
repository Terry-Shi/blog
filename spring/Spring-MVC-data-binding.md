Spring MVC 数据绑定

处理流程：
![http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/images/mvc.png](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/images/mvc.png)

![Context hierarchy in Spring Web MVC](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/images/mvc-contexts.gif)


数据来源：

- request.getParameter() -> @RequestParam 绑定请求参数 -> @RequestParam("userName") 
- request.getAttribute()
- session.getAttribute()
- URL -> @PathVariable 绑定URL中的变量
- HTTP Head -> @RequestHeader 绑定请求头参数 -> @RequestHeader("Accept-Language") 
- cookie -> @CookieValue 绑定Cookie的值 -> @CookieValue("JSESSIONID")


目标：绑定这些数据到Controller类的具体处理的method。同时也涉及到数据的组装，校验。

原则：convention over configuration

Controller类的处理method的参数可以以多种形式接受数据。

* Form中的path应该怎么设
* 不使用form:form 数据怎么绑定
* form tag中的modelAttribute or commandName 有什么作用？

Request-Handling Methods

- Annotation
- input arguments
- return value type


*  增强安全性 form-binding whitelisting

		@InitBinder("subscriber")
		public void initSubscriberBinder(WebDataBinder binder) {
		    binder.setAllowedFields(new String[] {
		    "firstName", "lastName", "email"
		    });
		}


参考:

[SpringMVC强大的数据绑定（2）——第六章 注解式控制器详解](http://jinnianshilongnian.iteye.com/blog/1705701)

Book: Pro Spring MVC with Web Flow 2012

官方网站 
http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html

Spring MVC 教程,快速入门,深入分析 
http://elf8848.iteye.com/blog/875830

