---
title: "Crontab问题记录"
date: 2020-09-07T15:56:32+08:00
tags:
- linux
categories: 
- linux
---

## crontab问题记录

服务器出现大量相同的`cron-msp`进程。

<!--more-->

**ps -aux**

```bash
root     20926  0.0  0.0  50216   384 ?        S    Aug28   0:00 /usr/sbin/CRON -f
smmsp    20928  0.0  0.0  12504   196 ?        Ss   Aug28   0:00 /bin/sh -c test -x /etc/init.d/sendmail && test -x /usr/share/sendmail/sendmail && test -x /usr/lib/sm.bin/sendmail && /usr/share/sendmail/s
smmsp    20932  0.0  0.0  12816   500 ?        S    Aug28   0:00 /bin/sh /usr/share/sendmail/sendmail cron-msp
smmsp    20966  0.0  0.0  12816   504 ?        S    Aug28   0:00 /bin/sh /usr/share/sendmail/sendmail cron-msp
smmsp    20967  0.0  0.0  26164   232 ?        S    Aug28   0:00 systemctl -p LoadState show sendmail.service
```

突然发现出现大量以上进程，大量占用的进程数资源。

查看创建日期，主要集中在July29, Aug8, Aug28

这进程描述看起来像是crontab定时任务和sendmail邮件的问题。

**cat /etc/passwd**

```bash
smmta:x:113:123:Mail Transfer Agent,,,:/var/lib/sendmail:/bin/false
smmsp:x:114:124:Mail Submission Program,,,:/var/lib/sendmail:/bin/false
```

**mail**

```bash
[-- Message  2 -- 29 lines, 1364 bytes --]:
From root@***  Wed Jul 29 13:41:01 2020
Date: Wed, 29 Jul 2020 10:40:56 +0800
Message-Id: <***@***>
From: root@*** (Cron Daemon)
To: root@***
Subject: Cron <root@***> test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )

/etc/cron.daily/logrotate:
error: error running shared postrotate script for '/var/log/mysql.log /var/log/mysql/*log '
Failed to kill unit rsyslog.service: Connection timed out
error: error running non-shared postrotate script for /var/log/syslog of '/var/log/syslog
'
run-parts: /etc/cron.daily/logrotate exited with return code 1
```

查看root用户邮件，但似乎不是这个原因引起。

**journalctl -u sendmail.service**

```bash
Jul 29 02:42:52 iZbp19844shvucajxsiimbZ sm-mta[1284]: rejecting connections on daemon MTA-v4: load average: 57
Jul 29 02:42:56 iZbp19844shvucajxsiimbZ sm-mta[1284]: rejecting connections on daemon MSP-v4: load average: 58
```

查看senmail服务的日志，发现在July29, Aug8, Aug28集中出现大量的`rejecting connections on daemon MTA-v4: load average:`

这估计就是问题所在了。

看了下日志记录时间，都是在凌晨2点，想起来我服务器每天都凌晨2点都是csgo服务器自动更新的时间。接下来继续排查。

steam 用户**mail**

```bash
 U 82 Cron Daemon        Tue Jul 28 02:01   43/1766  Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
 U 83 Cron Daemon        Wed Jul 29 13:41 2558/192856 Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
 U 84 Cron Daemon        Thu Jul 30 02:01   44/1875  Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
 ...
  U 92 Cron Daemon        Fri Aug  7 02:01   43/1763  Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
 U 93 Cron Daemon        Sat Aug  8 20:21 4378/331132 Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
 ...
  U 112 Cron Daemon        Thu Aug 27 02:01   42/1679  Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
 O 113 Cron Daemon        Fri Aug 28 12:21 2268/168585 Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
 O 114 Cron Daemon        Sat Aug 29 02:01   47/2018  Cron <steam@iZbp19844shvucajxsiimbZ> /home/steam/csgo_server_update.sh
```

由于csgo服务器是由steam用户来执行的。因此查看steam用户的邮件。发现这几天的邮件和其他相比非常大。

**查看83邮件**

```bash
 Update state (0x61) downloading, progress: 30.34 (809876498 / 2669667401)
 Update state (0x61) downloading, progress: 31.76 (847989460 / 2669667401)
 Update state (0x61) downloading, progress: 31.76 (847989460 / 2669667401)
 Update state (0x61) downloading, progress: 31.76 (847989460 / 2669667401)
 Update state (0x61) downloading, progress: 31.76 (847989460 / 2669667401)
 Update state (0x61) downloading, progress: 31.76 (847989460 / 2669667401)
 Update state (0x61) downloading, progress: 31.76 (847989460 / 2669667401)
 Update state (0x61) downloading, progress: 50.56 (1349653911 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
 Update state (0x61) downloading, progress: 52.05 (1389614764 / 2669667401)
```

查看其中一封邮件发现，下载卡在了某个结点。

**原因**

基本原因基本查清楚了，就是因为csgo服务器更新时，可能网络波动，导致文件一直下载不下来，引起sendmail出现了大量的rejecting。

但至于为什么会启动最开始提到的几个进程，还有待继续深入研究一下。

**解决**

目前最直接的解决方法，首先将这些进程全部kill，然后将steam用户的crontab定时任务邮件取消。

kill掉所有sendmail的垃圾进程

```bash
kill -9 `ps -ef | grep sendmail | awk '{print $2}' ` 
```

取消定时任务的邮件发送

```bash
crontab -e # 进入编辑状态
```

添加以下内容

```bash
MAILTO=""
```