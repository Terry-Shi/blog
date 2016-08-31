### Thymeleaf 介绍

### Thymeleaf 是什么

Thymeleaf是一个开源的Java模板引擎库。

Thymeleaf与其他模板引擎的最大优势，其模板文件本身也是一个格式良好的XML/HTML文件，并且可以直接被浏览器打开。改变了在传统模板引擎下前端设计人员和后端开发人员的协作方式，能有效的提高工作效率。

### 标准表达式语法 Standard Expression Syntax

- 变量表达式  ${……} Variable Expressions:

```HTML
<input type="text" name="userName" value="James Carrot" th:value="${user.name}" />
```

​    上述代码为引用user对象的name属性值。实际上会执行((User)ctx.getVariables().get("user").getName();

- 选择变量表达式/星号表达式 *{……} Selection Variable Expressions: 

```HTML
<div th:object="${session.user}">                                                                       
     <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>    
</div>
```

​    选择表达式一般跟在th:object后，直接取object中的属性。

上面这段模版等价于

```html
<div>
    <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>    
</div>
```

**if no object selection has been performed, dollar and asterisk syntaxes are exactly equivalent.**

```html
<div>
  <p>Name: <span th:text="*{session.user.name}">Sebastian</span>.</p>
  <p>Surname: <span th:text="*{session.user.surname}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{session.user.nationality}">Saturn</span>.</p>
</div>
```


- 消息表达式/支持国际化  #{……}  Message Expressions: 

```HTML
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
<!-- 带参数的情况,实际上执行((User) ctx.getVariables().get("session").get("user")).getName();
 -->
<p th:utext="#{home.welcome(${session.user.name})}">
  Welcome to our grocery store, Sebastian Pepper!
</p>
```

 　　调用国际化的welcome语句,国际化资源文件如下

```properties
resource_en_US.properties：
home.welcome=Welcome to here！

resource_zh_CN.properties：
home.welcome=欢迎您的到来！
```

- URL表达式  @{……}   Link URL Expressions:

```HTML
<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" 
   th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

          @{……}支持绝对路径和相对路径。其中相对路径又支持跨上下文调用url和协议的引用（//code.jquery.com/jquery-2.0.3.min.js）。

　　　当URL为后台传出的参数时，代码如下

```HTML
<img src="../../static/assets/images/qr-code.jpg" th:src="@{${path}}" alt="二维码" />
```

更复杂的情况

```html
<a th:href="@{${url}(orderId=${o.id})}">view</a>
<a th:href="@{'/details/'+${user.login}(orderId=${o.id})}">view</a>
```

#### 常用的th标签

- 简单数据转换（数字，日期）

```html
 　　<dt>价格</dt>
  　 <dd th:text="${#numbers.formatDecimal(product.price, 1, 2)}">180</dd>
　　 <dt>进货日期</dt>
　　 <dd th:text="${#dates.format(product.availableFrom, 'yyyy-MM-dd')}">2014-12-01</dd>
```

- 字符串拼接

```html
<dd th:text="${'$'+product.price}">235</dd>
```

- 转义和非转义文本

​    当后台传出的数据为  `This is an &lt;em&gt;HTML&lt;/em&gt; text. &lt;b&gt;Enjoy yourself!&lt;/b&gt;`  时，若页面代码如下则出现两种不同的结果

```html
<div th:text="${html}">
　　This is an &lt;em&gt;HTML&lt;/em&gt; text. &lt;b&gt;Enjoy yourself!&lt;/b&gt;
</div> 

<div th:utext="${html}">
　　This is an <em>HTML</em> text. <b>Enjoy yourself!</b>
</div>
```

- 表单中

```html
<form th:action="@{/bb}" th:object="${user}" method="post" th:method="post">
    <input type="text" th:field="*{name}"/>
    <input type="text" th:field="*{msg}"/>
    <input type="submit"/>
</form>
```

- 遍历集合中数据

```html
//用 th:remove 移除除了第一个外的静态数据，用第一个tr标签进行循环迭代显示。也就是移除了`White table` `Reb table` `Blue table`这三行。
<tbody th:remove="all-but-first">
  //将后台传出的 productList 的集合进行迭代，用product参数接收，通过product访问属性值
  <tr th:each="product:${productList}">
    //用count进行统计，有顺序的显示
    <td th:text="${productStat.count}">1</td> ？？？productStat是哪里定义的？？？
    <td th:text="${product.description}">Red Chair</td>
    <td th:text="${'$' + #numbers.formatDecimal(product.price, 1, 2)}">$123</td>
    <td th:text="${#dates.format(product.availableFrom, 'yyyy-MM-dd')}">2014-12-01</td>
  </tr>
  <tr>
    <td>White table</td>
    <td>$200</td>
    <td>15-Jul-2013</td>
  </tr>
  <tr>
    <td>Reb table</td>
    <td>$200</td>
    <td>15-Jul-2013</td>
  </tr>
  <tr>
    <td>Blue table</td>
    <td>$200</td>
    <td>15-Jul-2013</td>
  </tr>
</tbody>
```

- 条件判断

`<span th:if="${product.price lt 100}" class="offer">Special offer!</span>`
　　不能用"<”，">"等符号，要用"lt"等替代
```html
<!-- 当gender存在时，选择对应的选项；若gender不存在或为null时，取得customer对象的name-->
<td th:switch="${customer.gender?.name()}">
  <img th:case="'MALE'" src="../../../images/male.png" th:src="@{/images/male.png}" alt="Male" />
  <!-- Use "/images/male.png" image -->
  <img th:case="'FEMALE'" src="../../../images/female.png" th:src="@{/images/female.png}" alt="Female" /> 
  <!-- Use "/images/female.png" image -->
  <span th:case="*">Unknown</span>
</td>
```

```html
<!--在页面先显示，然后再在显示的数据基础上进行修改-->
<div class="form-group col-lg-6">
    <label>姓名<span>&nbsp;</span></label>
 　　<!--除非resume对象的name属性值为null，否则就用name的值作为placeholder值-->
    <input type="text" class="form-control" th:unless="${resumes.name} eq '' or ${resumes.name} eq null" data-required="true" th:placeholder="${resumes.name}" />
　　 <!--除非resume对象的name属性不为空，否则就定义一个field方便封装对象，并用placeholder提示-->
    <input type="text" th:field="${resume.name}" class="form-control" th:unless="${resumes.name} ne null" data-required="true" th:placeholder="请填写您的真实姓名"  />
</div>

<!-- 增加class="enhanced"当balance大雨10000 -->
<td th:class="${customer.balance gt 10000} ? 'enhanced'" th:text="${customer.balance}">350</td>
```

- 根据后台数据选中select的选项

```html
 <div class="form-group col-lg-6">
  <label >性别<span>&nbsp;Sex:</span></label>
  <select th:field="${resume.gender}" class="form-control" th:switch="${resumes.gender.toString()}" data-required="true">
    <option value="男" th:case="'男'" th:selected="selected" >男</option>
    <option value="女" th:case="'女'" th:selected="selected" >女</option>
    <option value="">请选择</option>
  </select>
 </div>
```



##### Ref:

- 最具体清晰的还是官方文档 http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html
- [thymeleaf 学习笔记-基础篇
- Thymeleaf 模板的使用 http://www.jianshu.com/p/ed9d47f92e37
