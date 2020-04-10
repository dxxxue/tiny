### authorized_keys修改不生效的解决
- 查看日志 cat /var/log/secure 系统相关的日志
![authorized_keys_log](./img/authorized_keys_log.jpg)
- 一般都是权限问题(SSH不希望home目录和~/.ssh目录对组有写权限)
- 对.ssh目录一般都是700, 对authorized_keys文件一般都是600即可

### 下个问题(例如设置ssh登录心跳问题)
