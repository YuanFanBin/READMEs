# Menu

* [install](#install)
    * [CentOS7.x](#1-centos7x)
* [php7.1.x extension](#php71x-extension)
    * [1. Compile PHP7.1.x and Add php.ini](#1-compile-php71x-and-add-phpini)
    * [2. Add extension](#2-add-extension)
    * [3. Test](#3-test)
* [Arch PHP Extension](#arch-php-extension)
* [PHP Yii2](#php-yii2)

# Install

## 1. CentOS7.x

```sh
$ sudo rpm -Uhv https://centos7.iuscommunity.org/ius-release.rpm
$ wget ....
$ ./configure && make && make install
$ sudo yum install \
    php71u-bcmath \
    php71u-cli \
    php71u-common \
    php71u-devel \
    php71u-enchant \
    php71u-fpm \
    php71u-fpm-nginx \
    php71u-gd \
    php71u-json \
    php71u-mbstring \
    php71u-mcrypt \
    php71u-mysqlnd \
    php71u-opcache \
    php71u-pdo \
    php71u-pear \
    php71u-pecl-apcu \
    php71u-pecl-igbinary \
    php71u-pecl-imagick \
    php71u-pecl-memcached \
    php71u-pecl-redis \
    php71u-process \
    php71u-recode \
    php71u-soap \
    php71u-xml \
    php71u-xmlrpc
```

# php7.1.x extension

## 1. Compile PHP7.1.x and Add php.ini

```sh
$ cd php-7.1.x
$ ./configure --prefix=/usr/local/php7.1.x \
              --with-config-file-path=/usr/local/php7.1.x/etc
$ make && sudo make install
$ /bin/cp -f php.ini-development /usr/local/php7.1.x/etc
```

## 2. Add extension

```sh
$ cd php-7.1.x/ext
$ ./ext_skel --extname=EXTNAME
```
    ...
    1.  $ cd ..
    2.  $ vi ext/EXTNAME/config.m4
    3.  $ ./buildconf
    4.  $ ./configure --[with|enable]-EXTNAME
    5.  $ make
    6.  $ ./sapi/cli/php -f ext/EXTNAME/EXTNAME.php
    7.  $ vi ext/EXTNAME/EXTNAME.c
    8.  $ make
    ...

write something function (ex. say())

```c
/* {{{ proto string EXTNAME()
 */
PHP_FUNCTION(say)
{
	zend_string *str;

	str = strpprintf(0, "Hello, EXTNAME!");
	RETURN_STR(str);
}
/* }}} */
```

## 3. Test

```sh
$ vim /usr/local/php7.1.x/etc/php.ini

    extension=ROOT/php-7.1.x/ext/EXTNAME/modules/EXTNAME.so

$ cd ./ext/EXTNAME && make test
```

# Arch PHP Extension

[Arch PHP Extension](https://wiki.archlinux.org/index.php/PHP)

```sh
$ pacman -Ss php-
```

# PHP Yii2

- [Yii PHP Framework Version 2](http://www.yiiframework.com/doc-2.0/index.html): 官方文档
- [The Definitive Guide to Yii 2.0](http://www.yiiframework.com/doc-2.0/guide-intro-yii.html): 官方权威指南（英文）
- [Yii 2.0 权威指南](http://www.yiichina.com/doc/guide/2.0) - 官方权威指南（中文）
- [Yii2 CSRF 防范策略](http://www.cnblogs.com/ganiks/p/yii2-request-csrf-safe-strategy.html) - 从Yii2的Request看其CSRF防范策略，作者配有流程图，便于理解
- [Yii 2.0 CSRF validation for AJAX request](https://stackoverflow.com/questions/30153705/yii-2-0-csrf-validation-for-ajax-request): Yii2 中的 `csrf-param`, `csrf-token`
- [Yii2 SSO登陆 - zacklee](https://segmentfault.com/a/1190000004650718): 关于Yii2如何实现跨域的SSO登录的解析
- [Yii2 SSO登陆逻辑 - zacklee](https://segmentfault.com/a/1190000005669675): 全面解析Yii2跨域的SSO登录逻辑
- [Yii2 多域名跨域同步登录退出 - ruzuojun](https://getyii.com/topic/216)
- [Yii2 下 session 跨域名共存 - `_string_`](http://www.yiichina.com/tutorial/1073)
- [js跨域访问](http://zjblogs.com/js/Access-Control-Allow-Origin.html): js跨域访问，No ‘Access-Control-Allow-Origin‘ header is present on
- [延伸: OAuth2.0 - 阮一峰](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)：理解OAuth 2.0
- [延伸：顶级域名，二级域名](https://segmentfault.com/a/1190000006932934): 总结一下顶级域名和子级域名之间的cookie共享和相互修改、删除
- [实践：Yii2配置及搭建 - 白狼](http://www.manks.top/yii2-frame-rbac-template.html): yii2搭建完美后台并实现rbac权限控制实例教程
