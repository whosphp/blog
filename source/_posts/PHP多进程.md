---
title: PHP多进程
date: 2017-03-23 10:53:16
tags:
---

```php
// 转化为守护进程
function daemonize()
{
    $pid = pcntl_fork();
    if ($pid == -1)
    {
        die("fork(1) failed!\n");
    }
    elseif ($pid > 0)
    {
        //让由用户启动的进程退出
        exit(0);
    }
​
    //建立一个有别于终端的新session以脱离终端
    posix_setsid();
​
    $pid = pcntl_fork();
    if ($pid == -1)
    {
        die("fork(2) failed!\n");
    }
    elseif ($pid > 0)
    {
        //父进程退出, 剩下子进程成为最终的独立进程
        exit(0);
    }
}
​
daemonize();
​
// 设定子进程数量
$worker_num = 10;
​
// 设定接收到信号时操作
pcntl_signal(SIGINT, function () {
  exit();
});
​
while (true) {
  $pid = pcntl_fork();
  
  //父进程和子进程都会执行下面代码
  if ($pid == -1) {
    exit("fork error");
  } else if ($pid) {
    // 在父进程执行
    static $execute = 0;
    $execute++;
    if($execute >= $worker_num) {
      pcntl_wait($status);
      $execute--;
    }
  } else {
    // 在子进程执行
    // 业务逻辑放在这
    
    exit();
  }
​
  // 信号处理函数在此执行
  pcntl_signal_dispatch();
}
```

