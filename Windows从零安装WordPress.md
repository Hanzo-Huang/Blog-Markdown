# Windows从零安装WordPress

> 在Linux中，可以用Linux运维工具配合docker很便捷的安装并配置MySQL、nginx、php、WordPress，但是Windows服务器中，我还没有发现类似的面板，就尝试学习用最原始的方法进行安装。

## 准备工作

1. [Microsoft Visual C++ Redistributable Package](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170)

2. [MySQL](https://dev.mysql.com/downloads/installer/)

3. [nginx](https://nginx.org/en/download.html)

4. [php](https://windows.php.net/download/)

5. [WordPress](https://wordpress.org/download/)

   ![download](./guide/download.png)

### Microsoft Visual C++ Redistributable Package

> Microsoft Visual C++ Redistributable Package为MySQL依赖的环境

直接同意并安装即可。

![Microsoft Visual C++ Redistributable Package](./guide/VC++.png)

### MySQL

> MySQL数据库为WordPress必须的环境之一，提供数据存放服务

打开安装包，选择`Server only`，后续默认继续即可

![mysql-1](./guide/mysql-1.png)

输入的密码为数据库`root`管理用户的密码，<font color=red>请牢记</font>

![mysql-2](./guide/mysql-2.png)

### nginx

> nginx在我们建站的过程中起到作为Web 服务器的用处，负责将网页内容对应至网址端口

直接将下载的安装包解压即可，我解压在`C:\server\nginx`，双击目录的`nginx.exe`即运行程序。

浏览网址http://127.0.0.1/，出现以下内容则运行正常

![nginx](./guide/nginx.png)

**停止nginx**

`WIN + R`输入`cmd`进入命令行，定位至文件夹（`C:\server\nginx`）并停止应用服务

```bash
cd C:\server\nginx
nginx.exe -s stop
```

### php

> php为WordPress必须的环境之一，php是为Web开发提供的一种简单、高效、灵活的编程语言，类似于html，wordpress就是用php进行编写的

直接解压压缩包至`C:\server\php`，后续进行配置

### WordPress

> *WordPress*本身，提供了基础的创建页面，创建文章功能，可以安装丰富插件

直接解压压缩包至`C:\server\wordpress`

**最终`C:\server`文件夹有一下内容**（位置只是根据个人偏好选择，任意位置都可以）

![folder](./guide/folder.png)

## 配置

<font color=red>注意：下面配置的地址均更改为实际安装位置，如果安装位置和我一样，则直接复制粘贴即可</font>

### 1. MySQL添加数据库

在`开始`菜单中找到`MySQL 8.0 Command Line Client`

![mysql-3](./guide/mysql-3.png)

输入刚才的`root`密码并回车，进入MySQL命令行，输入以下命令创建名为`wordpress`的数据库

```mysql
CREATE DATABASE wordpress;
```

### 2. nginx配置

nginx的配置文件是`nginx\conf`里的`nginx.conf`，可以用记事本打开。找到第35行开始的`server`

1. `location /`（第43行）

   `root`: `wordpress`所在位置

   `index`: 添加`index.php`，因为wordpress使用php编写

           location / {
               root   C:/server/wordpress; # 设置为wordpress的位置
               index  index.html index.htm index.php; # 添加index.php
           }

2. `location ~ \.php$`（第65行）

   首先需要将这几行前的`#`删掉以启用php

   `root`: `wordpress`所在位置

   `fastcgi_param`: 直接修改为`$document_root$fastcgi_script_name`

   ```
           location ~ \.php$ {
               root           C:/server/wordpress; # 设置为wordpress的位置
               fastcgi_pass   127.0.0.1:9000;
               fastcgi_index  index.php;
               fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
               include        fastcgi_params;
           }
   ```

以下为修改好的配置，如果前面安装位置和我相同，则可以直接复制替换`server`部分。

```
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   C:/server/wordpress; # 设置为wordpress的位置
            index  index.html index.htm index.php; # 添加index.php
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root           C:/server/wordpress; # 设置为wordpress的位置
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name; # 修改为$document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
```

### 3. php配置

进入php文件夹，找到`php.ini-development`配置文件并复制为`php.ini`。

1. 搜索`extension_dir`，找到第778行的内容

   ```
   ;extension_dir = "ext"
   ```

   删去最开始的`;`，将`ext`改为php中的真实路径

   ```
   extension_dir = "C:\server\php\ext"
   ```

2. 搜索`cgi.fix_pathinfo`，找到第815行内容

   ```
   ;cgi.fix_pathinfo=1
   ```

   删去`;`

   ```
   cgi.fix_pathinfo=1
   ```

3. 添加扩展`php_mysqli.dll`

   因为配置文件中没有找到该项，于是我们自行添加至任意位置即可，为了方便管理，我放在`extension`项的末尾（第975行）。直接加入以下两行并保存

   ```
   ;mysql extension
   extension=php_mysqli.dll
   ```

### 4. 启动nginx和php

`win + r`输入`cmd`打开命令提示行，分别输入以下两行

```
start C:\server\nginx\nginx.exe
C:\server\php\php-cgi.exe -b 127.0.0.1:9000 -c C:\server\php\php.ini
```

![run nginx and php](./guide/run nginx and php.png)

注意：运行第二行之后会卡住不动，不要关闭窗口，保持即可。

接下来在浏览器中访问`127.0.0.1`

![wp-setup](./guide/wp-setup.png)

可以发现成功进入wordpress的`setup-config`页面，至此启动成功。

#### 设置批处理文件

刚才的窗口是不能关的，一旦关闭，php-cgi就会关闭，就不能解析php文件。我们可以用批处理来打开/关闭nginx和php-cgi， 

　　这里用到RunHiddenConsole(用来隐藏CMD命令窗)，把RunHiddenConsole.exe和批处理文件放在同一级目录

1. **start.bat**

   ```bat
   @echo off
   
   set PHP_FCGI_MAX_REQUESTS = 1000
   
   echo Starting PHP FastCGI...
   
   rem 下面这里的分别替换成你的php-cgi.exe和php.ini的路径 ，后面的-b,-c等参数必须保留且注意前后空格
   
   RunHiddenConsole E:\self\soft\php-7.2.11/php-cgi.exe -b 127.0.0.1:9001 -c E:\self\soft\php-7.2.11/php.ini 
   
   echo Starting nginx...
   
   rem 注意替换成你的nginx目录
   
   E:\self\soft\nginx-1.14.0/nginx.exe -p E:\self\soft\nginx-1.14.0/
   
   cd D:/webServer/nginx-1.5.2/
   
   pause
   ```

2. **stop.bat**

   ```bat
   @echo off
   echo Stopping nginx...  
   taskkill /F /IM nginx.exe > nul
   echo Stopping PHP FastCGI...
   taskkill /F /IM php-cgi.exe > nul
   exit
   ```

   

**Reference:**

[windows下搭建nginx+php开发环境(by 苦逼成长印记)-cnblog](https://www.cnblogs.com/wwjchina/p/9804576.html)
