# 1、Spring的事务传播特性有哪些？


# 2、Spring中过滤器和拦截器有什么区别？
```java

// Filter 是servlet的体系下面的
package javax.servlet;

public interface Filter {
    // 在容器初始化时调用init方法，只会调用一次
    default void init(javax.servlet.FilterConfig filterConfig) throws javax.servlet.ServletException { /* compiled code */ }

    // 具体的逻辑处理
    void doFilter(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse, javax.servlet.FilterChain filterChain) throws java.io.IOException, javax.servlet.ServletException;

    // 容器销毁时，被调用
    default void destroy() { /* compiled code */ }
}

// spring容器中的一员
package org.springframework.web.servlet;

public interface HandlerInterceptor {
    // 请求方法前置拦截，该方法会在Controller处理之前进行调用
    default boolean preHandle(javax.servlet.http.HttpServletRequest request, javax.servlet.http.HttpServletResponse response, java.lang.Object handler) throws java.lang.Exception { /* compiled code */ }
    // preHandle返回结果为true时，在Controller方法执行之后，视图渲染之前被调用
    default void postHandle(javax.servlet.http.HttpServletRequest request, javax.servlet.http.HttpServletResponse response, java.lang.Object handler, @org.springframework.lang.Nullable org.springframework.web.servlet.ModelAndView modelAndView) throws java.lang.Exception { /* compiled code */ }
    // 在preHandle返回ture，并且整个请求结束之后，执行该方法
    default void afterCompletion(javax.servlet.http.HttpServletRequest request, javax.servlet.http.HttpServletResponse response, java.lang.Object handler, @org.springframework.lang.Nullable java.lang.Exception ex) throws java.lang.Exception { /* compiled code */ }
}
```
> 相同点：
> 
> 拦截件和过滤器都能对请求进行一些过滤处理操作，可以将通用的操作封装在其中
> 
> 都能通过Order注解指定执行顺序
> 
> 不同点：
> 
> 过滤器是依托与Servlet容器进行使用，因此只能应用的web项目中，
> 拦截器是Spring中进行定义的，只要使用了Spring都可以使用Spring
> 
> 过滤器是基于函数回调来实现的，拦截器是基于Java反射机制实现
> 
# 3、BeanFactory和FactoryBean的区别？
> 总结：
> 
> BeanFactory是Spring最底层的容器接口，可以根据bean定义的信息，返回对应的实例对象，支持整个sring的生命周期流程。
> 
> FactoryBean就是一个简单的对象工程，实现了此接口的方法在整个bean周期中，可以使用自定义的方式来创建对象，而不需要进行默认的bean流程的创建。
> 
