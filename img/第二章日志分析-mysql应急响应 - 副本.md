# 第二章日志分析-mysql应急响应

```
mysql应急响应 ssh账号 root  密码 xjmysql
ssh env.xj.edisec.net  -p xxxxx
1.黑客第一次写入的shell flag{关键字符串} 
2.黑客反弹shell的ip flag{ip}
3.黑客提权文件的完整路径 md5 flag{md5} 注 /xxx/xxx/xxx/xxx/xxx.xx
4.黑客获取的权限 flag{whoami后的值}
```

## 1.黑客第一次写入的shell flag{关键字符串} 

先去到/var/www/html/使用

```
grep -rE "eval|assert|system|passthru|shell|shell_exec" *.php
```

![image-20250218085955727](第二章日志分析-mysql应急响应.assets/image-20250218085955727.png)

发现sh.php有一句话木马，查看一下

![image-20250218090044859](第二章日志分析-mysql应急响应.assets/image-20250218090044859.png)

```
flag{ccfda79e-7aa1-4275-bc26-a6189eb9a20b}
```

## 2.黑客反弹shell的ip flag{ip}

去查看mysql日记

```
cat /var/log/mysql/error.log
```

![image-20250218090650329](第二章日志分析-mysql应急响应.assets/image-20250218090650329.png)

发现存在一个1.sh是一个shell脚本位置再/tmp/1.sh

```
cat /tmp/1.sh
```

![image-20250218090940827](第二章日志分析-mysql应急响应.assets/image-20250218090940827.png)

```
flag{192.168.100.13}
```

## 3.黑客提权文件的完整路径 md5 flag{md5} 注 /xxx/xxx/xxx/xxx/xxx.xx

UDF提权说明:

UDF（Userdefined function）可翻译为用户自定义函数，其为 mysql 的一个拓展接口，可以为 mysql 增添一些函数。比如 mysql 一些函数没有，我就使用 UDF 加入一些函数进去，那么我就可以在 mysql 中使用这个函数了。

条件：

获取了MySQL控制权：获取到账号密码，并且可以远程连接

获取到的账户具有写入权限，即secure_file_priv值为空

什么情况下需使用mysql提权？

拿到了mysql的权限，但是没拿到mysql所在服务器的任何权限，通过mysql提权，将mysql权限提升到操作

既然要提权，那黑客肯定知晓了账号密码，而且连接上了，那么我们可以猜想，是不是web目录下有页面泄露了mysql账号密码，回到web目录

```
grep -rE 'root' *.php
```

![image-20250218091506158](第二章日志分析-mysql应急响应.assets/image-20250218091506158.png)

出现用户密码连接myql

```
mysql -uroot -p334cc35b3c704593
```

![image-20250218091829042](第二章日志分析-mysql应急响应.assets/image-20250218091829042.png)

```
show global variables like '%secure%';
```

![image-20250218091903349](第二章日志分析-mysql应急响应.assets/image-20250218091903349.png)



可以发现secure_file_priv` 变量为空意味着 MySQL 没有对文件操作进行限制。这就是黑客利用进行UDF (User Defined Functions) 提权，因为他们可以将任意的共享库文件（例如 `.so` 文件）上传到服务器并通过 MySQL 加载执行。

在进行 UDF (User Defined Function) 提权时，攻击者通常会将恶意共享库文件放在 MySQL 插件目录中。这个目录的默认路径通常是 `/usr/lib/mysql/plugin/`，但具体路径取决于 MySQL 的安装和配置。

Ctrl+c退出数据库；

进入/usr/lib/mysql/plugin/目录下；

![image-20250218092504464](第二章日志分析-mysql应急响应.assets/image-20250218092504464.png)

```
/usr/lib/mysql/plugin/udf.so 
```

![image-20250218093009337](第二章日志分析-mysql应急响应.assets/image-20250218093009337.png)

```
flag{b1818bde4e310f3d23f1005185b973e7}
```

## 4.黑客获取的权限 flag{whoami后的值}

让我们提交黑客获取的权限，那黑客既然使用hacker进行了提权，那他在库中肯定写入了自定义函数，我们去数据库中查询一下新增的函数有那些，在次之前可以简单分析进程看看有什么线索；

![image-20250218093218835](第二章日志分析-mysql应急响应.assets/image-20250218093218835.png)

UDF 提权的典型痕迹

    异常的 .so 文件：（上面也有提到）
        检查这些目录下是否有最近创建的 .so 文件，特别是名字看起来可疑或不符合系统文件命名规范的文件。
    
    MySQL 日志：
        检查 MySQL 日志文件（如 mysql.log 或 error.log）中是否有异常的文件操作记录或 CREATE FUNCTION 语句。
    
    MySQL 函数表：
        检查 mysql.func 表中是否有异常的 UDF 函数。
    
    SELECT * FROM mysql.func;
![image-20250218093609040](第二章日志分析-mysql应急响应.assets/image-20250218093609040.png)

得到了黑客新增的函数，搜索函数

```
select sys_eval('whoami');
```

![image-20250218093702371](第二章日志分析-mysql应急响应.assets/image-20250218093702371.png)

```
flag{mysql}
```

