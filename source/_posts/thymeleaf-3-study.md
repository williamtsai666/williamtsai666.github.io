---
title: Thymeleaf3 学习 
date: 2017-11-28 16:43:57
tags:
---

### 1.介绍

Thymeleaf (麝香草叶子), **/ˈtaɪmˌlɪːf/** 是一个服务端java模板引擎框架,它能够处理多种数据格式，包括HTML, XML, JavaScript, CSS以及普通文本。

### 2.简单的示例

```html
<!DOCTYPE html>
<!--声明 th 名称空间-->
<html xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Good Thymes Virtual Grocery</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="stylesheet" type="text/css" media="all" 
          href="../../css/gtvg.css" th:href="@{/css/gtvg.css}" />
  </head>
  <body>
    <!-- 引用文本-->
    <p th:text="#{home.welcome}">Welcome to our grocery store!</p>
  </body>
</html>
```

Thymeleaf使用`th:*` 给现有HTML标签增加属性,因此直接打开模板也能预览效果,很方便 。使用时首先需要像在jsp中一样，在`html`标签内声明th名称空间，然后就可以使用了。

#### 3.Standard Expression Syntax(标准表达式语法)

- Variable Expressions(变量表达式): `${...}`
- Selection Variable Expressions(选择变量表达式): `*{...}`
- Message Expressions(消息表达式): `#{...}`
- Link URL Expressions(链接url表达式): `@{...}`
- Fragment Expressions(代码片段表达式): `~{...}`

##### (1).Messages (信息)

该表达书主要为了实现i18n国际化，需要将多语言的文本放在`/WEB-INF/templates`目录下，如下

- `/WEB-INF/templates/home_en.properties` 英文的翻译.
- `/WEB-INF/templates/home.properties` 默认的文本

```html
<!--常用使用方法-->
<p th:text="#{home.welcome}">Hello World</p> 

<!--unescaped text 不转义文本-->
<p th:utext="#{home.welcome}">
```

也可以传参给文本

```properties
#home.properties文件
home.welcome=hello, {0}!
```

```html
<!-- Thymeleaf解析后会将p标签中的Welcome User!替换为th:text指定的文本 -->
<p th:text="#{home.welcome(${session.user.name})}">
  Welcome User!
</p>
<!--消息的key也可以通过动态去获取  -->
<p th:utext="#{${welcomeMsgKey}(${session.user.name})}">
  Welcome User!
</p>
```

##### (2).Variables(变量)

Thymeleaf的 ${...}是OGNL (Object-Graph Navigation Language) expressions 语法格式，跟jsp中使用方法差不多,使用示例

```htm
/*
* Access to properties using the point (.). Equivalent to calling property getters.
*/
${person.father.name}

/*
* Access to properties can also be made by using brackets ([]) and writing 
* the name of the property as a variable or between single quotes.
*/
${person['father']['name']}

/*
* If the object is a map, both dot and bracket syntax will be equivalent to 
* executing a call on its get(...) method.
*/
${countriesByCode.ES}
${personsByName['Stephen Zucchini'].age}

/*
* Indexed access to arrays or collections is also performed with brackets, 
* writing the index without quotes.
*/
${personsArray[0].name}

/*
* Methods can be called, even with arguments.
*/
${person.createCompleteName()}
${person.createCompleteNameWithSeparator('-')}
```

######  表达式基本对象(Expression Basic Objects)，#开始

- `#ctx`: the context object. 

- `#vars:` the context variables.

- `#locale`: the context locale.

- `#request`: (only in Web Contexts) the `HttpServletRequest` object.

- `#response`: (only in Web Contexts) the `HttpServletResponse` object.

- `#session`: (only in Web Contexts) the `HttpSession` object.

- `#servletContext`: (only in Web Contexts) the `ServletContext` object.

####### 表达式工具对象

- `#execInfo`: information about the template being processed. 
- `#messages`: methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
- `#uris`: methods for escaping parts of URLs/URIs
- `#conversions`: methods for executing the configured *conversion service* (if any).
- `#dates`: methods for `java.util.Date` objects: formatting, component extraction, etc.
- `#calendars`: analogous to `#dates`, but for `java.util.Calendar` objects.
- `#numbers`: methods for formatting numeric objects.
- `#strings`: methods for `String` objects: contains, startsWith, prepending/appending, etc.
- `#objects`: methods for objects in general.
- `#bools`: methods for boolean evaluation.
- `#arrays`: methods for arrays.
- `#lists`: methods for lists.
- `#sets`: methods for sets.
- `#maps`: methods for maps.
- `#aggregates`: methods for creating aggregates on arrays or collections.
- `#ids`: methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

