# zabbix

## zabbix 3.0.9

ref: http://www.jianshu.com/p/4d98ff87db5f

### mysql

```sql
GRANT ALL PRIVILEGES ON zabbix.* 'zabbix'@'localhost';
```

### BUG

```log
A non well formed numeric value encountered [zabbix.php:21 → require_once() → ZBase->run() → ZBase->processRequest() → CView->getOutput() → include() → make_status_of_zbx() → CFrontendSetup->checkRequirements() → CFrontendSetup->checkPhpMemoryLimit() → str2mem() in include/func.inc.php:410]
```

    安装完成之后启动就出现这个问题，这个是因为PHP 7.1.0类型强化，处理方法也很简单找到Zabbix WEB目录下include/func.inc.php文件

```sh
$ sed -i '/$last = strtolower(substr($val, -1));/a$val = substr($val,0,-1);' /var/www/zabbix/include/func.inc.php
```

### test

```sh
$  curl -i -X POST -H 'Content-Type:application/json' -d'{"jsonrpc": "2.0","method":"user.login","params":{"user":"Admin","password":"zabbix"},"auth": null,"id":0}' http://192.168.1.88/zabbix/api_jsonrpc.php
```

ref: http://www.linuxidc.com/Linux/2017-02/140146.htm
