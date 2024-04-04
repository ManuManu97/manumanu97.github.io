# Writing PHP or ZEND extension in C.

## setup PHP debug environtment:

1. requirements:

to build normal PHP dev env:
```
sudo apt-get install build-essential autoconf automake bison flex re2c gdb libtool make pkgconf valgrind git libxml2-dev libsqlite3-dev
```

to build apache2+php dev env:
```
sudo apt-get install build-essential autoconf automake bison flex re2c gdb libtool make pkgconf valgrind git libxml2-dev libsqlite3-dev apache2-dev apache2
```

2. download php-src and select our version:
```
git clone https://github.com/php/php-src.git
cd php-src
git checkout php-7.4.1 # version we want
```

3. configuration like SAPI(cli,fastcgi,fpm,...) and enable/diasable embedded php exts. (full list available if *configure --help*.

```
./buildconf --force
./configure --enable-debug --prefix=$PWD/../php-bin/DEBUG --with-config-file-path=$PWD/../php-bin/DEBUG/etc
make -j4
make test
make install
```

4. create custom php.ini file and put the following for base purpose:
```
mkdir $PWD/../php-bin/DEBUG/etc
cat > $PWD/../php-bin/DEBUG/etc/php.ini <<EOL
date.timezone=GMT
max_execution_time=30
memory_limit=128M

error_reporting=E_ALL | E_STRICT ; catch all error and warnings
display_errors=1
log_errors=1

zend_extension=opcache.so
opcache.enable=1
opcache.enable_cli=1
opcache.protect_memory=1      ; catch invalid updates of shared memory
EOL
```

## Generate PHP ext skeleton 

1. generate the skeleton with the script

```php
php php-src/ext/ext_skel.php --ext test --dir .
```

it will create the directory *test* in the directory specified by --dir with the following 5 files:

* **config.m4** is an extension configuration script used by “phpize” or “buildconf” to add extension configuration options into the “configure” command. 
* **config.w32** is a similar configuration file for the Windows build system, which is discussed later in this blog. 
* **php_test.h** is a C header file that contains our common extension definitions. It’s not necessary for simple extensions with a single-source C file, but it’s useful in case the implementation is spread among few files. 
* **test.c** is the main extension implementation source. It defines all the structures that allow to plug the extension into PHP and make all their internal functions, classes and constants to be available. 
* **tests** refers to the directory with PHP tests. We will review them late

## PHP lifecycle

PHP can be run by cli or fpm starting from main.c, instead if is running as module into webserver it's started a little bit after Apache2.
Starting up -> **the module startup step** -> abbreviated=**MINIT**

Once started, PHP waits to handle one/several requests.
php cli -> only one request because you execute a script
web env -> could server several requests (manage infinite requests or specific number before reload the process).
every time a new request arrives to be handled->**request startup step**->abbr=**RINIT**

The request is served, time to shutdown the request->**request shutdown step**->**RSHUTDOWN**

After having handled N requests PHP shut down itself. shutting down the PHP process -> **the module shutdown step**->**MSHUTDOWN**

![alt text](images/php_module_structure.png)

To handle several requests at the same time, you need to run a parallelism model. There exists two of them in PHP:

* The process-based model
* The thread-based model

![alt text](images/php_process_vs_thread.png)

## Zend vs PHP

| Feature           | Zend Extensions                             | PHP Extensions                        |
|-------------------|---------------------------------------------|---------------------------------------|
| Purpose           | Extend the functionality of the PHP engine | Provide additional functions/classes |
| Language          | Written in C                                | Written in C                          |
| Interaction       | Directly with the Zend Engine               | Dynamically loaded into PHP runtime   |
| Functionality     | Modify Zend Engine behavior, add constructs| Add functions, classes for PHP scripts|
| Examples          | OPcache, Xdebug, Zend Guard                 | MySQLi, GD, cURL, etc.               |

## Modify .c file:

if you want to modify the *.c* file you should take into account some stuff like:

1. remember to add your *DEBUG* PHP env in the path as first entry before doing phpize and configure.
```
export PATH=/path/php-bin/DEBUG/bin:$PATH 
```

2. if you want to write a **PHP_FUNCTION(foo_name)** you should add the following code:

```c
Example:

PHP_FUNCTION(test_double){
    double x;

	ZEND_PARSE_PARAMETERS_START(1,1)
		Z_PARAM_DOUBLE(x)
	ZEND_PARSE_PARAMETERS_END();
	php_printf("the double of n %f, is %f",x,x*3);
}

ZEND_BEGIN_ARG_INFO(arginfo_test_double, 0)
	ZEND_ARG_INFO(0, double)
ZEND_END_ARG_INFO() // in which yo specify the argument type and info of the param

static const zend_function_entry pino_functions[] = {
	/*
    other PHP_FE(function_name, arginfo_function_name)
    */
	PHP_FE(test_double,		arginfo_test_double) // your entry
	PHP_FE_END
};
```

afther that if you already installed the extension with phpize && ./configure you can reinstall it using:
```
make
make install
```



