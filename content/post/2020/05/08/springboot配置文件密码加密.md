---
title: springboot配置文件密码加密
date: 2020-05-08 13:59:47
tags:
- springboot
categories:
- springboot
---

## Spring Boot配置文件密码加密

使用`Jasypt`对springboot项目中的配置文件中的一些敏感信息,如数据库密码等,进行加密,避免明文暴露.

<!--more-->

### 1. 引入依赖

```xml
<!-- https://mvnrepository.com/artifact/com.github.ulisesbocchio/jasypt-spring-boot-starter -->
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.2</version>
</dependency>
```
### 2. 在配置文件中配置密钥

```yaml
jasypt:
  encryptor:
    password: xxxxxxx
```

### 3. 在测试文件中获取加密信息

```java
@Autowired
StringEncryptor encryptor;

@Test
void contextLoads() {

    String url = encryptor.encrypt("");
    String uname = encryptor.encrypt("");
    String pwd = encryptor.encrypt("");

    System.out.printf("url===>%s\n",url);
    System.out.printf("uname===>%s\n",uname);
    System.out.printf("pwd===>%s\n",pwd);
}
```

### 4. 根据输出的密文替换配置文件中的明文

```yaml
spring:
  datasource:
    username: ENC(xxxxxx)
    password: ENC(xxxxxx)
    url: ENC(xxxxxx)
```

格式`ENC()`是固定的,括号中填写对应的密文

### 5. 测试数据库能否正常连接

测试能成功连接就可以继续打包部署了

### 6. 打包部署

1. 首先将配置文件中的密钥删除.将测试文件中的加密也删除.因为可以通过密钥加密获得明文.

   将以下内容删除.**一定要备份记住这个密钥,后面还要使用**

   ```yaml
   jasypt:
     encryptor:
       password: xxxxxxx
   ```

2. **不要通过IDEA的maven直接打包**

   IDEA的maven打包会进行测试,比如数据库连接.

   因为我们已经把密钥删除了,因此测试是无法通过的,也就无法打包成功

3. 通过命令行**跳过测试**打包

   ```cmd
   mvn clean package -DskipTests
   ```

4. 项目部署启动

   通过以下方式添加之前删除的密钥

   ```cmd
   java -jar xxx.jar --jasypt.encryptor.password=xxx 
   ```

   