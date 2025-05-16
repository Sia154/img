# 第二章日志分析-apache日志分析

```
账号密码 root apacherizhi
ssh root@IP
1、提交当天访问次数最多的IP，即黑客IP：
2、黑客使用的浏览器指纹是什么，提交指纹的md5：
3、查看index.php页面被访问的次数，提交次数：
4、查看黑客IP访问了多少次，提交次数：
5、查看2023年8月03日8时这一个小时内有多少IP访问，提交次数:
```

分析 Apache 的日志，Apache+Linux 日志路径一般是以下三种：

- /var/log/apache/access.log
- /var/log/apache2/access.log
- /var/log/httpd/access.log

1、提交当天访问次数最多的IP，即黑客IP：

当天指的是2023.8.3

cat access.log* | grep "03/Aug/2023:08:" | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 10

![image-20240612085440332](第二章日志分析-apache日志分析.assets/image-20240612085440332.png)

192.168.200.2

2、黑客使用的浏览器指纹是什么，提交指纹的md5：

要求我们查找攻击者的指纹，我们可以根据 IP 在日志中进行反查，所以我们在`access.log*文件中匹配`192.168.200.2`

cat /mnt/c/Users/Anonymous/Desktop/apache2/access.log* | grep "192.168.200.2" | head -20

![image-20240612090713395](第二章日志分析-apache日志分析.assets/image-20240612090713395.png)

```
Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/115.0Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36 
两个浏览器指纹一个一个试
这两个字符串都是User Agent字符串，用于标识和描述发出网络请求的用户代理的类型和版本信息。它们通常被浏览器发送到服务器，以便服务器可以根据这些信息调整其响应
只有第2个正确
```

2d6330f380f44ac20f3a02eed0958f66

3、查看index.php页面被访问的次数，提交次数：

cat access.log* | grep "/index.php" | wc -l

![image-20240612091623821](第二章日志分析-apache日志分析.assets/image-20240612091623821.png)

27

4、查看黑客IP访问了多少次，提交次数：

cat access.log* | grep -w "192.168.200.2" | wc -l

![image-20240612092014189](第二章日志分析-apache日志分析.assets/image-20240612092014189.png)

6555

5、查看2023年8月03日8时这一个小时内有多少IP访问，提交次数:

cat access.log* | grep "03/Aug/2023:08:" | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 10

![image-20240612092104582](第二章日志分析-apache日志分析.assets/image-20240612092104582.png)

5个

5