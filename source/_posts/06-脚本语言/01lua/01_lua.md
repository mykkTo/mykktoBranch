---
title: Lua脚本语言基础
author: mykk
top: false
cover: false
toc: true
mathjax: false
tags:
  - nginx
  - lua
  - openresty
categories:
  - 计算机语言
abbrlink: fafe363a
reprintPolicy: cc_by
date: 2022-04-03 23:17:13
summary:
coverImg:
img:
password:
---



## 一、概述



### 1、概念



Lua是一种轻量、小巧的脚本语言，用标准C语言编写并以源代码形式开发。设计的目的是为了嵌入到其他应用程序中，从而为应用程序提供灵活的扩展和定制功能。

### 2、特性

跟其他语言进行比较，Lua有其自身的特点：

（1）轻量级

```
Lua用标准C语言编写并以源代码形式开发，编译后仅仅一百余千字节，可以很方便的嵌入到其他程序中。
```

（2）可扩展

```
Lua提供非常丰富易于使用的扩展接口和机制，由宿主语言(通常是C或C++)提供功能，Lua可以使用它们，就像内置的功能一样。
```

（3）支持面向过程编程和函数式编程



### 3、应用场景

Lua在不同的系统中得到大量应用，场景的应用场景如下:

游戏开发、独立应用脚本、web应用脚本、扩展和数据库插件、系统安全上。



## 二、安装

### 1、官网

在linux上安装Lua非常简单，只需要下载源码包并在终端解压、编译即可使用。

Lua的官网地址为:`https://www.lua.org`

### 2、下载并且解压

#### 1、下载

```
wget https://www.lua.org/ftp/lua-5.4.1.tar.gz
```



#### 2、安装依赖

libreadline-dev依赖包，需要通过命令来进行安装

```bash
yum install -y readline-devel
```



#### 3、解压

```bash
tar zxvf lua-5.4.1.tar.gz
```



#### 4、编译安装

```bash
cd lua-5.4.1

make linux

make install
```



#### 5、验证

```lua
lua -v
```





## 三、Lua的语法

### 1、第一个程序

#### 1、进入控制台操作

1、用 `lua`进入

2、编写输出语句，print("xxxx")

3、ctrl+D 退出控制台

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404120524.png)



#### 2、lua 文件运行

1、创建 test.lua

2、编写语句，保存

3、运行

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404121521.png)



3、lua直接运行

1、创建 test2.lua

2、编写语句的时候前缀加上声明，可以直接运行

`#!/usr/local/bin/lua`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404121846.png)

3、文件赋权限，运行

```bash
chmod 755 test2.lua

./test2.lua
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404122053.png)

### 2、注释

#### 1、单行注释的语法为：

```lua
--注释内容
```



#### 2、多行注释的语法为:

```lua
--[[
	注释内容
	注释内容
--]]
```



#### 3、取消多行

如果想取消多行注释，只需要在第一个--之前在加一个-即可，如：

```lua
---[[
	注释内容
	注释内容
--]]
```



### 3、标识符

换句话说标识符就是我们的**变量名**，Lua定义变量名以一个字母 A 到 Z 或 a 到 z 或下划线 _ 开头后加上0个或多个字母，下划线，数字（0到9）。这块建议大家最好不要使用下划线加大写字母的标识符，因为Lua的保留字也是这样定义的，容易发生冲突。注意Lua是区分大小写字母的。



简单来说参考java规范就好

### 4、关键字

下列是Lua的关键字，大家在定义常量、变量或其他用户自定义标识符都要避免使用以下这些关键字：

| and      | break | do    | else   |
| -------- | ----- | ----- | ------ |
| elseif   | end   | false | for    |
| function | if    | in    | local  |
| nil      | not   | or    | repeat |
| return   | then  | true  | until  |
| while    | goto  |       |        |

一般约定，以下划线开头连接一串大写字母的名字（比如 _VERSION）被保留用于 Lua 内部全局变量。这个也是上面我们不建议这么定义标识符的原因。



### 5、运算符

Lua中支持的运算符有算术运算符、关系运算符、逻辑运算符、其他运算符。

算术运算符:

```
+   加法
-	减法
*	乘法
/	除法
%	取余
^	乘幂
-	负号
```

例如:

```
10+20	-->30
20-10	-->10
10*20	-->200
20/10	-->2
3%2		-->1
10^2	-->100
-10		-->-10
```

关系运算符

```
==	等于
~=	不等于
>	大于
<	小于
>=	大于等于
<=	小于等于
```

例如:

```
10==10		-->true
10~=10		-->false
20>10		-->true
20<10		-->false
20>=10		-->true
20<=10		-->false
```

逻辑运算符

```
and	逻辑与	 A and B     &&   
or	逻辑或	 A or B     ||
not	逻辑非  取反，如果为true,则返回false  !
```

逻辑运算符可以作为if的判断条件，返回的结果如下:

```
A = true
B = true

