---
title: Spring为每个请求添加请求头
description:  Spring mvc 通过filter为每个请求添加请求头
date: 2018-01-30 21:43:00
tags: 
    - Spring  
categories:
    - Spring
---

# Spring为每个请求添加请求头
## 需求描述
部分请求需要添加token请求头,此时有两个方法实现该需求
1. 请求获取token,然后在前台加上token,ps:可以保存在cookie
2. 通过filter添加

每个方法都可以实现,根据自己需求实现,我需求只有几个请求需要,所以采用第二种,代码如下所示

```java
public class CustomRequestWrapper extends HttpServletRequestWrapper {

    private Map<String,String> headerMap = new HashMap<>();

    public void addHeader(String name,String value){
        headerMap.put(name,value);
    }

    public CustomRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    @Override
    public String getHeader(String name) {
        String headerValue = super.getHeader(name);
        if (headerMap.containsKey(name)) {
            headerValue = headerMap.get(name);
        }
        return headerValue;
    }

    @Override
    public Enumeration getHeaders(String name) {
        List<String> names = Collections.list(super.getHeaders(name));
        if (headerMap.containsKey(name)) {
            names.add(headerMap.get(name));
        }
        return Collections.enumeration(names);
    }

    @Override
    public Enumeration getHeaderNames() {
        List<String> names = Collections.list(super.getHeaderNames());
        for (String name : headerMap.keySet()) {
            names.add(name);
        }
        return Collections.enumeration(names);
    }
}
```


```java
public class CustomFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        CustomRequestWrapper customRequestWrapper = new CustomRequestWrapper(request);
        customRequestWrapper.addHeader("token","123");
        filterChain.doFilter(customRequestWrapper,servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```

```xml
<filter>
    <filter-name>customFilter</filter-name>
    <filter-class>com.*.CustomFilter</filter-class>
 </filter>
<filter-mapping>
    <filter-name>customFilter</filter-name>
    <url-pattern>/download/*</url-pattern>
</filter-mapping>
```