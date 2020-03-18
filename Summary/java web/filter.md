[TOC]

# 1.什么是Filter？如何使用？

定义：过滤器用于控制所有web资源的访问，包括servlet、静态资源等，可以针对访问前request和访问后response分别做拦截处理。

Filter生命周期

```java
init(FilterConfig filterConfig)		初始化，仅在web服务器启动时执行一次
doFilter(req,resp,filterChain)		过滤
destroy()	销毁Filter，仅执行一次
```

使用，一般继承自HttpFilter即可

```java
@WebFilter(urlPatterns = {"*.do"})
public class LoginFilter extends HttpFilter {

    @Override
    protected void doFilter(HttpServletRequest req, HttpServletResponse res, FilterChain chain) throws IOException, ServletException {
        System.out.println("过滤请求");
        if(req.getSession().getAttribute("user")==null){
            res.sendRedirect("/login.html");
        }
        super.doFilter(req, res, chain);
        System.out.println("过滤响应");
    }
}
```

# 2.Filter链的执行顺序

根据Filter在web.xml中的注册顺序执行过滤。

1.按顺序执行每个过滤器chain.doFilter()之前的代码

2.获取目标资源

3.反序执行chain.doFilter()之后的代码

# 3.使用场景？

场景1：字符集过滤器CharacterEncodingFilter 

```java
doFilter(req,resp,chain){
	req.setCharacterEncoding(charset);
	resp.setCharacterEncoding(charset);
}
```

场景2：自定义Request装饰所有request

```java
// 装饰者模式
/*
 * 1、实现与被增强对象相同的接口 
 * 2、定义一个变量记住被增强对象
 * 3、定义一个构造函数，接收被增强对象
 * 4、覆盖需要增强的方法
 * 5、对于不想增强的方法，直接调用被增强对象（目标对象）的方法
*/
class BodyReaderHttpServletRequestWrapper extends HttpServletRequestWrapper{
	//定义一个变量记住被增强对象(request对象是需要被增强的对象)
	private HttpServletRequest request;
	//定义一个构造函数，接收被增强对象
	public MyCharacterEncodingRequest(HttpServletRequest request) {
		super(request);
		this.request = request;
	}
    ...
}
```

场景3：自定义Response装饰所有response