A and B	-->true
A or  B -->true
not A 	-->false
------------------------------------------------------------------------------------------------
A = true
B = false

A and B	-->false
A or  B -->true
not A 	-->false
------------------------------------------------------------------------------------------------
A = false
B = true

A and B	-->false
A or  B -->true
not A 	-->true

```

其他运算符

```
..	连接两个字符串
#	一元预算法，返回字符串或表的长度
```

例如:

```
> "HELLO ".."WORLD"		-->HELLO WORLD
> #"HELLO"			-->5
```

#### 

### 6、全局变量&局部变量

在Lua语言中，全局变量无须声明即可使用。在默认情况下，变量总是认为是全局的，如果未提前赋值，默认为nil:

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404123008.png)



要想声明一个局部变量，需要使用local来声明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404123115.png)





### 7、Lua数据类型

Lua有8个数据类型

```
nil(空，无效值)
boolean(布尔，true/false)
number(数值)
string(字符串)
function(函数)
table（表）
thread(线程)
userdata（用户数据）
```

可以使用type函数测试给定变量或者的类型：

```
print(type(nil))				-->nil
print(type(true))               --> boolean
print(type(1.1*1.1))             --> number
print(type("Hello world"))      --> string
print(type(type(X)))            --> string  ，type函数返回的也是字符串类型
print(type(print))              --> function
print(type(type))               -->function
print(type{})					-->table
print(type(io.stdin))			-->userdata
```

##### nil

nil是一种只有一个nil值的类型，它的作用可以用来与其他所有值进行区分，也可以当想要移除一个变量时，只需要将该变量名赋值为nil,垃圾回收就会会释放该变量所占用的内存。

##### boolean

boolean类型具有两个值，true和false。boolean类型一般被用来做条件判断的真与假。在Lua语言中，只会将false和nil视为假，其他的都视为真，特别是在条件检测中0和空字符串都会认为是真，这个和我们熟悉的大多数语言不太一样。



##### number

在Lua5.3版本开始，Lua语言为数值格式提供了两种选择:integer(整型)和float(双精度浮点型)[和其他语言不太一样，float不代表单精度类型]。

数值常量的表示方式:

```
>4			-->4
>0.4		-->0.4
>4.75e-3	-->0.00475
>4.75e3		-->4750
```

不管是整型还是双精度浮点型，使用type()函数来取其类型，都会返回的是number

```
>type(3)	-->number
>type(3.3)	-->number
```

所以它们之间是可以相互转换的，同时，具有相同算术值的整型值和浮点型值在Lua语言中是相等的

##### string

Lua语言中的字符串即可以表示单个字符，也可以表示一整本书籍。在Lua语言中，操作100K或者1M个字母组成的字符串的程序很常见。

可以使用单引号或双引号来声明字符串

```
>a = "hello"
>b = 'world'
>print(a)	-->hello
>print(b) 	-->world
```

如果声明的字符串比较长或者有多行，则可以使用如下方式进行声明

```
html = [[
<html>
<head>
<title>Lua-string</title>
</head>
<body>
<a href="http://www.lua.org">Lua</a>
</body>
</html>
]]
```

##### table

​	table是Lua语言中最主要和强大的数据结构。使用表， Lua 语言可以以一种简单、统一且高效的方式表示数组、集合、记录和其他很多数据结构。 Lua语言中的表本质上是一种辅助数组。这种数组比Java中的数组更加灵活，可以使用数值做索引，也可以使用字符串或其他任意类型的值作索引(除nil外)。

创建表的最简单方式:

```
> a = {}
```

创建数组:

​	我们都知道数组就是相同数据类型的元素按照一定顺序排列的集合，那么使用table如何创建一个数组呢?

```
>arr = {"TOM","JERRY","ROSE"}
```

​	要想获取数组中的值，我们可以通过如下内容来获取:

```
print(arr[0])		nil
print(arr[1])		TOM
print(arr[2])		JERRY
print(arr[3])		ROSE
```

​	从上面的结果可以看出来，数组的下标默认是从1开始的。所以上述创建数组，也可以通过如下方式来创建

```
>arr = {}
>arr[1] = "TOM"
>arr[2] = "JERRY"
>arr[3] = "ROSE"
```

上面我们说过了，表的索引即可以是数字，也可以是字符串等其他的内容，所以我们也可以将索引更改为字符串来创建

```
>arr = {}
>arr["X"] = 10
>arr["Y"] = 20
>arr["Z"] = 30
```

当然，如果想要获取这些数组中的值，可以使用下面的方式

```
方式一
>print(arr["X"])
>print(arr["Y"])
>print(arr["Z"])
方式二
>print(arr.X)
>print(arr.Y)
>print(arr.Z)
```

当前table的灵活不进于此，还有更灵活的声明方式

```
>arr = {"TOM",X=10,"JERRY",Y=20,"ROSE",Z=30}
```

如何获取上面的值?

```
TOM :  arr[1]
10  :  arr["X"] | arr.X
JERRY: arr[2]
20  :  arr["Y"] | arr.Y
ROESE?
```

##### function

在 Lua语言中，函数（ Function ）是对语句和表达式进行抽象的主要方式。

定义函数的语法为:

```
function functionName(params)

