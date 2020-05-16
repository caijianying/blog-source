---
title: 基于spring security oauth2 client最佳实践
tags:
  - 教程
originContent: ''
categories:
  - java
  - spring
toc: false
date: 2020-05-16 23:07:45
---

# 基于spring security oauth2 client最佳实践

## 开篇语

最近很少写文章，一个是确实是很忙，另外一个原因是没有什么深度的技术文章可写。之前写blog的原因是为了技术存档，便于自己某天需要的时候再去看看，另外是总结一下。这段时间不太想写种水文，这篇文章同样不是什么深度性的文章，不过确实困扰了我超过3天时间，网络上很多文章都没能解决我的问题，基本上大家是介绍整个oauth,体系很大，文章却写的不全，要么就是方案很复杂（有点追求，不想采用），对我几乎无帮助。

按说官网文档应该够全了，但是对于一个不熟悉`spring security`的人，想要快速入手，还是很难，文档我就看了很久也没有找到自己想要的。官方的demo局限于github,google。我想实现的是自定义的oauth2登录。

当我解决了以后，发现别的小伙伴也有类似的疑惑。索性就写下来，只是技巧，写最少的代码，最优雅的完成自己想要的功能。本篇文章不讲解oauth认证基本知识。

## 实践

### 1.引入相应的依赖包

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```



### 2.参数配置

```
spring.security.oauth2.client.provider.customer.authorization-uri=http://xxxxxxxx/oauth2/v1/authorize
spring.security.oauth2.client.provider.customer.token-uri=http://xxxxxxxx/oauth2/v1/token
spring.security.oauth2.client.provider.customer.user-info-uri=http://xxxxxxxx/oauth2/v1/user-info
spring.security.oauth2.client.provider.customer.user-info-authentication-method=header
spring.security.oauth2.client.provider.customer.user-name-attribute=name

spring.security.oauth2.client.registration.app.client-id=xxxxxxxxxxx
spring.security.oauth2.client.registration.app.client-secret=xxxxxxxxxxx
spring.security.oauth2.client.registration.app.client-name=Client for user scope
spring.security.oauth2.client.registration.app.provider=customer
spring.security.oauth2.client.registration.app.scope=user
spring.security.oauth2.client.registration.app.redirect-uri={baseUrl}/login/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.app.client-authentication-method=basic
spring.security.oauth2.client.registration.app.authorization-grant-type=authorization_code
```



3.代码实现

在页面登录按钮，添加跳转地址`/oauth2/authorization/app` 这个是默认的地址，可以通过配置文件修改

新建Oauth2LoginSecurityConfig 实现如下功能既可

```java
@Configuration
public class Oauth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests(a -> a
                        .antMatchers("/**").permitAll().anyRequest().authenticated()
                )
                .exceptionHandling(e -> e
                        .authenticationEntryPoint(new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED))
                )
                .formLogin()
                .loginPage("/#/user/login")
                .permitAll()
                .and()
                .logout().permitAll()
                .and()
                .oauth2Login().userInfoEndpoint().and().successHandler(new AuthenticationSuccessHandler() {
            @Override
            public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                OAuth2User oAuth2User = (OAuth2User) authentication.getPrincipal();
                Map<String, Object> attributes = oAuth2User.getAttributes();
                // ....登录成功以后，既可获取用户信息
                
                // ..跳转到成功页面
                httpServletResponse.sendRedirect("/#/home");
            }
        });
    }

}
```

上面核心代码为`.oauth2Login().userInfoEndpoint().and().successHandler()` 这个完成获取code,根据code获取token,根据token获取user信息。熟悉oauth2 code的认证流程，应该就能明白。希望能给你带来一点点帮助。

## 后话

上面没有什么说的，不过最近越来越发现在职场上需要一种能力，快速学习的能力，对未知事物有非方向上认知错误。也就是说当我们对一个技术架构，或者框架不熟的情况下，一些基本技术常识往往就发挥很大的作用。在开发的路上少追求技巧性的东西，多追究一些理论和本质。这样对快速学习一个技术会有很大的帮助。