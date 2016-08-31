### Spring Boot和Thymeleaf集成

----------
#### 表单数据绑定 Form data binding 
##### Client -> Server
HTTP Request(HTML page submit) ---> Controller； 数据绑定到HTTP请求的处理方法的输入参数上,此时数据源可以是表单，也可以是URL中问号后的Key value值对，Cookie，HTTPHeader等等。

##### Case: Form数据绑定到Controller中方法的输入参数， 此时Form中没有th:object。（因为th:object不会影响HTML代码，只在服务器端与数据合并时发挥作用，所以HTML FORM提交时不产生影响。)

Thymeleaf模版

```html
<form method="POST" th:action="@{/requestparam}" >
	<input type="text" name="firstName" id="firstName"  th:value="${firstName}"> </input>
	<input type="text" name="lastName" id="lastName"  th:value="${lastName}"> </input>
	<input type="submit"></input>
</form>
```

浏览器端看到的HTML源代码

```html
<form method="POST" action="/requestparam">
	<input type="text" name="firstName" id="firstName" value="set param firstname"> </input>
	<input type="text" name="lastName" id="lastName" value="set param lastname"> </input>
	<input type="submit" />
</form>
```
Controller源代码:**可以**直接通过HttpServletRequest取到数据。（Java Servlet直接提供的功能）

```java
@RequestMapping(method=RequestMethod.POST)
public String submit(HttpServletRequest request, HttpServletResponse response, HttpSession session) {
   System.out.println(" ****** bind to HttpServletRequest ****** ");
   System.out.println(request.getParameter("firstName"));
   System.out.println(request.getParameter("lastName"));
   String firstName = WebUtils.findParameterValue(request, "firstName");
   String lastName = WebUtils.findParameterValue(request, "lastName");
   System.out.println(firstName);
   System.out.println(lastName);
   return "redirect:/requestparam";
```
   }

输出：

	set param firstname
	set param lastname
	set param firstname
	set param lastname

Controller源代码:通过@RequestParam可以直接绑定到输入参数

```java
@RequestMapping(method=RequestMethod.POST)
public String submit(
        @RequestParam("firstName") String fn, 
        @RequestParam("lastName") String ln) {
    System.out.println(" ****** bind to Variable ****** ");
    System.out.println(fn);
    System.out.println(ln);
	return "RequestParam"; // "RequestParam" is ViewName
}
```

Controller源代码:可以直接绑定到POJO

```java
@RequestMapping(method=RequestMethod.POST)
public String submit(Contact contact) {
      System.out.println(" ****** bind to POJO ****** " + contact);
    System.out.println(contact.getFirstName());
    System.out.println(contact.getLastName());
    return "RequestParam";
}
```

Controller源代码:**无法**直接绑定到Map

```java
@RequestMapping(method=RequestMethod.POST)
public String submit(Map<String,Object> model) {
   System.out.println(" ****** bind to Map ****** " + model);
   System.out.println(model.get("firstName"));
   System.out.println(model.get("lastName"));
   return "RequestParam";
}
```



Controller源代码:**无法**直接绑定到ModelAndView

```java
@RequestMapping(method=RequestMethod.POST)
public ModelAndView submit(ModelAndView model) {
   System.out.println(" ****** bind to ModelAndView ****** " + model);
   System.out.println(model.getModel().get("firstName"));
   System.out.println(model.getModel().get("lastName"));
   model.setViewName("RequestParam");
```



----------

##### Case: Form数据绑定到Controller中方法的输入参数，且Thymeleaf的Form中使用th:object

Thymeleaf模版

```html
<form method="POST" th:action="@{/requestparam}" th:object="${map}">
  <input type="text" name="firstName" id="firstName"  th:value="${firstName}"> </input>
  <input type="text" name="lastName" id="lastName"  th:value="${lastName}"> </input>
  <input type="submit">submit with th:object </input>
</form>
```

HTML源码 （没有出现th:object相关的内容，表示在form提交时th:object并没有产生任何影响）

```html
<form method="POST" action="/requestparam">
  <input type="text" name="firstName" id="firstName" value="set param firstname"> </input>
  <input type="text" name="lastName" id="lastName" value="set param lastname"> </input>
  <input type="submit">submit with th:object </input>
</form>
```

小结：th:object并没有在Form提交时产生影响，因为不影响生成的HTML代码

- HttpServletRequest，@RequestParam("firstName")，POJO，
  - 可以获得Form中数据 (数据其实并不局限于form，例如 url?key=value也可以)
- Map, Model, ModelMap， ModelAndView
  - 无法获得Form中数据
  - 但是可以通过加标注@ModelAttribute("map")在输入参数前来绑定form中数据到模型中。 后面有具体例子。

