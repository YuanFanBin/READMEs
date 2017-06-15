# php

## Install

### CentOS7

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

## php7.1.x ext

### Compile php7.1.x & Add php.ini

```sh
$ cd php-7.1.x
$ ./configure --prefix=/usr/local/php7.1.x \
              --with-config-file-path=/usr/local/php7.1.x/etc
$ make && sudo make install
$ /bin/cp -f php.ini-development /usr/local/php7.1.x/etc
```

### Add extension

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

### Test

```sh
$ vim /usr/local/php7.1.x/etc/php.ini

    extension=ROOT/php-7.1.x/ext/EXTNAME/modules/EXTNAME.so

$ cd ./ext/EXTNAME && make test
```

## Extension (https://wiki.archlinux.org/index.php/PHP)

```sh
$ pacman -Ss php-
```
