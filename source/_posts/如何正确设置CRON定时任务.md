---
title: 如何正确设置CRON定时任务
date: 2017-03-22 10:49:30
tags:
---

转载自[火丁笔记](http://huoding.com/)，原文链接http://huoding.com/2016/12/12/573

相信很多人看了标题后都会纳闷：设置 CRON 定时任务有什么难的？不过请相信我，正确设置 CRON 真的不是一件简单的事情！各位看官不妨听我慢慢道来。


关于 CRON，出镜率最高的一个问题莫过于：为什么手动执行一切正常，放到 CRON 里就不执行呢？实际上此类问题多半是因为环境变量导致的，答案就在配置文件里：

```shell> cat /etc/crontab
 SHELL=/bin/bash
 PATH=/sbin:/bin:/usr/sbin:/usr/bin
 MAILTO=root
 HOME=/
 
 # For details see man 4 crontabs

 # Example of job definition:
 # .---------------- minute (0 - 59)
 # |  .------------- hour (0 - 23)
 # |  |  .---------- day of month (1 - 31)
 # |  |  |  .------- month (1 - 12) OR jan, ...
 # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun, ...
 # |  |  |  |  |
 # *  *  *  *  * user-name command to be executed
```

比如说，你的 PHP 命令位于 /usr/loca/bin 目录，而你在 profile 里已经把这个目录加到了环境变量 PATH 里，不过 crontab 里可没有 /usr/loca/bin 目录，于是就出问题了。对付此类问题的方法很简单，那就是设置 CRON 的时候尽可能使用完整的全路径。

此外，有人喜欢直接在 /etc/crontab 里配置定时任务，这同样是十恶不赦的做法，多数时候，我们都应该使用 crontab -e 的方法来设置，原因是这样有语法检查。

如果本文的内容仅限于此类小菜，那么未免有些太对不起各位看官，下面上一道硬菜：设置一个 PHP 脚本，每分钟执行一次，怎么搞？听起来这分明就是一道送分题啊：

```* * * * * /path/to/php /path/to/file`
让我们设想如下情况：假如上一分钟的 A 请求还没退出，下一分钟的 B 请求也启动了，就会导致出现 AB 同时请求的情况，如何避免？答案是 flock，它实现了锁机制：

 `flock -xn /tmp/lock /path/to/php /path/to/file`
让我们再来重放一下故障场景：假如上一分钟的 A 请求还没退出，下一分钟的 B 请求也启动了，那么 B 请求会发现 A 请求还没有释放锁，于是它不会执行。

看起来似乎完美解决了问题，不过让我们在加入一点特殊情况：假如因为某些无法预知的原因，导致脚本不能正常结束请求，进而导致不能正常释放锁，那么后续所有其它的 CD 等请求也都无法执行了，如何避免？答案是 timeout，它实现了超时控制机制：

 `timeout -s SIGINT 100 flock -xn /tmp/lock /path/to/php /path/to/file`
让我们再来重放一下故障场景：假如上一分钟的 A 请求还没退出，下一分钟的 B 请求也启动了，那么 B 请求会发现 A 的请求还没有释放锁，于是它不会执行，不过下下分钟的 C 请求肯定能执行，因为在这之前，A 请求已经因为超时被 timeout 干掉了。

当然，无论是锁机制，还是超时控制禁止，我们都可以自己实现，不过既然系统已经提供了这样的功能，那么除非你对自己的编码能力有自信，否则还是使用系统的吧。