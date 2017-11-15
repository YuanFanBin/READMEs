# MySQL

## MAC 忘记root密码

```sh
# 进入系统偏好设置，关闭MySQL服务
$ cd /usr/local/mysql/bin               # 进入mysql bin目录
$ sudo su                               # 获取root权限
$ ./mysqld_safe --skip-grant-tables &   # 重启服务器

# 开启新终端
$ mysql -uroot -p                       # 提示输入密码时随便输入即可
mysql > flush privileges;               # 获取权限
mysql > SET PASSWORD FOR 'root'@'localhost'=password('新密码');
```

参考资料：http://www.jianshu.com/p/47caaf07fdfa
