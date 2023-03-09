---
title: 初识Spring Security（一）
abbrlink: 34298
date: 2022-07-02 17:15:00
tags: [Java,Spring Security]
categories: Java
description: Spring Boot集成Spring Security，配置文件详解。

---

## 相关依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

## Spring Securit配制文件SecurityConfig

- 处理访问无权限是返回结果  
  
  ```Java
  @Component
  public class RestfulAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request,
                       HttpServletResponse response,
                       AccessDeniedException e) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        //Jackson核心对象
        ObjectMapper mapper = new ObjectMapper();
        response.getWriter().println(mapper.writeValueAsString(Result.forbidden("所请求资源，没有权限访问！")));
        response.getWriter().flush();
    }
  }
  ```

- 处理Token失效或未登录是返回结果  
  
  ```Java
  @Component
  public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        //Jackson核心对象
        ObjectMapper mapper = new ObjectMapper();
        response.getWriter().println(mapper.writeValueAsString(Result.unauthorized("未登录或者token失效！")));
        response.getWriter().flush();
    }
  }
  ```

- 配制文件主要内容  
    SecurityConfig接管Spring Security的配置，必须要继承WebSecurityConfigurerAdapter重写configure方法。并且通常添加@EnableWebSecurity注解开启方法过滤注解
  
  ```Java
  @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity
                .csrf().disable() //关闭CSRF
                .sessionManagement()// 基于token，所以不需要session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 允许对于网站静态资源的无授权访问
                .antMatchers(HttpMethod.GET,
                        "/",
                        "/*.html",
                        "/favicon.ico",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/swagger-resources/**",
                        "/v2/api-docs/**"
                )
                .permitAll()
                // 对登录注册要允许匿名访问
                .antMatchers("/system/user/login", "/system/user/register")
                .permitAll()
                //跨域请求会先进行一次options请求
                .antMatchers(HttpMethod.OPTIONS)
                .permitAll()
                //允许访问druid监控页面，由于CSRF跨站点请求伪造(Cross—Site Request Forgery)的原因，会进不去druid监控页面
                .antMatchers("/druid/*")
                .permitAll()
                .anyRequest()// 除上面外的所有请求全部需要鉴权认证
                .authenticated();
        // 禁用缓存
        httpSecurity.headers().cacheControl();
        // 添加JWT filter
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        // 添加自定义未授权和未登录结果返回
        httpSecurity.exceptionHandling()
                .accessDeniedHandler(restfulAccessDeniedHandler)
                .authenticationEntryPoint(restAuthenticationEntryPoint);
    }
  ```

- authenticationManager无法注入问题  
  在项目起动过程时，报错AuthenticationManager无法注入问题。报错信息如下：
  
  ```
  Description:
  ```

Field userService in com.zhjAdm.system.user.service.impl.UserDetailsServiceImpl required a bean of type 'org.springframework.security.authentication.AuthenticationManager' that could not be found.

The injection point has the following annotations:
    - @org.springframework.beans.factory.annotation.Autowired(required=true)

Action:

Consider defining a bean of type 'org.springframework.security.authentication.AuthenticationManager' in your configuration.

```
解决方案，在配制文件中添加：
```java
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
```