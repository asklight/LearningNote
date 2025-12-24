xss

经常使用的html和js

```html
<a href="www.baidu.com">链接文本</a>
<img src="url">
<a onclick="didid">点击文本</a>
<script>alert(5+6)</script>
```

html转义字符

![image-20251223232913717](C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251223232913717.png)

1.无转义

2.用“>闭合

3.html实体转义的绕过：单引号不会被转义  .htmlspecialchars($str)，

使用`.htmlspecialchars($str，ENT_QUOTES)`单引号就会被转义

4.<>被替换掉了，没法构造标签，就使用属性来绕过 ” onclick=console.log("1111") "

5.script和onclick都被加入了下划线，就使用js伪协议 

 `"><a href=javascript:alert()>点击获得100万</a> "`

6.script on href data src 都被添加了下划线  使用大小写绕过（第五关不用是因为他把字母都变小写了）

7.从开始到结束只替换了一次，而且替换成了空  就用双写绕过

`"/><SCRscriptIPT>alert()</SCRscriptIPT><"`

8.双引号转义，被输入的内容被放在href中，可以用javascript伪协议，但script被加了一个下划线，可以使用16进制插入伪协议

9.对第八关加了一个判断，判断是否含有http://(这个需要慢慢试)，把http://放在注释里

10.输入框被隐藏，试出可以传递参数的值，使用type="text"显示出来，再用payload

11.通过请求头控制xss   使用HackBar构造referer

12





### 反射型

例如：url

### 存储型

1. 黑客脚本注入
2. 无过滤保存至数据库
3. 用户访问
4. 返回页面
5. 脚本自动执行

![image-20251223224822561](C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251223224822561.png)

例如：发送评论、弹幕

### DOM型

也是url但是部分url未被发送给服务器

![image-20251223225049547](C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251223225049547.png)