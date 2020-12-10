---
title: java单例模式
date: 2020-05-01 18:18:41
tags:
- java
categories:
- java
---

## java单例模式

### 1. 饿汉式

```java
// 饿汉式,类加载时完成单例的初始化===线程安全
class Singleton1 {
    private static Singleton1 instance = new Singleton1();

    private Singleton1(){}

    public static Singleton1 getInstance() {
        return instance;
    }
}
```

<!--more-->

### 2. 懒汉式(线程不安全)

```java
// 懒汉式，在获取实例的时候才初始化===线程不安全
class Singleton2 {
    private static Singleton2 instance;

    private Singleton2(){}

    public static Singleton2 getInstance() {
        if(instance == null) {
            instance = new Singleton2();
        }
        return instance;
    }
}
```

### 3. 懒汉式(同步)

```java
// 懒汉式，在获取实例的时候才初始化，并且获取实例的方法加锁===线程安全
class Singleton3 {

    private static Singleton3 instance;

    private Singleton3(){}

    public static synchronized Singleton3 getInstance() {
        if(instance == null) {
            instance = new Singleton3();
        }
        return instance;
    }
}
```

### 4. 懒汉式(双重检测)

```java
// 懒汉式，双重检测锁===线程安全
class Singleton4 {
    // 一定要使用volatile关键字
    private volatile static Singleton4 instance;

    private Singleton4(){}

    public static Singleton4 getInstance() {
        // 双重检测
        // 第一次检测避免不必要的同步操作
        if(instance == null) {
            synchronized (Singleton4.class) {
                // 第二次检测，只有instance为被初始化才会创建
                if(instance == null) {
                    instance = new Singleton4();
                }
            }
        }
        return instance;
    }
}
```

### 5. 静态内部类

```java
// 静态内部类===线程安全
class Singleton5 {

    private Singleton5(){}

    private static class SingletonHolder {
        private static final Singleton5 instance = new Singleton5();
    }

    public static Singleton5 getInstance() {
        return SingletonHolder.instance;
    }
}
```

### 6. 枚举类

```java
// 枚举类实现
enum Singleton6 {

    INSTANCE;

    public String method() {
        return "enum singleton";
    }
}
```

### 参考

[https://blog.csdn.net/qq_34337272/article/details/80455972](https://blog.csdn.net/qq_34337272/article/details/80455972)

[https://blog.csdn.net/mnb65482/article/details/80458571](https://blog.csdn.net/mnb65482/article/details/80458571)