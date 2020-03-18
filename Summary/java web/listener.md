[TOC]

# 1. 什么是监听器？

监听器是一个<u>**实现特定接口的普通java程序**</u>，监听另一个java对象的方法调用或属性改变，当监听对象发生上述事件后，监听器的某个方法立即执行。



事件监听三要素：**<u>事件源、事件对象、事件监听器</u>**

# 2. java web中有哪些监听器？

三大域：<u>**ServletContext、HttpSession、ServletRequest**</u>



<u>**I. 监听域对象的创建/销毁**</u>

1.ServletContextListener，监听服务器的创建/销毁

```java
public interface ServletContextListener extends EventListener{
    // 当ServletContext被创建时调用
    default public void contextInitialized(ServletContextEvent sce) {}
    // 当ServletContext被销毁时调用
    default public void contextDestroyed(ServletContextEvent sce) {}
}
```

demo

```java
@WebListener
public class MyServletContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("ServletContext..init..");
        sce.getServletContext().setAttribute("name","me");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("ServletContext...destroy...");
        System.out.println(sce.getServletContext().getAttribute("name"));
    }
}
```

2.HttpSessionListener，监听会话的创建/销毁

```java
public interface HttpSessionListener extends EventListener{
    default public void sessionCreated(HttpSessionEvent se) {}
    default public void sessionDestroyed(HttpSessionEvent se) {}
}
```

3.ServletRequestListener，监听请求的创建/销毁

```java
public interface ServletRequestListener extends EventListener{
    default public void requestDestroyed(ServletRequestEvent sre) {}
    default public void requestInitialized(ServletRequestEvent sre) {}
}
```

<u>**II. 监听域对象的属性变更**</u>

```java
public interface ServletContextAttributeListener extends EventListener {
    default public void attributeAdded(ServletContextAttributeEvent event) {}
    default public void attributeRemoved(ServletContextAttributeEvent event) {}
    default public void attributeReplaced(ServletContextAttributeEvent event) {}
}
```

```java
public interface HttpSessionAttributeListener extends EventListener{
    default public void attributeAdded(HttpSessionBindingEvent event) {}
    default public void attributeRemoved(HttpSessionBindingEvent event) {}
    default public void attributeReplaced(HttpSessionBindingEvent event) {}
}
```

```java
public interface ServletRequestAttributeListener extends EventListener{
    default public void attributeAdded(ServletRequestAttributeEvent srae) {}
    default public void attributeRemoved(ServletRequestAttributeEvent srae) {}
    default public void attributeReplaced(ServletRequestAttributeEvent srae) {}
}
```

<u>**III. 感知Session绑定的事件监听器**</u>

① HttpSessionBindingListener

```java
// 实现该接口的类，可以在对象被session绑定、解绑时触发如下方法
public interface HttpSessionBindingListener extends EventListener {
	default public void valueBound(HttpSessionBindingEvent event) {}
    default public void valueUnbound(HttpSessionBindingEvent event) {}
}
```

demo

```java
// 实现类不需要注册到web.xml
public class MyHttpSessionBindingListener implements HttpSessionBindingListener {
    @Override
    public void valueBound(HttpSessionBindingEvent event) {
        System.out.println("bound-----");
    }

    @Override
    public void valueUnbound(HttpSessionBindingEvent event) {
        System.out.println("unbound-----");
    }
}

// 当session绑定该类时会触发
HttpSession session=req.getSession();
session.setAttribute("o",new MyHttpSessionBindingListener());
session.removeAttribute("o");
```

② HttpSessionActivationListener

```java
public interface HttpSessionActivationListener extends EventListener {
    // 将对象与session钝化到磁盘
	default public void sessionWillPassivate(HttpSessionEvent se) {}
    // 将对象与session从磁盘活化到内存
    default public void sessionDidActivate(HttpSessionEvent se) {}
}
```

demo

修改tomcat的context.xml实现自动序列化到硬盘

```xml
<!--
session的attribute空闲1分钟后，就序列化到C:\Users\18342\.IntelliJIdea2019.1\system\tomcat\Unnamed_webTest\work\Catalina\localhost\ROOT\me-->
<Manager className="org.apache.catalina.session.PersistentManager" maxIdleSwap="1">
	<Store className="org.apache.catalina.session.FileStore" directory="me"/>
</Manager>
```

定义要序列化的类

```java
import javax.servlet.http.HttpSessionActivationListener;
import javax.servlet.http.HttpSessionEvent;
import java.io.Serializable;

/**
 * @ Author: xin
 * @ Date: 2019/6/4 14:22
 */
public class User implements HttpSessionActivationListener, Serializable {

    private static final long serialVersionUID = 42L;

    private String name;
    private String password;

    public User() {}

    public User(String name, String password) {
        this.name = name;
        this.password = password;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public void sessionWillPassivate(HttpSessionEvent se) {
        System.out.println(name+password+"与session"+se.getSession().getId()+"被序列化到硬盘了");
    }

    @Override
    public void sessionDidActivate(HttpSessionEvent se) {
        System.out.println(name+password+"与session"+se.getSession().getId()+"反序列化到内存了");
    }
}
```

当执行req.getSession().setAttribute("user",new User(name,password))后，1分钟未使用则保存为硬盘session文件，当再次使用getAttribute时，session文件读取到内存中。