end
```

函数被调用的时候，传入的参数个数与定义函数时使用的参数个数不一致的时候，Lua 语言会通过 抛弃多余参数和将不足的参数设为 nil 的方式来调整参数的个数。

```
function  f(a,b)
print(a,b)
end

f()		--> nil  nil
f(2)	--> 2 nil
f(2,6)	--> 2 6
f(2.6.8)	--> 2 6 (8被丢弃)
```

可变长参数函数

```
function add(...)
a,b,c=...
print(a)
print(b)
print(c)
end

add(1,2,3)  --> 1 2 3
```

函数返回值可以有多个，这点和Java不太一样

```
function f(a,b)
return a,b
end

x,y=f(11,22)	--> x=11,y=22	
```

##### thread

thread翻译过来是线程的意思，在Lua中，thread用来表示执行的独立线路，用来执行协同程序。

##### userdata

userdata是一种用户自定义数据，用于表示一种由应用程序或C/C++语言库所创建的类型。

### 8、Lua控制结构

Lua 语言提供了一组精简且常用的控制结构，包括用于条件执行的证 以及用于循环的 while、 repeat 和 for。 所有的控制结构语法上都有一个显式的终结符： end 用于终结 if、 for 及 while 结构， until 用于终结 repeat 结构。

##### if then elseif else

if语句先测试其条件，并根据条件是否满足执行相应的 then 部分或 else 部分。 else 部分 是可选的。

```
function testif(a)
 if a>0 then
 	print("a是正数")
 end
end

function testif(a)
 if a>0 then
 	print("a是正数")
 else
 	print("a是负数")
 end
end
```

如果要编写嵌套的 if 语句，可以使用 elseif。 它类似于在 else 后面紧跟一个if。根据传入的年龄返回不同的结果，如

```
age<=18 青少年，
age>18 , age <=45 青年
age>45 , age<=60 中年人
age>60 老年人

function show(age)
if age<=18 then
 return "青少年"
elseif age>18 and age<=45 then
 return "青年"
elseif age>45 and age<=60 then
 return "中年人"
elseif age>60 then
 return "老年人"
end
end
```

##### while循环

顾名思义，当条件为真时 while 循环会重复执行其循环体。 Lua 语言先测试 while 语句 的条件，若条件为假则循环结束；否则， Lua 会执行循环体并不断地重复这个过程。

语法：

```
while 条件 do
  循环体
