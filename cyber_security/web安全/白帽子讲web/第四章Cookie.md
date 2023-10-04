# 第四章
# Cookie和会话简介
用户通过账号密码登陆，服务端创建会话生成随机SessionID，通过Cookie字段将SessionID返回给客户端，客户端每次访问服务器就会带上这个Cookie，服务器根据SessionID查找用户会话。
Cookie可以通过后端代码设置也可以通过前端JavaScript代码设置
后端代码通过框架设置Cookie时，会在HTTP的响应头中包含以一个Set-Cookie字段，用来设置Cookie
通过JavaScript代码设置时，会在HTTP响应体的JavaScript代码中包含document.cookie字段来设置cookie。
经过浏览器渲染后用户的Cookie就会被浏览器存储。
## 第一方Cookie和第三方Cookie
第一方Cookie是指用户当前访问的网站直接植入的Cookie，通常是网站用于正常功能的Cookie，用于记住用户偏好设置
第三方Cookie是由第三方网站植入的Cookie用于追踪访问者，实现个性化广告投放
是针对网站来说的第一方和第三方
## Cookie属性
Cookie有多个属性，在服务端可以设置Cookie时可以设置相应的属性值
### Dmain属性
用于指定Cookie在哪些域名中生效，即访问哪些域名时浏览器会发送这个Cookie，也决定了哪些域名的网页可以通过JavaScript访问这个Cookie。带"."其自域名都会有效，否则只对本域名生效
例如：
```
Dmain=.baidu.com
Dmain=baidu.com
```
通过服务端的Set-Cookie或者前端JavaScript写入Cookie时，Dmain的值只能为当前网页域名的父域名或本域名，不能为其子域名
不指定Dmain属性时，Cookie的生效范围仅限于当前域名（host头指定的域名），被称为Host-Only Cookie
目前主流浏览器都遵循RFC6265规范，Dmain属性值没有“.”，其子域名也会生效 ，因此不用指定Dmain属性更为安全。
Dmain属性不包含端口信息，同一域名同时在不同端口运行了多个web应用，使用 Cookie存储重要数据时也需要评估每个应用的安全性。
其他子域名的应用或其他端口的应用除了可以读取到当前应用的Cookie，也能写入特定名称的Cookie，从而干扰当前应用，让其读取到错误的Cookie内容。
### Path属性
Path用于指定Cookie的生效路径，只有当访问这个路径或其子路径时，浏览器才会发送这个Cookie，不设置则默认为当前页面所在的路径。
不能通过path属性来做安全隔离，iframe标签能嵌入另一页面进行读取Cookie(需要父页面进行授权，同源则不需授权)
例如：
```
<iframe id="test" src="/admin/" width="0" height="0"></iframe>
<script>
window.onload = function(){
    alert(document.getElementById('test').contentDocument.cookie);
}
</script>    
```
在同一域名下不同路径下运行不同应用无法做到安全隔离，应该将其部署到不同域名下，通过同源策略保证Cookie的安全
### Expires属性
该属性可以设置Cookie的有效期，浏览器会在Cookie到期后自动将其删除。没有指定该属性则是“临时Cookie”或者“会话Cookie”，关掉浏览器后会自动删除。
浏览器对每个站点都有最大Cookie数量限制，超过就会删除旧的Cookie。
如果存在Cookie注入漏洞（如何CRLF注入漏洞），攻击者就可以植入多个漏洞，导致受害者的Cookie被挤掉
#### CRLF漏洞操纵Cookie
有参数入口，会将某些数据存入响应的字段中或者存储到数据库中，绕过过滤后，就可以将利用CRLF(%0D%0A)对数据包中的字段进行重写覆盖，添加字段，注入恶意代码(根据字段的不同能够达到的目的也不同)


### HttpOnly属性
HttpOnly属性的作用是让Cookie只能用于HTTP/HTTPS传输，不允许客户端JavaScript读取。一定程度上减少了XSS漏洞带来的危害
某些服务端框架的调试或报错信息会展示HTTP请求头的内容，PHP中使用phpinfo()也会展示请求头信息，使Cookie泄漏到前端页面。如果存在XSS漏洞，还是可以通过JavaScript获取带有HttpOnly属性的Cookie。
XST攻击，同时利用TRACE方法和XSS漏洞可获取带有HttpOnly属性的Cookie。在存在XSS漏洞的应用中通过XMLHttpRequest向服务发起TRACE请求，即可获取Cookie。目前新版浏览器基本上都不再支持在XMLHttpRequest中使用TRACE方法，而且接受TRACE方法的浏览器也不多
在Apache HTTP Server 2.2.x版本中，当HTTP请求头的长度超出所允许的最大长度时，服务器会返回一个400错误页面，以及全部的请求头信息。在存在XSS漏洞的应用中，通过XSS生成一个超长的Cookie，再通过XMLHttpRequest访问服务端，就可以读取带有HttpOnly的Cookie
### Secure属性
Secure属性设置后，Cookie只会在HTTPS请求中被发送给服务器，确保Cookie的传输安全
在客户端通过JavaScript或者服务端Set-Cookie设置Cookie时，如果当前网站使用的是HTTP协议，带有Secure属性的Cookie会写入失败。
### SameSite属性
SameSite用来限制Cookie能否跨站。有三个值None，LAX，Strict
1、None
不做限制，任何情况下都会发送Cookie。但是当SameSite被设置为None时，会要求Cookie带上Secure属性，只能通过HTTPS发送
2、LAX
普通跨站请求不会发送Cookie，导航到其他网站(点击链接)时会发送Cookie。跨站点提交表单场景中，只有GET方法会带上Cookie，POST方法不会带Cookie没有指定SameSite属性时，默认为SameSite=Lax
3、Strict
完全禁止跨站请求发送Cookie，只有当请求站点与浏览器地址栏中的URL域名属于同一个站点（“第一方Cookie”）时才会发送Cookie
SameSite在某些场景下会阻止跨站发送Cookie，SameSite一定程度上限制了CSRF攻击。

