[TOC]

# 1. 什么是servlet?

servlet是位于servlet容器中的组件，常见的servlet容器有tomcat服务器，主要用于处理请求并作出响应。

Servlet接口主要的方法有

```java
init(ServletConfig config) 初始化方法

service(ServletRequest req,ServletResponse res)

destroy() 应用服务器关闭后
```

# 2. 如何使用servlet?

**1.定义HttpServlet类**

​	通常用HttpServlet实现类来处理请求，通过**重写GenericServlet的doGet()和doPost()方法**即可，因为GenericServlet的service()方法已经实现根据不同的request method调用不同处理方法。

**2.在web.xml中添加该servlet**

```xml
<servlet>
	<servlet-name>Test</servlet-name>
    <servlet-class>com.hand.MyServlet</servlet-class>
	<!--可以添加初始化参数，这样即可在init()中获取-->
   	<init-param>
    	<param-name>sys</param-name>
    	<param-value>hello</param-value>
    </init-param>
    <!--默认值为0，即懒加载-->
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
   <servlet-name>Test</servlet-name>
   <url-pattern>/test</url-pattern>
</servlet-mapping>
```

# 3. 什么是ServletContext？

**整个应用只有一个ServletContext**，用于servlet和servlet容器间的通信。

当然我们也可以ServletContext中放入一些**全局共享资源**。

方式1：web.xml中配置

```xml
<context-param>
    <param-name>context</param-name>
    <param-value>contextValue</param-value>
</context-param>
```

方式2：代码中getServletContext().setAttribute();

# 4. web资源的关系有哪三种？

1.请求转发（地址栏url不变，请求在servlet1处理完后转发到servlet2继续处理，可携带Attribute）

```java
req.getRequestDispatcher("/test2").forward(req,resp);
```

2.请求重定向（地址栏url改变，req在servlet1处理完后，生成**<u>新的req</u>**到servlet2）

```java
resp.sendRedirect("/test2");
```

3.请求包含（地址栏url不变，请求在servlet1处理完后，继续由servlet2处理，将2次响应合并后返回）

```java
req.getRequestDispatcher("/test2").include(req,resp);
```

# 5. 四种会话跟踪（解决Http无状态的方法）

会话跟踪就是要维护上下文通信的状态，即：明确当前是哪个用户在进行什么操作。

**1.url重写**

​	将参数放置在地址栏上

**2.隐藏表单域**

​	原理与url重写类似，只是不在地址栏显示

**3.cookie**

定义：servlet将cookie存储于客户端，客户端收到cookie后保存到本地，之后每次请求servlet都会带上cookie。

操作：服务端的每个servlet都可以通过**<u>request.getCookie()</u>** 获得cookie数组，通过**<u>response.addCookie(cookie)</u>**添加新的key-value（自行设置cookie有效期，默认-1，即会话结束即清除）

应用场景：

​	① 用户登录后，服务端创建创建含用户名的cookie发送给浏览器，浏览器保存到本地，之后再请求servlet就不需要登录了。

​	② 同理，用户购物车，可以保存到本地cookie。

**4.session**

定义：servlet从request获取session，也就是检查cookie里有没有JSESSIONID，如果没有的话则创建并保存到客户端cookie，之后客户端每次请求servlet只需要带上cookie中的JSESSIONID。

操作：

```java
// 从请求中获取session，没有的话就自动创建，在servlet响应后保存JSESSIONID到客户端cookie
HttpSession session=req.getSession();
System.out.println("test6:"+session.getId());
// session中保存会话的所有信息
session.setAttribute("o","ooo");
session.getAttribute("o");
```

应用场景：

​	① 用户登录后，服务端创建session，在session中保存用户信息，将JSESSIONID发送给浏览器，浏览器保存JSESSIONID到本地，之后再请求servlet就不需要登录了。

​	② 同理，用户购物车，可以保存在servlet的session attribute中。



注意：session和cookie可以设置不同的有效期。由于cookie默认是-1，关闭浏览器即清除cookie，导致JSESSIONID也被清除，因此看起来session也无效了，实际上session会等到有效期（默认30min）结束。

