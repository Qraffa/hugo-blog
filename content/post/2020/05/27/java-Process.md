---
title: java-Process
date: 2020-05-27 16:38:39
tags:
- java
categories:
- java
---

## Process

java通过process来创建新进程来执行系统命令任务.

<!--more-->

```java
// 运行
String max_time = String.valueOf(1000 * 20);
String max_memory = String.valueOf(32 * 1024 * 1024);
String input_path = "/home/qraffa/IODemo/123/1.in";
String output_path = "/home/qraffa/IODemo/123/1.out";
String exePath = "/home/qraffa/IODemo/123/main";
// 命令执行参数
List<String> cmd = LanguageUtil.RunC(exePath, max_time, max_memory, input_path, output_path);

// 开启进程执行
Process process = new ProcessBuilder().command(cmd).start();
Process process2 = new ProcessBuilder().command(cmd).start();
// 获取进程pid,该方法在JDK9+支持
System.out.println(process.pid());
System.out.println(process2.pid());
```

#### 设置命令参数

使用`command()`方法,该方法可以使用`List<String>`或者多个字符串做参数

所有参数中的第一个,表示需要执行的命令,后面的参数表示该命令的参数

#### 启动执行

使用ProcessBuilder的`start()`方法启动,并返回一个Process实例

#### 获取执行结果

获取该结果会导致去调用任务的线程阻塞,直到等到结果返回.

```java
byte[] bytes = new byte[512];
int cnt = process.getInputStream().read(bytes);
System.out.println(new String(bytes, 0, cnt));
```

当运行多个不同时长的任务时,调用了`getInputStream`去获取结果,只会阻塞到该process的进程任务完成.其他任务会在后台继续执行,直到结束,但主线程会结束.

#### 多任务执行

可以同时启动多个进程去执行不同任务,多任务并发,但在调用了`getInputStream`去获取结果时会阻塞.

启动两个进程去执行不同的任务,process2的运行时间大约在19s,process的运行时间大约在2s.

如果先阻塞获取process2的结果,会阻塞.然后先得到process2的结果,再立刻得到process的结果

如果先阻塞获取process的结果,会阻塞2s,获取process的结果,然后再阻塞获取process2的结果

```java
byte[] bytes2 = new byte[512];
int cnt2 = process2.getInputStream().read(bytes2);
System.out.println(new String(bytes2, 0, cnt2));

byte[] bytes = new byte[512];
int cnt = process.getInputStream().read(bytes);
System.out.println(new String(bytes, 0, cnt));
```