end
```

例子:实现数组的循环

```
function testWhile()
 local i = 1
 while i<=10 do
  print(i)
  i=i+1
 end
end
```

##### repeat循环

顾名思义， repeat-until语句会重复执行其循环体直到条件为真时结束。 由于条件测试在循环体之后执行，所以循环体至少会执行一次。

语法

```
repeat
 循环体
 until 条件
```



```
function testRepeat()
 local i = 10
 repeat
  print(i)
  i=i-1
 until i < 1
end
```

##### for循环

数值型for循环

语法

```
for param=exp1,exp2,exp3 do
 循环体
end
```

param的值从exp1变化到exp2之前的每次循环会执行 循环体，并在每次循环结束后将步长(step)exp3增加到param上。exp3可选，如果不设置默认为1

人话：从输出1开始，每次 +10，小于100（1，11，21，...,91）

```
for i = 1,100,10 do
print(i)
end
```

泛型for循环

泛型for循环通过一个迭代器函数来遍历所有值，类似于java中的foreach语句。

语法

```
for i,v in ipairs(x) do
	循环体
end
```

i是数组索引值，v是对应索引的数组元素值，ipairs是Lua提供的一个迭代器函数，用来迭代数组，x是要遍历的数组。

例如:

```
arr = {"TOME","JERRY","ROWS","LUCY"}
for i,v in ipairs(arr) do
 print(i,v)
end
```

上述实例输出的结果为

```
1	TOM
2	JERRY
3	ROWS
4	LUCY
```

但是如果将arr的值进行修改为

```
arr = {"TOME","JERRY","ROWS",x="JACK","LUCY"}
```

同样的代码在执行的时候，就只能看到和之前一样的结果，而其中的x为JACK就无法遍历出来，缺失了数据，如果解决呢?

我们可以将迭代器函数变成pairs,如

```
for i,v in pairs(arr) do
 print(i,v)
