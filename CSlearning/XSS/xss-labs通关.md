## xss-labs

经常使用的html和js

```html
<a href="www.baidu.com">链接文本</a>
<img src="url">
<a onclick="didid">点击文本</a>
<script>alert(5+6)</script>
```

html转义字符

![image-20251223232913717](C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251223232913717.png)

1. 无转义
2. 用“>闭合
3. html实体转义的绕过：单引号不会被转义  .htmlspecialchars($str)，使用`.htmlspecialchars($str，ENT_QUOTES)`单引号就会被转义
4. <>被替换掉了，没法构造标签，就使用属性来绕过 ” onclick=console.log("1111") "
5. script和onclick都被加入了下划线,就使用js伪协议 `"><a href=javascript:alert()>点击获得100万</a> "`
6. script on href data src 都被添加了下划线  使用大小写绕过（第五关不用是因为他把字母都变小写了）
7. 从开始到结束只替换了一次,而且替换成了空,就用双写绕过`"/><SCRscriptIPT>alert()</SCRscriptIPT><"`
8. 双引号转义，被输入的内容被放在href中，可以用javascript伪协议，但script被加了一个下划线，可以使用16进制插入伪协议
9. 对第八关加了一个判断，判断是否含有http://(这个需要慢慢试)，把http://放在注释里
10. 输入框被隐藏，试出可以传递参数的值，使用type="text"显示出来，再用payload
11. 通过请求头控制xss   使用HackBar构造referer：” type="text" onclick="alert(0)" "
12. 查看源码得知通过user_agent获取字段
13. 同12关11关，只是使用了cookie
14. 图片的exif信息
15. ng-include这个意思是当html过于复杂时，可以将部分代码打包成单独文件，在ng-include来引用这个独立的html文件，简单说用于包含外部html文件，包含的内容作为指定元素的子节点，ng-clude属性的值可以是一个表达式，返回一个文件名，默认情况下，包含的文件需要包含在同一个域名下。直接把第一关包含进来 `src='level1.php?name=<a onclick=alert()>11</a>'`
16. " "被变成了&nbsp，就使用回车来代替，回车的url编码是%0a
17. 这个我自己想的是用双引号闭合构造了一个`"onclick=alert()`，但是双引号被转义了，没想到输入内容加空格直接就闭合了,它是直接取的字符串进去，之后我得多加空格`111 onclick=alert()`
18. 跟17关一模一样
19. 20都是flash相关的内容，过时了就没看

## pikachu xss练习

1. 反射型xss(get) 输入`<script>alert()</script>`测试，发现输入框限制输入字符数，修改前端限制后成功alert
2. 反射型xss(post) 这个使用弱密码试出来密码，然后一样的payload`<script>alert()</script>`
3. 存储型xss 很容易直接`<a console.log(11)>11</a>`
4. DOM型xss 



### 反射型 

恶意脚本作为请求的一部分发送到服务器，服务器将其”反射“回响应中，受害者点击恶意链接后触发

### 存储型

恶意脚本被存储在服务器上，所有访问该页面的用户都会受影响

1. 黑客脚本注入
2. 无过滤保存至数据库
3. 用户访问
4. 返回页面
5. 脚本自动执行

![image-20251223224822561](C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251223224822561.png)

例如：发送评论、弹幕

### DOM型

修改页面的dom结构实现，不依赖服务器响应，完全再客户端运行。也是url但是部分url未被发送给服务器

![image-20251223225049547](C:\Users\GWG\AppData\Roaming\Typora\typora-user-images\image-20251223225049547.png)