除了影响Cookie的发送，也会影响Cookie的写入。Cookie的写入分为前端JavaScript写入和服务端Set-Cookie写入。  
如果将SameSite属性设置为None，并加上 Secure属性，就可以成功写入Cookie
服务端的Set-Cookie策略与前端写入是一致的，跨站点Cookie的SameSite属性为Lax时无法写入。

此处的同站点并非同源策略的同源，没有端口限制，也没有限定域名完全一样。
属于公共后缀列表的会被认为是一个顶级域名如`github.io`
```https://publicsuffix.org```
该网址能查询到公共后缀列表名单
### SameParty属性
为了解决SameSite=Lax严格过滤带来的不便，出现了SameParty属性。
网站将可信网站集合定义在/.well-known/first-party-set文件中，当一个网站的页面要请求另一个网站资源时，浏览器会检测这两个网站是否处于同一个First-Party Sets，如果是，则将带上第三方Cookie
## 安全使用Cookie
### 正确设置属性值
+ HTTPS应用中，将Cookie设置为Secure。 Cookie:PHPSessionID=;Secure
+ 没必要让子域名读取Cookie时，不要设置Dmain属性值
+ 大部分场景不需要客户端JavaScript读取Cookie。重要Cookie设置HttpOnly属性
+ 如不需要被其他网站引用，与会话有关的Cookie将其SameSite属性设置为LAX，以减少CSRF攻击
### Cookie前缀
防止子域名的安全漏洞影响到所有子域名站点，因此有了Cookie前缀
web应用可以为Cookie名称添加特定的前缀，告诉浏览器这些Cookie应该满足特定的要求。
#### __Host- 
该前缀需满足四个条件，浏览器才会接受这个Cookie。
+ 带有Secure属性
+ 不包含Dmain属性
+ Path属性为“/”
+ 当前为HTTPS连接
例如：
>document.cookie='__Host-ID=111;Secure;Path=/'
 #### __Secure-
 满足两个条件：
 + 带有Secure属性
 + 当前为HTTPS连接
 
### 保密性和完整性
将数据保存在服务端，对存储在Cookie中的数据进行加密或签名
尽量将不同应用 部署在不同子域名下，并使用Cookie前缀将Cookie与域名绑定
## 会话安全
会话的本质是标识不同的访问者，并记录他们的状态
### 会话管理
#### 会话ID（SessionID）的随机性
防止攻击者对SessionID进行猜解，需要增加其随机性以及足够的长度
#### 过期和失效
由于Expires属性可以在前端更改，会话的超时机制应该有服务端来实现，在修改密码、账号挂失等业务场景中，也应该在服务端使该账号相关会话数据失效
#### 绑定客户端
将会话与客户端进行绑定会更安全
在web应用中可以将浏览器的User-Agent与会话绑定，该绑定关系比较弱。
在移动App中有更好的唯一标识客户端的机制，可实现会话与设备之间更强的绑定关系
更安全的做法是将会话与IP地址进行绑定，但会牺牲用户体验
#### 安全传输
现代web应用都是将SessionID写入Cookie中，大部分情况下建议开启HttpOnly和Secure属性
老的Web应用通过URL参数传递SessionID，很容易造成泄漏。

+ 跳转到站外链接或者加载其他站点资源时，会通过Referer泄漏出去
+ 浏览器历史记录会被保存在URL中，现代浏览器还支持设备之间同步历史记录
+ 服务端日志会记录URL，这些日志可能会被泄漏出去
+ 可能被用户无意间分享出去
#### 客户端存储会话
除了前面的将会话存储在服务端，客户端只保存一个很短的SessionID。也有将会话存储在客户端，最典型的就是JWT(JSON Web Token)用于管理会话。
JWT本质上是带有签名的JSON数据，必要时可进行加密。用户登录后，服务端将会话信息生成为一个带签名的JWT写入客户端，客户端每次访问都带上JWT，服务端验证签名的有效性，并提取会话相关信息。
优点：服务端无状态，非常容易扩容，后端有web服务器集群时不需要在多台服务器之间做会话同步
缺点：
对已签发的会话Token无法进行吊销，这将导致很多账号安全功能无法实施，如退出账号、修改密码等。为此需在签发Token时加入一个恰当时常的有效期。
密钥泄漏时攻击者就可以签发任意的JWT，使所有账号都受到威胁。
### 固定会话攻击
攻击者诱导用户使用攻击者指定的SessionID，当受害者登录成功后，这个SessionID就关联了受害者的身份，攻击者就拥有了受害者在目标网站上的身份
关键步骤：让受害者使用攻击者指定的SessionID
实现方法：
+ 一个恶意的子域名应用可以设置一个在其他子域名也生效的Cookie，恶意应用就可以设置在其他应用的SessionID。同一域名在不同端口运行了多个应用，或在一个域名的不同路径下运行了不同应用，恶意应用都可以写入一个在其他应用中也有效的Cookie。
+ 一些应用中，允许通过 URL参数来指定SessionID，攻击者诱导受害者点击一个带有SessionID参数的链接
+ 在存在XSS漏洞或CRLF注入漏洞的应用中，攻击者也可能给受害者植入特定的Cookie来实施固定会话攻击
防御：
当用户的登录状态变化后，服务端应该为用户生成一个新的SessionID