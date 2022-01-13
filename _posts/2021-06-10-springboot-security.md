---
layout: post
title: "springboot security"
date:   2021-06-10
tags: [spring]
comments: true
author: xiaobai
---

springboot security 基础用法

<!-- more -->

- [导入依赖](#导入依赖)
- [新建SecurityConfig类](#新建securityconfig类)

### 导入依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



### 新建SecurityConfig类

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    //授权
    @Override
    protected void configure(HttpSecurity http) throws Exception {
      // 请求授权的规则
        http.authorizeRequests()
                .antMatchers("/").permitAll()
                .antMatchers("/leven1").hasRole("vip1")
                .antMatchers("/leven2").hasRole("vip2")
                .antMatchers("/leven3").hasRole("vip3");

        // 没有权限跳转到login页面, 定制login页面
        // usernameParameter 接受登录用户名的参数
        // passwordParameter 接受登录密码的参数
        // loginProcessingUrl 登录成功跳转的页面
        http.formLogin().loginPage("/login").usernameParameter("user").passwordParameter("pwd").loginProcessingUrl("/");
        // 开启记住我功能
        // rememberMeParameter: 接收记住我的参数
        http.rememberMe().rememberMeParameter("remember");
        // 开启注销功能,注销成功后跳转到首页
        http.logout().logoutSuccessUrl("/");

    }

    // 登录认证
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 从内存中获取认证的用户信息
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1")
                .and()
                .withUser("admin").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2", "vip3");
      
      // 从数据库获取登录用户信息
        User.UserBuilder user = User.withDefaultPasswordEncoder();
        auth.jdbcAuthentication()
                .dataSource(dataSource)
                .withDefaultSchema()
                .withUser(user.username("root").password("123456").roles("vip1"))
                .withUser(user.username("admin").password("123456").roles("vip2"));

    }
}

```

