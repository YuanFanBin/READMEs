# php7.1.x ext

## Compile php7.1.x & Add php.ini

```sh
$ cd php-7.1.x
$ ./configure --prefix=/usr/local/php7.1.x \
              --with-config-file-path=/usr/local/php7.1.x/etc
$ make && sudo make install
$ /bin/cp -f php.ini-development /usr/local/php7.1.x/etc
```

## Add extension

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

## Test

```sh
$ vim /usr/local/php7.1.x/etc/php.ini

    extension=ROOT/php-7.1.x/ext/EXTNAME/modules/EXTNAME.so

$ cd ./ext/EXTNAME && make test
```
