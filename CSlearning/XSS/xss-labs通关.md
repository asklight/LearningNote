# xss原理

### 反射型 

恶意脚本作为请求的一部分发送到服务器，服务器将其”反射“回响应中，受害者点击恶意链接后触发（服务端把用户输入反射到页面）

### 存储型

恶意脚本被存储在服务器上，所有访问该页面的用户都会受影响

![image-20251223224822561](file:///C:/Users/GWG/AppData/Roaming/Typora/typora-user-images/image-20251223224822561.png?lastModify=1766582526)

例如：发送评论、弹幕

### DOM型

完全发生在浏览器端，不依赖服务端的响应输出，而是通过篡改页面的DOM结构，利用前端js对dom的操作漏洞，注入恶意脚本。

# xss练习

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

4. DOM型xss 查看前端代码，发现里面是个a标签，尝试”发现被转义了，就换用单引号’成功闭合，使用事件属性`1' onclick="console.log(1)"`,发现成功注入

5. DOM型xss-x 跟上面的差不多，`1' onmouseover="console.log(0)" ' `

6. xss盲打  这是一个留言板，直接提交一个a标签，`<a onclick="alert()">111</a>`在管理员界面发现可用

7. xss之过滤 这个输入的内容被放入了p标签中，直接使用</p>闭合标签，再加入一个a标签

8. xss之htmlspecialchars 这个方法把双引号"转义了，用单引号闭合

9. xss之href输出 这个单引号双引号都闭合不了，尝试js伪协议：`javascript:alert()`变成

   `<a href="javascript:alert()">1111</a>`

10. xss之js输出 这个我感觉我被js判断那里迷惑了，这个使用get获取的message参数，同时还获取了submit和value参数但并没有渲染到前端，我一直在这里下手什么都做不出；
  实际上message参数在`<script></script>`标签里面，直接闭合再写个a标签就行了，跟第七题一样。payload:`'</script><a onclick="alert()">11</a>`

## DVWA xss

### DOM

1. low 直接url中输入`<script>alert()</script>`

2. medium 发现输入内容在select标签的option标签里面，直接闭合加a标签

   payload:`></option></select><a onclick=alert()></a>`

   (这个我做的时候想闭合option里面的value，但看到”变成%22，‘变成%27，空格变成%20，我还以为做了什么过滤呢，原来没影响，我闭合value后用onclick没效果是因为option标签没有onclick属性)
   
3. high 查看源码发现用的是switch case,这下必须得包含那几个选项了，然而form表单提交的数据想经过js过滤，就可以通过注释绕过，payload:`#<script>alert()</script>`

### Reflected

1. low F12发现输入内容在<pre>标签中，直接闭合加a标签

   payload:`</pre><a onclick=alert()>11</a>`

2. medium 跟low没区别，看了看源码原来是把<script>替换成了空，这样的话可以双写绕过

3. high 依旧可以使用`</pre><a onclick=alert()>11</a>`

### Stored

1. low 同样是放在br标签里，直接闭合加a标签，payload:`<br><a onclick=alert()>11</a>`
2. medium 这个message我怎么也闭合不了，`''<br>`会被替换，就尝试闭合name，name在前端有字数限制，修改后构造payload:`"</div><a onclick=alert()>11</a>`
3. high 修改前端，用name闭合依旧可以(我也不知道修改前端算不算作弊？)



## ctfhub

1.空格过滤：想要构造标签`<a onclick=alert()>111</a>`,但是空格被过滤，可使用回车符等替换，制表符%09，换行符%0A，回车符%0D，试了一下发现不行，使用/分割元素可行

做了几个终于明白了，这个需要获得flag，就得让bot机器人访问你构造的url(这个url中提供的是xss平台的图片)，然后你通过xss平台接受cookie