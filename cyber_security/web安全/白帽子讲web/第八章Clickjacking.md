# 第八章
# 点击劫持
## 点击劫持简介
点击劫持是一种视觉上的欺骗手段，攻击者使用透明的、不可见的iframe，覆盖在一个网页上，诱使用户在该网页上进行操作。
例如：
```
<!DOCUTYPE html>
<html>
    <head>
        <style>
        iframe{
            position:absoulute;
            top:-800px;
            width:400px;
            height:1000px;
            z-index:999;
            opacity:0.8;
        }
        
        button{
            position:absolute;
            top:40px;
            left:40px;
            z-index:1;
            width:80px;
            heoght:30px;
        }
        </style>
    </head>
    <body>
        <iframe src="https://detail.tmall.com/item.htm?id=561011248788&shuId=3430456733"></iframe>
        <button>click me</button>
    </body>
</html>
```
因为iframe页面的z-index值大于button的z-index值所以，iframe页面会覆盖在按钮之上。
都必须使用绝对定位使用(position:absolute);z-index的值要设置得足够大；通过设置opacity来控制iframe页面的透明度，值为0时，页面完全不可见。
点击劫持攻击与CSRF攻击有异曲同工之处，都是在用户不知情的情况下诱使用户完成一些操作。
## 图片覆盖攻击(XSIO)
点击劫持的本质是一种视觉欺骗。 有学者提出了Cross Site Image Overlaying(XSIO)攻击
攻击者发布如下内容的文章：
```
<a href="https://evil.site">
<img style="postion:absolute;z-index:999;width:80px;top:6px;left:15px" src="http:expamle.com:8000/wp-content/uploads/2021/11/Doge.png">/</a>
```
点开该链接后，网站的导航栏Logo将会被覆盖，用户点击该图片将会 跳转到攻击者指定的恶意网站。
还可以将图片伪装成一个正常的链接、按钮等
由于\<img>标签在很多系统中对用户是开放的，因此在现实中有非常多的站点存在被XSIO攻击的可能性。在防御XSIO攻击时需要检查用户提交的HTML代码中，\<img>标签的style属性是否能导致图片浮出边界。
## 拖拽劫持与数据窃取
现代浏览器都支持Drag&Drop的API，拖拽使得用户操作变得简单。
拖拽劫持的思路是诱使用户从隐藏的不可见的iframe页面中"拖拽"出攻击者希望得到的数据，放到攻击者能控制的另一个页面中，从而窃取数据。
现代浏览器对拖拽劫持进行了一定防御，无法从跨域的iframe页面中拖拽出数据，但是在不同窗口之间跨域拖拽数据是可行的。
由于需要受害者开启两个窗口，所以这种攻击成功率不高。
## 其他劫持方式
在浏览器中曾有很多中劫持方式，如今都已不可用了。一种被称为Filejacking的劫持攻击乐意窃取用户本机文件。采用Webkit引擎的浏览器支持上传整个文件夹 ，攻击者构造一个"下载页面"，实际是上传页面。后来浏览器对此类攻击做了方法，在 上传文件时会提醒用户正在上传文件，有操作风险。
## 防御点击劫持
点击劫持是一种视觉上的欺骗，针对该类劫持，一般通过禁止跨域的iframe来防范的。
### Frame Busting
可以写一段JavaScript代码禁止iframe嵌套，该方法叫做Frame Busting，例如：
```
if(top.location!=loaction){
    top.location=self.location;
}
```
常见的Frame Busting有如下检测方式：
```
if(top!=self)
if(top.location!=self.location)
if(top.location!=location)
if(parent.frames.length>0)
if(window!=top)
if(window.top!==window.self)
if(parent&&parent!=window)
if(parent&&parent.frames&&parent.frames.length>0)
if((self.parent&&!(self.parent===self))&&(self.parent.frames.length!=0))
```
以往旧的浏览器常使用这些方式，但是Frame Busting也存在一些缺陷，由于是用JavaScript写的，控制能力并不是很强，很多方法都可以绕过。
此外，通过设置iframe的sandbox属性，可以限制iframe页面中的JavaScript代码执行，从而使Frame Busting失效。
### Cookie的SameSite属性
当把会话的Cookie设置成Strict或Lax模式时，iframe跨站点加载页面时就不会发送该Cookie，所以iframe中的页面将出于未登录状态，攻击者就无法实施攻击了。
### X-Frame-Options
一种更标准的方案，使用一个HTTP头X-Frame-Options
X-Frame-Option就是为解决点击劫持而生的。有三个值DENY,SAMEORIGIN,ALLOW-FROM=url
当值为DENY时，浏览器会拒绝当前页面通过iframe被加载；值为SAMEORIGIN，当前页面只能被同源的其他页面通过frame加载，嵌套的frame时也要求所有上层页面都同源；值为ALLOW-FROM=url时，则可以允许哪些URL能通过frame加载当前页面，该选项很少被使用到，大多浏览器不支持该特性。
一直是以"X-"开头，没有称为HTTP标准，而HTTP标准中主推另一个功能欠打的方法即CSP的frame-ancestors指令
### CSP:frame-ancestors 
CSP 中的frame-ancestors指令用于指示哪些资源可以加载当前页面，用法非常灵活，有如下几种：

+ host:指定的域名或者泛域名都可以加载
+ 'self':当前URL同源的网站可以加载
+ 'none':任何资源都不允许加载
例如：
```
Content-Security-Policy:frame-ancestors 'none';
Content-Security-Policy:frame-ancestors 'self' https://example.com;
```
指定了当前源以及https://example.com的页面作为当前页面的父页面