```html
<!--表达式工具对象的使用-->
<p>
  Today is: <span th:text="${#calendars.format(today,'dd MMMM yyyy')}">13 May 2011</span>
</p>
```

#### (3).Expressions on selections (asterisk syntax)(选择表达式 - 星号语法)

```html
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
  </div>

<!-- 等价于 -->
<div>
  <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
</div>

<!-- 还可以混合使用-->
<div th:object="${session.user}">
  <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
<!-- #object 引用表达式的对象 -->
<div th:object="${session.user}">
  <p>Name: <span th:text="${#object.firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```

`th:object`定义选择的对象,在标签内使用`*{...}`来取出对象中相应的数据，如果没有选择的对象直接使用`*{...}`等价于`#{...}`

#### (4).Link URLs(链接URL)

```html
<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" 
  th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

####  (5).Literals(文本)

```html
<!-- 基本的使用-->
<p>
  Now you are looking at a <span th:text="'working web application'">template file</span>.
</p>

<p>The year is <span th:text="2013">1492</span>.</p> 

<!-- ==false 需要写在{}外面才交由thymeleaf处理，写在里面时交由 OGNL/SpringEL处理 -->
<div th:if="${user.isAdmin()} == false"> </div>
<div th:if="${user.isAdmin() == false}"></div>
<!--判空-->
<div th:if="${variable.something} == null">
<!-- 文本链接 -->
<span th:text="'The name of the user is ' + ${user.name}">
  
  
  <!-- 文本替换,(免去+号链接字符串),需要写在 || 之间
        仅变量，信息表达式可以使用${...}, *{...}, #{...}
-->
  <span th:text="|Welcome to our application, ${user.name}!|">
    
    
```

#### (6).Arithmetic operations(算术操作)

```html
<!-- 也可以使用文本 div (/), mod (%).-->
<div th:with="isEven=(${prodStat.count} % 2 == 0)">
```

#### (7).Comparators and Equality(比较)

html起止标签的缘故，大于，小于等需要使用转义

`gt` (`>`), `lt` (`<`), `ge` (`>=`), `le` (`<=`), `not` (`!`) `eq` (`==`), `neq`/`ne` (`!=`).

```html
<div th:if="${prodStat.count} &gt; 1">
<span th:text="'Execution mode is ' + ( (${execMode} == 'dev')? 'Development' : 'Production')" 
```

####  (8).Conditional expressions(条件表达式)

```html
<!--C中的三元运算符 -->
<tr th:class="${row.even}? 'even' : 'odd'">
</tr>
<!-- 省略-->
<tr th:class="${row.even}? 'alt'">
  ...
</tr>
```

#### (9). Default expressions (Elvis operator) 默认表达式 

```html
<div th:object="${session.user}">
  <p>Age: <span th:text="*{age}?: '(no age specified)'">27</span>.</p>
</div>
<!-- 等价于,省略表达式为true时的语句 -->
<p>Age: <span th:text="*{age != null}? *{age} : '(no age specified)'">27</span>.</p>
```

#### (10). The No-Operation token(空操作符)

空操作符( No-Operation token)使用下划线表示`_`，它允许原生页面定义的文本为默认值,便于模板的设计

```html
<span th:text="${user.name} ?: 'no user authenticated'">...</span>
<span th:text="${user.name} ?: _">no user authenticated</span>
```

#### (11). Data Conversion / Formatting (数据转换/格式化)

Thymeleaf 定义了双花括号语法用于变量`${...}`和选择表达式`*{...}`

```html
<td th:text="${{user.lastAccessDate}}">...</td>
```

###  4.Iteration(遍历)

```html
<table>
  <tr>
    <th>NAME</th>
    <th>PRICE</th>
    <th>IN STOCK</th>
  </tr>
  <tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
    <td th:text="${prod.name}">Onions</td>
    <td th:text="${prod.price}">2.41</td>
    <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
  </tr>