end
```

上述实例就输出的结果为

```
1	TOM
2	JERRY
3	ROWS
4	LUCY
x	JACK
```



## 四、OpenResty



### 1、基本操作

#### 1、是什么

OpenResty是一个基于Nginx与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。所以本身OpenResty内部就已经集成了Nginx和Lua，所以我们使用起来会更加方便。



#### 2、安装

**[详情可见【OpenResty】](/posts/829b453d)**

##### 1、拉取

```docker
docker pull openresty/openresty
```



##### 2、挂载并启动

**说明下：**

- 完全基于上面的nginx配置的三个部分，唯一修改的是 `nginx.conf`配置文件，这边挂载改成了 `nginx2.conf`，用于保留之前的（自己懒而已）
- 新增了，lua文件挂载
- 修改了容器内的挂载位置，因为 openresty容器位置不一样了(外部还是不变，容器内的位置变了)

```docker
docker run -d -p 443:443 -p 80:80 --name openresty -v /root/nginx/www:/usr/local/openresty/nginx/html -v /root/nginx/conf/nginx2.conf:/usr/local/openresty/nginx/conf/nginx.conf -v /root/nginx/logs:/usr/local/openresty/nginx/logs -v  /root/nginx/lls/:/usr/local/openresty/nginx/ssl -v  /root/nginx/lua/:/usr/local/openresty/nginx/lua  openresty/openresty
```



### 2、ngx_lua的使用

使用Lua编写Nginx脚本的基本构建块是指令。指令用于指定何时运行用户Lua代码以及如何使用结果。下图显示了执行指令的顺序。 



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404144653.png)



先来解释下*的作用

```
*：无 ， 即 xxx_by_lua ,指令后面跟的是 lua指令
*:_file，即 xxx_by_lua_file 指令后面跟的是 lua文件
*:_block,即 xxx_by_lua_block 在0.9.17版后替换init_by_lua_file
```

#### init_by_lua*

```
该指令在每次Nginx重新加载配置时执行，可以用来完成一些耗时模块的加载，或者初始化一些全局配置。
```

#### init_worker_by_lua*

```
该指令用于启动一些定时任务，如心跳检查、定时拉取服务器配置等。
```

#### set_by_lua*

```
该指令只要用来做变量赋值，这个指令一次只能返回一个值，并将结果赋值给Nginx中指定的变量。
```

#### rewrite_by_lua*

```
该指令用于执行内部URL重写或者外部重定向，典型的如伪静态化URL重写，本阶段在rewrite处理阶段的最后默认执行。
```

#### access_by_lua*

```
该指令用于访问控制。例如，如果只允许内网IP访问。
```

#### content_by_lua*

```
该指令是应用最多的指令，大部分任务是在这个阶段完成的，其他的过程往往为这个阶段准备数据，正式处理基本都在本阶段。
```

#### header_filter_by_lua*

```
该指令用于设置应答消息的头部信息。
```

#### body_filter_by_lua*

```
该指令是对响应数据进行过滤，如截断、替换。
```

#### log_by_lua*

```
该指令用于在log请求处理阶段，用Lua代码处理日志，但并不替换原有log处理。
```

#### balancer_by_lua*

```
该指令主要的作用是用来实现上游服务器的负载均衡器算法
```

#### ssl_certificate_by_*

```
该指令作用在Nginx和下游服务开始一个SSL握手操作时将允许本配置项的Lua代码。
```

#### 需求:

```
http://192.168.200.133?name=张三&gender=1
Nginx接收到请求后，根据gender传入的值，如果gender传入的是1，则在页面上展示
张三先生,如果gender传入的是0，则在页面上展示张三女士,如果未传或者传入的不是1和2则在页面上展示张三。
```

实现代码

**注意**：加入配置文件的时候 #内容要去掉

```
location /getByGender {
	default_type 'text/html';
	set_by_lua $name "
		#获取请求url上的值 name，gender
		local uri_args = ngx.req.get_uri_args()
		gender = uri_args['gender']
		name = uri_args['name']
		if gender=='1' then
			return name..'先生'
		elseif gender=='0' then
			return name..'女士'
		else
			return name
		end
	";
	#解决乱码问题
	charset utf-8;
	
	return 200 $name;
}
```



![1649073290089](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1649073290089.png)







### 3、ngx_lua操作Redis

#### 1、Api及其语句说明

```lua
lua-resty-redis提供了访问Redis的详细API，包括创建对接、连接、操作、数据处理等。这些API基本上与Redis的操作一一对应。
（1）redis = require "resty.redis"
（2）new
	语法: redis,err = redis:new(),创建一个Redis对象。
（3）connect
	语法:ok,err=redis:connect(host,port[,options_table]),设置连接Redis的连接信息。
	ok:连接成功返回 1，连接失败返回nil
	err:返回对应的错误信息
（4）set_timeout
	语法: redis:set_timeout(time) ，设置请求操作Redis的超时时间。
（5）close
	语法: ok,err = redis:close(),关闭当前连接，成功返回1，失败返回nil和错误信息
（6）redis命令对应的方法
	在lua-resty-redis中，所有的Redis命令都有自己的方法，方法名字和命令名字相同，只是全部为小写。
```



#### 2、实现

```lua
location /testRedis {
    default_type "text/html";
    content_by_lua_block{
        local redis = require "resty.redis" -- 引入Redis
        local redisObj = redis:new()  --创建Redis对象
        redisObj:set_timeout(1000) --设置超时数据为1s
        local ok,err = redisObj:connect("10.0.4.7",6379) --设置redis连接信息，这边不要用127.0.0.1
        if not ok then --判断是否连接成功
         ngx.say("failed to connection redis",err)
         return
        end
        ok,err = redisObj:set("username","TOM")--存入数据
        if not ok then --判断是否存入成功
         ngx.say("failed to set username",err)
         return
        end
        local res,err = redisObj:get("username") --从redis中获取数据
        ngx.say(res)	--将数据写会消息体中
        redisObj:close()
    }
}
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404220444.png)





### 4、ngx_lua操作Mysql

#### 1、查询

##### 1、准备

```bash
driverClass=com.mysql.jdbc.Driver
url=jdbc:mysql://10.0.4.7:3306/nginx_db
username=root
password=root
```

##### 2、建库