#####  Server -> Client 

##### Case： Controller中数据和Thymeleaf模版结合形成Response的内容

controller类处理方法的返回值类型可以是

1. 一个字符串，表示view的name
2. ModelAndView，同时包含数据和view的name
   返回的数据可以**直接**放在Map 或 Model 或 ModelMap 或 ModelAndView中。

Controller的方法中Map可以put值给页面 (有无 th:object="${contact}" 都可以)

```java
@RequestMapping(method=RequestMethod.GET)
public String home(Map<String,Object> model) {
    model.put("firstName", "set param firstname");
    model.put("lastName", "set param lastname");
	return "RequestParam";
}
```

Model, ModelMap, ModelAndView也可以传值(有无 th:object="${contact}" 都可以)

```java
@RequestMapping(method=RequestMethod.GET)
public String home(Model model) {
    model.addAttribute("firstName", "set param firstname in Model");
    model.addAttribute("lastName", "set param lastname in Model");
    return "RequestParam";
}

@RequestMapping(method = RequestMethod.GET)
public ModelAndView home(ModelAndView model) {
    model.getModel().put("firstName", "set param firstname in ModelAndView");
    model.getModel().put("lastName", "set param lastname in ModelAndView");
    model.setViewName("RequestParam");
    return model;
}
```

直接用POJO对象无法传递值。 可以用@ModelAttribute间接传递POJO对象，此时必须有th:object="${contact}"  且要th:value="*{firstName}"  <－星号表达式 
反之：如果模板中有th:object="${contact}"  则context中必须有key为contact的对象 否则th:value="*{firstName}" 会导致null异常 `org.springframework.expression.spel.SpelEvaluationException: EL1007E:(pos 0): Property or field 'firstName' cannot be found on null`

Thymeleaf模版

```html
<form method="POST" th:action="@{/requestparam}" th:object="${contact}">
  <input type="text" name="firstName" id="firstName"  th:value="*{firstName}"> </input>
  <input type="text" name="lastName" id="lastName"  th:value="${lastName}"> </input>
  <input type="submit">submit with th:object </input>
</form>
```

Controller

```java
@RequestMapping(method = RequestMethod.GET)
public String home(@ModelAttribute("contact") Contact contact) { //@ModelAttribute("contact") 会自动把contact对象加入到模型对象中，contact在作为输入参数前已经绑定了Reqeust中的数据。 例如http://localhost:8080/requestparam?firstName=%22haha%22。 会造成system out输出Contact [id=null, firstName="haha", lastName=null, phoneNumber=null, emailAddress=null]

    System.out.println(contact);
    contact.setFirstName("set param firstname in POJO");
    contact.setLastName("set lastname in POJO");
    return "RequestParam";
}
```

或者

```java
@RequestMapping(method = RequestMethod.GET)
public ModelAndView home(ModelAndView model) {
    Contact contact = new Contact();
    contact.setFirstName("set param firstname in ModelAndView.POJO");
    contact.setLastName("set lastname in ModelAndView.POJO");
    model.getModel().put("contact", contact);
    model.setViewName("RequestParam");
    return model;
}
```

* （未验证）ModelMap应该和Mode，ModelAndView一样可以。

* **原理：**
  - Spring MVC通过@RequestMapping把请求引导到具体的处理方法上。
  - Spring MVC一旦发现处理方法有Map或Model，ModelMap类型的输入参数，就会将请求内的隐含模型对象的引用传递给这些输入参数。

* th:value和th:field的区别？ 

  Values for `th:field` attributes must be selection expressions (`*{...}`), which makes sense given the fact that they will be evaluated on the form-backing bean and not on the context variables (or model attributes in Spring MVC jargon).

  ```html
  <input type="text" th:field="*{datePlanted}" />
  等于
  <input type="text" id="datePlanted" name="datePlanted" th:value="*{datePlanted}" />
  ```

* `${}` 和 `*{}` 的区别？`${}` 是变量表达式，`${v1}`表示context中key为v1的值。 `*{}` 是选择变量表达式。 `*{v1}`表示 v1是某个bean的属性，不是单独存在的变量名。

#### 格式转换 TODO

#### validation TODO

### 参考资料
- http://www.thymeleaf.org/doc/tutorials/2.1/thymeleafspring.html
- http://qiita.com/rubytomato@github/items/387d46ea34eb92071065
- http://www.jianshu.com/p/5ac18abc91f0
- 《Spring Boot Cookbook》阅读笔记 
- Spring in action
- Spring 3.x企业应用开发实战