</table>
```

###### th:each说明

`prod`保存每次遍历的对象，`iterStat`保存遍历的相关信息

- The current *iteration index*, starting with 0. This is the `index` property. 遍历索引，从0开始
- The current *iteration index*, starting with 1. This is the `count` property. 遍历计数，从1开始
- The total amount of elements in the iterated variable. This is the `size` property.  被遍历对象的大小 
- The *iter variable* for each iteration. This is the `current` property. 当前遍历的对象
- Whether the current iteration is even or odd. These are the `even/odd` boolean properties. 奇偶布尔值
- Whether the current iteration is the first one. This is the `first` boolean property. 是否是遍历第一个
- Whether the current iteration is the last one. This is the `last` boolean property. 是否是遍历最后一个

###### 被遍历的对象类型

- Any object implementing `java.util.Iterable`
- Any object implementing `java.util.Enumeration`.
- Any object implementing `java.util.Iterator`, whose values will be used as they are returned by the iterator, without the need to cache all values in memory.
- Any object implementing `java.util.Map`. When iterating maps, iter variables will be of class `java.util.Map.Entry`.
- Any array.
- Any other object will be treated as if it were a single-valued list containing the object itself.

### 5.Conditional Evaluation(条件表达式)

#### if和unless语句

```html
<!-- 如果if表达式成立则显示该标签-->
<a href="comments.html"
  th:href="@{/product/comments(prodId=${prod.id})}" 
  th:if="${not #lists.isEmpty(prod.comments)}">view</a>

<!--还可以使用相反的表达式unless -->
<a href="comments.html"
  th:href="@{/comments(prodId=${prod.id})}" 
  th:unless="${#lists.isEmpty(prod.comments)}">view</a>
```

###### if遵循的规则

- If value is not null:
  - If value is a boolean and is `true`.
  - If value is a number and is non-zero
  - If value is a character and is non-zero
  - If value is a String and is not “false”, “off” or “no”
  - If value is not a boolean, a number, a character or a String.
- (If value is null, th:if will evaluate to false).
#### switch/case语句

```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="#{roles.manager}">User is a manager</p>
  <p th:case="*">User is some other thing</p> <!--默认值-->
</div>
```

### 6.Template Layout (模板布局)

使用`th:fragment`定义需要复用的代码片段，`th:insert`或`th:replac`引用片段

`~{templatename::selector}` 引用片段 

`~{templatename}`引用整个文件 

`~{::selector}`或`~{this::selector}`引用自身的代码片段 

```html
<!--定义代码片段 -->
<html xmlns:th="http://www.thymeleaf.org">
  <body>
    <div th:fragment="copy">
      &copy; 2011 The Good Thymes Virtual Grocery
    </div>
  </body>
</html>

<!-- 引用代码片 -->
<body>
  <div th:insert="footer :: copy"></div>
</body>

<!-- 根据条件引用-->
<div th:insert="footer :: (${user.isAdmin}? #{footer.admin} : #{footer.normaluser})"></div>


<!-- 根据id引用 css选择器语法类似-->
<div id="copy-section">
  &copy; 2011 The Good Thymes Virtual Grocery
</div>
<div th:insert="~{footer :: #copy-section}"></div>
```

#### `th:insert` , `th:replace` ,`th:include`

- `th:insert` is the simplest: it will simply insert the specified fragment as the body of its host tag. (在定义的标签体内引用代码片段)

- `th:replace` actually *replaces* its host tag with the specified fragment.(将定义标签替换为引用的代码片段)

- `th:include` is similar to `th:insert`, but instead of inserting the fragment it only inserts the *contents* of this fragment.(Thymeleaf 3.0后不推荐使用)

  #### 参数化方式引用代码片段(Parameterizable fragment signatures)

  ```html
  <!--定义参数化代码片段-->
  <div th:fragment="frag (onevar,twovar)">
      <p th:text="${onevar} + ' - ' + ${twovar}">...</p>
  </div>
  <!--使用 使用参数名时顺序不重要 -->
  <div th:replace="::frag (${value1},${value2})">...</div>
  <div th:replace="::frag (onevar=${value1},twovar=${value2})">...</div>
  <div th:replace="::frag (twovar=${value2},onevar=${value1})">...</div>
  ```




### 7.Local Variables(局部变量)

```html
<!-- 使用th:with定义,可以再标签体中使用-->
<div th:with="firstPer=${persons[0]}">
  <p>
    The name of the first person is <span th:text="${firstPer.name}">Julius Caesar</span>.
  </p>
</div>
```




### 8.参考资料 

1.Thymleaf Document(http://www.thymeleaf.org/documentation.html)

2.属性优先级(http://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#attribute-precedence)

3.可用属性(http://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#setting-value-to-specific-attributes)
