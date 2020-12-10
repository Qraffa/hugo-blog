---
title: springboot整合shiro
date: 2020-03-22 01:59:12
tags:
- springboot
- shiro
categories:
- springboot
---

- 记录一下springboot简单整合shiro

<!--more-->

## 1.引入依赖

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.5.1</version>
</dependency>
```

## 2.自定义Realm

```java
public class LoginRealm extends AuthorizingRealm {
    // 权限验证
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    // 登录验证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = (String) token.getPrincipal();
        String password = new String((char[]) token.getCredentials());
        if(!username.equals(password)) {
            // 用户名和密码不一致
            throw new IncorrectCredentialsException();
        }
        return new SimpleAuthenticationInfo(username,username,getName());
    }
}
```

这里只用到了登录验证,因此权限验证的方法没有实现,直接`return null`即可

登录验证上,通过token参数获取其中传入的用户名和密码,然后这里应该通过数据库校验用户登录,我这里只简单的把密码当做用户名验证

## 3.shiro配置类

```java
@Configuration
public class ShiroConfig {

    @Bean
    public Realm realm() {
        return new LoginRealm();
    }

    @Bean
    public SessionManager sessionManager() {
        DefaultWebSessionManager defaultWebSessionManager = new DefaultWebSessionManager();
        // 该配置作用为取消jsessionid显示在url上
        defaultWebSessionManager.setSessionIdUrlRewritingEnabled(false);
        return defaultWebSessionManager;
        //return new CustomSessionManager();
    }

    @Bean
    public DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(realm());
        manager.setSessionManager(sessionManager());
        return manager;
    }

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(DefaultWebSecurityManager manager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(manager);

        // 配置拦截器
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<>();
        // anon表示不需要认证
        // authc表示需要认证才可访问
        // 拦截顺序从上到下
        filterChainDefinitionMap.put("/login","anon");
        filterChainDefinitionMap.put("/doLogin","anon");
        filterChainDefinitionMap.put("/**","authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);

        // 设置默认登录页
        shiroFilterFactoryBean.setLoginUrl("/login");
        return shiroFilterFactoryBean;
    }
}
```

## 4.controller类

```java
@Controller
public class LoginController {

    Logger logger = LoggerFactory.getLogger(getClass());

    @RequestMapping(value = "/doLogin",method = RequestMethod.POST)
    public String doLogin(String username,String password) {
        // shiro登录
        Subject subject = SecurityUtils.getSubject();
        subject.login(new UsernamePasswordToken(username,password));
        logger.info("{},成功登录",username);
        // 这里必须使用redirect
        return "redirect:ptp/user.html";
    }

    @RequestMapping(value = "/doHello")
    public String doHello() {
        return  "ptp/user.html";
    }

    @RequestMapping(value = "/login")
    public String login() {
        return "login.html";
    }
}
```

通过以上配置springboot与shiro整合完成.这样在没有登录的情况下会默认跳转到login页面,登录完成后会跳转到ptp/user页面.

login页面的简单表单

```html
<form method="post" action="/doLogin">
    用户名：<input name="username"><br>
    密码：<input name="password"><br>
    <input type="submit" value="登录">
</form>
```