```sql


create table users(
   id int primary key auto_increment,
   username varchar(30),
   birthday date,
   salary double
);

insert into users(id,username,birthday,salary) values(null,"TOM","1988-11-11",10000.0);
insert into users(id,username,birthday,salary) values(null,"JERRY","1989-11-11",20000.0);
insert into users(id,username,birthday,salary) values(null,"ROWS","1990-11-11",30000.0);
insert into users(id,username,birthday,salary) values(null,"LUCY","1991-11-11",40000.0);
insert into users(id,username,birthday,salary) values(null,"JACK","1992-11-11",50000.0);
```



##### 3、Api详解

```lua
（1）引入"resty.mysql"模块
	local mysql = require "resty.mysql"
（2）new
	创建一个MySQL连接对象，遇到错误时，db为nil，err为错误描述信息
	语法: db,err = mysql:new()
（3）connect
	尝试连接到一个MySQL服务器
	语法:ok,err=db:connect(options),options是一个参数的Lua表结构，里面包含数据库连接的相关信息
    host:服务器主机名或IP地址
    port:服务器监听端口，默认为3306
    user:登录的用户名
    password:登录密码
    database:使用的数据库名
（4）set_timeout
	设置子请求的超时时间(ms)，包括connect方法
	语法:db:set_timeout(time)
（5）close
	关闭当前MySQL连接并返回状态。如果成功，则返回1；如果出现任何错误，则将返回nil和错误描述。
	语法:db:close()
（6）send_query
	异步向远程MySQL发送一个查询。如果成功则返回成功发送的字节数；如果错误，则返回nil和错误描述
	语法:bytes,err=db:send_query(sql)
（7）read_result
	从MySQL服务器返回结果中读取一行数据。res返回一个描述OK包或结果集包的Lua表,语法:
	res, err, errcode, sqlstate = db:read_result() 
	res, err, errcode, sqlstate = db:read_result(rows) :rows指定返回结果集的最大值，默认为4
	如果是查询，则返回一个容纳多行的数组。每行是一个数据列的key-value对，如

    {
      {id=1,username="TOM",birthday="1988-11-11",salary=10000.0},
      {id=2,username="JERRY",birthday="1989-11-11",salary=20000.0}
    }
	如果是增删改，则返回类上如下数据
    {
    	insert_id = 0,
    	server_status=2,
    	warning_count=1,
    	affected_rows=2,
    	message=nil
    }
	返回值:
		res:操作的结果集
		err:错误信息
		errcode:MySQL的错误码，比如1064
		sqlstate:返回由5个字符组成的标准SQL错误码，比如42000

```



##### 4、实现

```lua
location /mysqlSearch{
    default_type 'text/html';
    content_by_lua_block{
        local mysql = require "resty.mysql"
        local db = mysql:new()
        local ok,err = db:connect{
            host="10.0.4.7", #不要用127.0.0.1
            port=3306,
            user="root",
            password="root",
            database="lua_db"
        }
        db:set_timeout(1000)

        db:send_query("select * from users where id =1")
        local res,err,errcode,sqlstate = db:read_result()
        ngx.say(res[1].id..","..res[1].username..","..res[1].birthday..","..res[1].salary)
    	db:close()
    }
}
```





##### 5、:baby_chick:优化:

```lua
location /mysqlSearch {
    default_type 'text/html';
    content_by_lua_block{
        local mysql = require "resty.mysql"
        local db = mysql:new()
        local ok,err = db:connect{
            host="10.0.4.7",
            port=3306,
            user="root",
            password="root",
            database="lua_db"
        }
        db:set_timeout(1000)
        local uri_args = ngx.req.get_uri_args()
        local reqId = uri_args['id']
        db:send_query("select * from users where id ="..reqId)
        local res,err,errcode,sqlstate = db:read_result()
        ngx.say(res[1].id..","..res[1].username..","..res[1].birthday..","..res[1].salary)
        db:close()
    }
}
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404233016.png)





##### 6、:artificial_satellite:lua-cjson

read_result()得到的结果res都是table类型，要想在页面上展示，就必须知道table的具体数据结构才能进行遍历获取。处理起来比较麻烦，接下来我们介绍一种简单方式cjson，使用它就可以将table类型的数据转换成json字符串，把json字符串展示在页面上即可。



步骤一：引入cjson

```
local cjson = require "cjson"
```

步骤二：调用cjson的encode方法进行类型转换

```
cjson.encode(res) 
```



优化代码：

```lua
location /mysqlSearch {
    default_type 'text/html';
    content_by_lua_block{
        local mysql = require "resty.mysql"
        local db = mysql:new()
        local cjson = require "cjson"
        local ok,err = db:connect{
            host="10.0.4.7",
            port=3306,
            user="root",
            password="root",
            database="lua_db"
        }
        db:set_timeout(1000)
        local uri_args = ngx.req.get_uri_args()
        local reqId = uri_args['id']
        db:send_query("select * from users where id ="..reqId)
        local res,err,errcode,sqlstate = db:read_result()
        ngx.say(cjson.encode(res))
         for i,v in ipairs(res) do
       ngx.say(v.id..","..v.username..","..v.birthday..","..v.salary)
        end
        db:close()
    }
}
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404234615.png)





