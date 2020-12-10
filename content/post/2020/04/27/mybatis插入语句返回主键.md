---
title: mybatis插入语句返回主键
date: 2020-04-27 18:56:45
tags:
- mybatis
categories:
- mybatis
---

## mybatis插入语句如何返回主键的问题

### useGeneratedKeys和keyProperty

问题背景：在做表插入的时候，我们希望插入完成后获取刚刚插入的主键，以便完成后续操作.

<!--more-->

最先想到的方法是先插入数据,然后根据插入的数据再查询一遍获取主键,但上述方法有很大局限性

解决方法:  
因此使用**useGeneratedKeys和keyProperty**,**useGeneratedKeys**设置为true后,插入语句返回的主键会映射到**keyProperty**指定的属性上

```xml
<insert id="addCategory" parameterType="CategoryDO" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO category(name,created)
        VALUES(#{name},#{created})
</insert>
```

该方法映射回去的时候是映射到传进来的参数`parameterType`的对象上的.需要通过该对象的set方法来获取需要的值,而不能直接通过dao层方法获取值,因为这样获取到的是mysql执行数据库影响的行数.

