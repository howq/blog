---
title: Ubuntu下编译php5.6并添加pcntl拓展
date: 2016-07-14 22:30:26
tags: 
    - php
    - 编译
    - pcntl
---

阅读[workerman](https://github.com/walkor/Workerman)开源项目需添加php的extension--pcntl，引入多线程。折腾了两天，先将经历记录于下：

在查阅了N多资料后，在ubuntu下引入php的extension无非不过两种方法：

1. 重新编译PHP的后面configrue提示加上–enable-pcntl。
2. 不重新编译php，直接编译安装pcntl扩展。

---
### 方法2

由于笔者初入ubuntu江湖，对ubuntu下的./configrue编译尚且不太熟悉。所以首先采用了方法2来添加pcntl.so

1. 查看已安装php的版本及相关配置信息
```
<?php phpinfo() ?>
```

2. 下载同版本[php](http://php.net/downloads.php)的源码

3. 编译pcntl.so

  3.1 生成pcntl.so
  ```
  # cd /usr/local/src/php-5.6.21/ext/pcntl

  # /usr/bin/phpize

  # ./configure –with-php-config=/usr//bin/php-config

  # make && make install

  ```
  生成的pcntl.so在/usr/lib/php5/20121212

  ***注意：***
  * phpize和php-config需使用已安装的php的

  3.2 修改php.ini
  > 如果找不到php.ini 可以使用locate php.ini 或者 find / -name php.ini进行查找

  然后将pcntl.so 加到php.ini中就可以了，使用php -m查看模块命令可以查看已安装的模块。  
  **奇迹尚未出现**，笔者就此试了若干次，也不能成功，无赖只能使用方法1。

---
### 方法1
1. 只需完全卸载php
  ```
  sudo apt-get –purge remove libapache2-mod-php5 php5 php5-gd php5-mysql
  sudo apt-get autoremove php5
  ```
  删除关联
  ```
  sudo find /etc -name "*php*" |xargs  rm -rf
  ```
  清楚残留信息
  ```
  dpkg -l |grep ^rc|awk ‘{print $2}’ |sudo xargs dpkg -P
  ```
  最后用 ```dpkg -l | grep php``` 和 ```dpkg -l | grep php5 ```检查，如无返回即干净卸载

  ***注意***
    笔者走了许多弯路，将apache、php、mysql全部卸载发现不能重新安装apache这可急坏了笔者
    ```
      Reading package lists... Done

      Building dependency tree       

      Reading state information... Done

      Some packages could not be installed. This may mean that you have

      requested an impossible situation or if you are using the unstable

      distribution that some required packages have not yet been created

      or been moved out of Incoming.

      The following information may help to resolve the situation:

      The following packages have unmet dependencies:

      apache2 : Depends: apache2-bin (= 2.4.7-1ubuntu4.8) but it is not going to be installed

      E: Unable to correct problems, you have held broken packages.

    ```

    所幸找到了方法解决：
    ```
    sudo aptitude install apache2

    sudo aptitude remove  libaprutil1

    sudo aptitude install apache2

    ```

2. 下载同版本[php](http://php.net/downloads.php)的源码

3. 编译php

  ```
  cd php-5.6.21

  ./configure --with-apxs2=/usr/bin/apxs --with-mysql=mysqlnd --enable-fpm --enable-pcntl --enable-mysqlnd --enable-opcache --enable-sockets --enable-sysvmsg --enable-sysvsem  --enable-sysvshm --enable-shmop --enable-zip --enable-ftp --enable-soap --enable-xml --enable-mbstring --disable-rpath --disable-debug --disable-fileinfo --with-mysql --with-mysqli --with-pdo-mysql --with-pcre-regex --with-iconv --with-zlib --with-mcrypt --with-gd --with-openssl --with-mhash --with-xmlrpc --with-curl --with-imap-ssl
  ```

  ***注意***
  需要重点关注的是:
  * ```--with-apxs2```和```--with-mysql```的位置
    apxs可以用```locate apxs```来查找，但是mysql的找不到只能用mysqlnd
    > 如果源码编译安装php的话，需要在编译时指定–with-apxs2=/usr/local/apache2/bin/apxs表示告诉编译器通过apache的mod_php5模块来提供对php的解析

  *  ```--prefix=/usr/local/php```  ```--with-config-file-path=/usr/local/php/etc```参数的使用
  笔者起先自定义的安装路径，使用```php -m``` 查看php未安装成功，索性去掉这两个参数，使用默认安装路径，结果成功了

  * 可能缺少某些库，只需使用```sudo apt-cache search```查找，然后安装即可

4. make && make install
      > Installing shared extensions:     /usr/local/lib/php/extensions/no-debug-non-zts-20131226/
      Installing PHP CLI binary:        /usr/local/bin/
      Installing PHP CLI man page:      /usr/local/php/man/man1/
      Installing PHP FPM binary:        /usr/local/sbin/
      Installing PHP FPM config:        /usr/local/etc/
      Installing PHP FPM man page:      /usr/local/php/man/man8/
      Installing PHP FPM status page:   /usr/local/php/php/fpm/
      Installing PHP CGI binary:        /usr/local/bin/
      Installing PHP CGI man page:      /usr/local/php/man/man1/
      Installing build environment:     /usr/local/lib/php/build/
      Installing header files:           /usr/local/include/php/
      Installing helper programs:       /usr/local/bin/
        program: phpize
        program: php-config
      Installing man pages:             /usr/local/php/man/man1/
        page: phpize.1
        page: php-config.1
      Installing PEAR environment:      /usr/local/lib/php/
      [PEAR] Archive_Tar    - already installed: 1.4.0
      [PEAR] Console_Getopt - already installed: 1.4.1
      [PEAR] Structures_Graph- already installed: 1.1.1
      [PEAR] XML_Util       - already installed: 1.3.0
      [PEAR] PEAR           - already installed: 1.10.1
      Wrote PEAR system config file at: /usr/local/php/etc/pear.conf
      You may want to add: /usr/local/lib/php to your php.ini include_path
      /usr/local/src/php-5.6.21/build/shtool install -c ext/phar/phar.phar /usr/local/bin
      ln -s -f phar.phar /usr/local/bin/phar
      Installing PDO headers:           /usr/local/include/php/ext/pdo/

    运行```php -m```
    #### 最幸福的时刻莫过于此，就犹如再遇许久未逢的那个姑娘
    **Success**

---
但是革命尚且成功，同志仍需努力
运行```<?php phpinfo(); ?>``` OMG!居然给我显示的是源码。
好吧，看来还得将编译安装php与apache相关连起来，经查阅资料，关键的关键在于```mod_php5.so```的加载
> Apache对于php的解析，就是通过众多Module中的php Module来完成的，加载php是通过php5这个模块来实现的，下面通过图来说明Apache加载php模块的过程及代码如何加载php。
　　把php最终集成到Apache系统中，还需要对Apache进行一些必要的设置。这里，我们就以php的mod_php5 SAPI运行模式为例进行讲解，至于SAPI这个概念后面我们还会在其它的文章中讲解。
　　假定我们安装的版本是Apache2 和 Php5，那么需要编辑Apache的主配置文件http.conf，在其中加入下面的几行内容：
　　Unix/Linux环境下：
　　LoadModule php5_module modules/mod_php5.so
　　AddType application/x-httpd-php .php
　　注：其中modules/mod_php5.so 是X系统环境下mod_php5.so文件的安装位置。
　　Windows环境下：
　　LoadModule php5_module d:/php/php5apache2.dll
　　AddType application/x-httpd-php .php
　　注：其中d:/php/php5apache2.dll 是在Windows环境下php5apache2.dll文件的安装位置。
　　这两项配置就是告诉Apache Server，以后收到的Url用户请求，凡是以php作为后缀，就需要调用php5_module模块（mod_php5.so/ php5apache2.dll）进行处理。

* 配置apache的配置文件，网上好多都说是配置httpd.conf，但是经查确实找不到该文件，便直接配置```/etc/apache2/apache2.conf```

添加：
```
LoadModule php5_module /usr/local/src/php-5.6.21/libs/libphp5.so
DirectoryIndex index.html index.shtml index.cgi index.php index.phtml index.php3
AddHandler php5-script .php .html
AddType text/html .php .html
```