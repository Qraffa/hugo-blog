---
title: springboot整合fastdep
date: 2020-04-11 00:00:00
tags:
- springboot
categories:
- springboot
---

## springboot整合fastdep

- 记录springboot整合fastdep的shiro-jwt模块,普通的整合shiro和jwt需要大量的配置.因此考虑使用该方式集成

<!--more-->

### 1.引入依赖

```xml
<dependency>
    <groupId>com.louislivi.fastdep</groupId>
    <artifactId>fastdep-shiro-jwt</artifactId>
    <version>1.0.3</version>
</dependency>
```

<!-- more -->

### 2.配置文件

```yml
fastdep:
  shiro-jwt:
    filter: #shiro过滤规则
      hello:
        path: /hello/**
        role: anon # anon表示不需要校验
      profile:
        path: /profile/**
        role: jwt # jwt表示需要token校验
    secret: "6Dx8SIuaHXJYnpsG18SSpjPs50lZcT52" # jwt秘钥
#    expireTime: 7200000 # token有效期
#    prefix: "Bearer "  # token校验时的前缀
#    signPrefix: "Bearer " # token生成签名的前缀
#    header: "Authorization" # token校验时的header头
#    以下对应为shiro配置参数，无特殊需求无需配置
#    loginUrl: 
#    successUrl: 
#    unauthorizedUrl: 
#    filterChainDefinitions: 
```

### 3.控制层

```java
@RestController
public class HelloController {

    @Autowired
    private JwtUtil jwtUtil;

    @GetMapping("/hello")
    public String doHello() {
        return "hello world!";
    }

    @PostMapping("/hello/login")
    public String login(@RequestBody User user) {
        System.out.println(user.toString());
        // 签名认证,返回token
        return jwtUtil.sign(user.getUsername());
    }

    @GetMapping("/profile")
    public String doProfile() {
        System.out.println(jwtUtil.getUserId());
        return "profile info!";
    }
}
```

### 4.跨域问题

使用之前的`addCorsMappings`方式配置跨域会与fastdep的拦截器出现冲突

因此使用如下方式实现跨域

```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config=new CorsConfiguration();
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        config.addAllowedOrigin("*");
        config.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource configSource=new UrlBasedCorsConfigurationSource();
        configSource.registerCorsConfiguration("/**",config);
        return new CorsFilter(configSource);
    }
}
```

### 参考

[https://fastdep.louislivi.com/#/module/fastdep-shiro-jwt](https://fastdep.louislivi.com/#/module/fastdep-shiro-jwt)