这个问题是由php的imap扩展解析hostname引起的网络问题。

有两种解决办法：

```shell
// 1. 关闭imap扩展（不建议）
// 2. 修改hosts文件，使imap能正确的将hostname识别为127.0.0.1
// 2.1 获取hostname
hostname
// 2.2 将上一条命令得到的hostname写入hosts文件
 127.0.0.1  localhost your-hostname
 ::1        localhost your-hostname
```