#### 2、增删改

```lua
location /mysql {
    default_type 'text/html';
    content_by_lua_block{
        local mysql = require "resty.mysql"
        local db = mysql:new()
        local cjson = require "cjson"
        local ok,err = db:connect{
            host="10.0.4.7",
            port=3306,
            user="root",
            password="root",
            database="lua_db"
        }
        db:set_timeout(1000)
        local uri_args = ngx.req.get_uri_args()
        local reqId = uri_args['id']
        local reqType = uri_args['reqType']
        db:send_query("select * from users where id ="..reqId)
        if reqType == 'search' then 
        	local res,err,errcode,sqlstate = db:read_result()
       	    ngx.say(cjson.encode(res))
            for i,v in ipairs(res) do
    	    ngx.say(v.id..","..v.username..","..v.birthday..","..v.salary)
     	    end
        elseif reqType == 'add'  then
            local res,err,errcode,sqlstate = db:query("insert into users(id,username,birthday,salary) values(6,'zhangsan','2023-11-11',32222.0)")
        elseif reqType == 'update' then 
            local res,err,errcode,sqlstate = db:query("update users set username='lisi' where id = 6")
        elseif reqType == 'delete'  then
            local res,err,errcode,sqlstate = db:query("delete from users where id = 6")
        end    
        db:close()
    }
}
```



### 5、Rdis缓存预热

使用ngx_lua模块完成Redis缓存预热。

步骤: 

（1）先得有一张表(users)

（2）浏览器输入如下地址

```
http://10.0.4.7?username=TOM
```

（3）从表中查询出符合条件的记录，此时获取的结果为table类型

（4）使用cjson将table数据转换成json字符串

（5）将查询的结果数据存入Redis中



```lua
init_by_lua_block{

	redis = require "resty.redis"
    mysql = require "resty.mysql"
    cjson = require "cjson"
}
location /redisPreheat{
			default_type "text/html";
			content_by_lua_block{
				
				--获取请求的参数username
				local param = ngx.req.get_uri_args()["username"]
				--建立mysql数据库的连接
				local db = mysql:new()
				local ok,err = db:connect{
					host="10.0.4.7",
					port=3306,
					user="root",
					password="root",
					database="lua_db"
				}
				if not ok then
				 ngx.say("failed connect to mysql:",err)
				 return
				end
				--设置连接超时时间
				db:set_timeout(1000)
				--查询数据
				local sql = "";
				if not param then
					sql="select * from users"
				else
					sql="select * from users where username=".."'"..param.."'"
				end
				local res,err,errcode,sqlstate=db:query(sql)
				if not res then
				 ngx.say("failed to query from mysql:",err)
				 return
				end
				--连接redis
				local rd = redis:new()
				ok,err = rd:connect("10.0.4.7",6379)
				if not ok then
				 ngx.say("failed to connect to redis:",err)
				 return
				end
				rd:set_timeout(1000)
				--循环遍历数据
				for i,v in ipairs(res) do
				 rd:set("user_"..v.username,cjson.encode(v))
				end
				ngx.say("success")
				rd:close()
				db:close()
			}
		}
```







## 发现

`:`  + 英文字母有小图标

`:c` --->  :alarm_clock::baby_chick:





关于撤回，vim

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220404233431.png)