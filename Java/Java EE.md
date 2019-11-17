## JSP中获取元素的值：
```java
<% String username=request.getParameter("username"); %>
```
## 重定向：
```java
resp.sendRedirect("/project/...");
```
## 转发:
```java
req.getRequestDispachter("/login/...").forward(request, response);
```
## 合并servlet
1. UserServlet?method=xxx
2.  用反射
 - 新建一个BaseServlet；无需配置；用来继承不直接访问
 - service():
```java
String methodName=req.getParameter("method");
Class clazz=this.getClass();//得到类的结构信息
Method method = clazz.getMethod(methodName,HttpServletRequest.class,HttpServletResponse.class);
method.invoke(this,req,resp);
```
 - ~~Object obj=clazz.newInstance();~~ **Servlet为单实例多线程**

## Cookie:
 - ### Servlet：
 ```java
Cookie cookie1=new Cookie("username", username);
cookie1.setMaxAge(60*60*24*10);//为0即删除
cookie1.setPath("/");
resp.addCookie(cookie1);
```

- ### JSP:
```java
<%
    Cookie[] cookies=request.getCookies();
    String username="";
    String password="";
    String checked="";
    if(cookies!=null) {         //cookies exist
        for (int i = 0; i < cookies.length; i++) {
            String tname = cookies[i].getName();
            if (tname.equals("username")) {
                username = cookies[i].getValue();
                checked="checked";
            }
            if (tname.equals("password")) {
                password = cookies[i].getValue();
            }
        }
    }
%>
```

 - cookie中文乱码解决:
```java
chinesename=URLEncoder.encode(username,"utf-8");
```

   - 解码：
```java
URLDecode.decode(uname,"utf-8");
```

## EL：
使用前提：req.setAttribute()
判断不为空：${not empty list}
##JSTL:（标准标签库）
[**菜鸟JSTL教程**](https://www.runoob.com/jsp/jsp-jstl.html "菜鸟JSTL教程")
 - 配置web.xml
 ```xml
<jsp-config>
        <taglib>
            <taglib-uri>http://java.sun.com/jsp/jstl/core</taglib-uri>
            <taglib-location>/WEB-INF/c.tld</taglib-location>
        </taglib>
       // ...
</jsp-config>
```

- JSP开头：
```java
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
```

- 常用标签：
  - **cout**:
<c:out>标签用来显示一个表达式的结果，与<%= %>作用相似，它们的区别就是<c:out>标签可以直接通过"."操作符来访问属性。
举例来说，如果想要访问customer.address.street，只需要这样写：
```java
<c:out value="customer.address.street">
```
<c:out>标签会自动忽略XML标记字符，所以它们不会被当做标签来处理。
```java
<c:out value="&lt要显示的数据对象（未使用转义字符）&gt" escapeXml="true" default="默认值"></c:out><br/>
<c:out value="&lt要显示的数据对象（使用转义字符）&gt" escapeXml="false" default="默认值"></c:out><br/>
```
结果：
```
&lt要显示的数据对象（未使用转义字符）&gt
<要显示的数据对象（使用转义字符）>
```
 - **cset**：
<c:set>标签用于设置变量值和对象属性。<c:set>标签就是<jsp:setProperty>行为标签的孪生兄弟。这个标签之所以很有用呢，是因为它会计算表达式的值，然后使用计算结果来设置 JavaBean 对象或 java.util.Map 对象的值。
<c:set var="salary" scope="session" value="${2000*2}"/>
<c:out value="${salary}"/>

 - **cif**：
 ```java
<c:if test="${salary > 2000}">
   <p>我的工资为: <c:out value="${salary}"/><p>
</c:if>
```
 - **cforEach、forTokens**：
这些标签封装了Java中的for，while，do-while循环。相比而言，<c:forEach>标签是更加通用的标签，因为它迭代一个集合中的对象。<c:forTokens>标签通过指定分隔符将字符串分隔为一个数组然后迭代它们。
foreach：
```html
<c:forEach var="i" begin="1" end="5">
   Item <c:out value="${i}"/><p>
</c:forEach>
```
fortokens：
```html
<c:forTokens items="google,runoob,taobao" delims="," var="name">
   <c:out value="${name}"/><p>
</c:forTokens>
```

## 过滤器Filter：（解决相同大量代码）
**Servlet、Filter、Listener为JavaEE三种并列技术**

- 过滤器是请求到达目标资源的预处理程序；
- 同时还是目标资源执行完毕后的后处理程序（响应离开服务器之前进行处理）

可定义多个过滤器，每个过滤器完成一个任务，组成一个过滤器链(chain)，依次进行
步骤：
1. 开发一个过滤器
2. 配置一个过滤器

**过滤器使用的设计模式：职责链模式**
```java
FirstFilter implements Filter
```
Filter中inti()、destroy()只执行一次
doFilter()相当于servlet中的service(),过滤范围内的每次响应请求执行一次
**内容：预处理操作**
调用下一个过滤器（或目标资源）后处理
### 配置web.xml

```<filter-mapping>``` 中 ```<url-pattern>```用通配符，或多个来配置过滤范围
dofilter()中使用resp与req需要强转
**ServletResponse→HttpResponse**
### 不需要过滤的情况：
```java
//获取请求路径
String  url=httpRequest.getRequestURL().toString();
//检查是否在作用范围 大于0说明包含
int x=path.indexof(...)
//下列语句得到的是"method=register"
httpRequest.getQueryString();
```

### 若放行则chain.doFilter(...);

chain的执行顺序：web.xml中```<filter-mapping>```顺序有关
## 监听器Listener：
监听Servlet容器中的事件，事件发生后，激活监听器
三个作用域对象：
1. ServletContext
2. HttpSession
3. ServletRequest

按监听事件划分：
1. 创建销毁
2. 属性的增加删除
3. session
implements 8种接口
