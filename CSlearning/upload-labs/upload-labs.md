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

总结：

### 🛠️ 文件上传漏洞测试操作表

| 测试阶段          | 绕过技术 (Technique)          | 具体操作步骤 / Payload 示例                                  | 适用场景                                   |
| :---------------- | :---------------------------- | :----------------------------------------------------------- | :----------------------------------------- |
| **1. 前端验证**   | **禁用 JS / 后缀伪造**        | 1. 浏览器禁用 JavaScript 提交。<br>2. 或者：把 `shell.php` 改名为 `shell.jpg` 上传，**Burp 抓包拦截**，将文件名改回 `shell.php` 放行。 | 网页弹窗提示，无流量产生。                 |
| **2. MIME 验证**  | **修改 Content-Type**         | 上传 `shell.php`，Burp 抓包，将 `Content-Type: application/octet-stream` 修改为 `image/jpeg` 或 `image/png`。 | 后端仅检查 HTTP 头部的 Content-Type。      |
| **3. 黑名单绕过** | **特殊后缀名**                | 尝试上传非标准后缀：`.php3`, `.php5`, `.phtml`, `.phpt` (需服务器支持解析)。 | 后端设置了不严谨的黑名单。                 |
|                   | **大小写混淆**                | 修改后缀为 `.PhP`, `.pHp`, `.Php` 等。                       | 仅 Windows 或对大小写不敏感的系统。        |
|                   | **点号绕过 (. )**             | 抓包修改文件名：`shell.php.` (Windows 会自动去除末尾的点)。  | Windows 服务器。                           |
|                   | **空格绕过 ( )**              | 抓包修改文件名：`shell.php ` (Windows 会自动去除末尾空格)。  | Windows 服务器。                           |
|                   | **点+空格+点**                | 抓包修改文件名：`shell.php. .` (经过处理后变成 `shell.php.`, 利用 Windows 特性还原为 `shell.php`)。 | 过滤逻辑为：删除末尾点->删除末尾空格。     |
|                   | **NTFS 流 (::$DATA)**         | 抓包修改文件名：`shell.php::$DATA`。                         | Windows NTFS 文件系统。                    |
|                   | **双写后缀**                  | 文件名改为 `shell.pphphp` (过滤掉中间的 `php` 后，两边拼合为 `php`)。 | 后端逻辑是 `str_replace` 且只替换一次。    |
| **4. 配置文件**   | **.htaccess (Apache)**        | 1. 先上传 `.htaccess` 文件，内容：`AddType application/x-httpd-php .jpg`<br>2. 再上传 `shell.jpg`，服务器会将其当作 PHP 解析。 | Apache 服务器，未禁止 .htaccess。          |
|                   | **.user.ini (Nginx/Apache)**  | 1. 先上传 `.user.ini`，内容：`auto_prepend_file=shell.jpg`<br>2. 再上传 `shell.jpg`<br>3. 访问同目录下任意存在的 PHP 文件，即可触发木马。 | 服务器使用 CGI/FastCGI 模式 (如 php-fpm)。 |
| **5. 路径截断**   | **00 截断 (GET)**             | 修改 URL 参数：`save_path=../upload/shell.php%00`            | PHP < 5.3.4 且 `magic_quotes_gpc=Off`。    |
|                   | **00 截断 (POST)**            | 在上传路径参数后添加 `shell.php+`，在 Hex 编辑器中将 `+` 的十六进制 `2b` 改为 `00`。 | 同上。                                     |
| **6. 内容验证**   | **文件头检查 (幻术字节)**     | 1. 在木马头部添加 `GIF89a`。<br>2. 使用 CMD 合成图片马：`copy normal.jpg /b + shell.php shell.jpg`。 | 检查文件开头 2 字节或使用 `getimagesize`。 |
|                   | **二次渲染绕过**              | 寻找 GIF/PNG 图片在渲染前后**未变动**的十六进制区域，将一句话木马插入该区域。 | 服务器使用 GD 库重绘图片。                 |
| **7. 逻辑漏洞**   | **条件竞争 (Race Condition)** | 1. Burp Intruder 开启多线程高并发上传 `shell.php`。<br>2. 同时开启多线程高并发访问 `shell.php`。<br>3. 在服务器删除文件前抢先执行并生成新马。 | 先保存文件，验证不通过再删除的逻辑。       |

1. **Check 1: 前端 JS** -> 改后缀 `.jpg` 抓包 -> 改回 `.php`。

2. **Check 2: MIME** -> 修改 `Content-Type: image/jpeg`。

3. Check 3: 后缀黑名单

    

   ->

   - 尝试 `.phtml`, `.php5`
   - 上传 `.htaccess` / `.user.ini`
   - (Win) 尝试 `空格`, `.`, `::$DATA`, `. .`, `大小写`
   - 尝试双写 `pphphp`

4. **Check 4: 后缀白名单** -> 尝试 `%00` 截断 (GET/POST)。

5. **Check 5: 内容检查** -> 加文件头 `GIF89a` / 制作图片马。

6. **Check 6: 逻辑/渲染** -> 条件竞争 / 二次渲染特定区注入。

## pikachu

这里面的三个都比较简单，一个前端一个mime一个图片马（图片马需要使用文件包含漏洞使php被解析）

## DVWA

1. low 无过滤直接上传.php
2. medium 尝试上传.php提示只能上传jpg,png，好像是白名单，尝试抓包修改content-type再上传，又成功了
3. 还是图片马

## ctfhub

1. 无验证
2. 前端验证
3. mime认证  修改content-type
4. .htaccess 文件 内容：`AddType application/x-httpd-php .jpg`
5. 00截断 get型直接在url中输入shell.php%00，拼接的时候会把文件名拼在url后面，就是shell.php%00.jpg，上传的时候检验不出来，但是服务器会把%00视为结束
6. 双写，后缀改为.pphphp
7. 文件头检查 1.在php文件开头添加GIF89a