# 第七章
# 跨站请求伪造(CSRF)
CSRF全称Cross Site Request Forgery，是web安全中最容易被忽视的一种攻击方式
## CSRF简介
在网页上执行的操作，实际上是向服务器发送HTTP请求来实现的。
攻击流程：
用户登录网站，网站给用户分配凭证，攻击者诱骗用户访问恶意网站，用户带着凭证发起攻击者自定义的请求。
## CSRF详解
### CSRF的本质
CSRF攻击本质上是借助用户的凭证，以用户的身份去执行特定操作。在用户访问攻击者构造的恶意页面是，如果此时浏览器访问第三方站点(带有CSRF漏洞)带上了第三方的Cookie；但是在部分场景中，在存在CSRF漏洞的网站上就可以向当前站点发起请求，虽然此时并没有"跨站点"，此时也不存在第三方Cookie，但也叫CSRF攻击。
在web应用中一般使用Cookie作为身份凭证，但是如果一个网站是基于源IP地址进行认证和授权的。那么来自内网IP地址的访问都认为是可信的，那么外部攻击者可以诱使一个内部员工去访问一个恶意页面，恶意页面会对内网应用发起请求，这也能实施CSRF攻击。
从外部利用CSRF攻击内网应用的场景是路由器的CSRF。因为路由器一般只把管理功能开放给内部网络，所以路由器厂商并不重视其安全防御功能，而且很多路由器都有默认的IP地址及控制台密码 ，这就方便了攻击这实施CSRF攻击。攻击者可以利用路由器漏洞修改路由器配置，如更改DNS地址，以实施更多攻击。Chrome的私有网络访问策略，就是为了缓解这种攻击。
在实施CSRF攻击的过程中，因为同源策略的限制，攻击者仅能让受害者发起请求，无法获取请求的返回内容。攻击者能够获得的结果是由应用提供的功能决定的。
### GET和POST请求
大多数CSRF发起攻击时，使用HTML标签都是`<img>、<iframe>、<script>`等带有`src`属性的标签，这类标签只能发起一次GET请求，而不是POST请求。
使用POST请求跨站提交表单也很容易，只需要构造一个表单并填好参数，再使用JavaScript代码自动提交。例如：
```
<form id="myForm" acrion="http://example.com/follow" method="POST">
    <input type="hidden" name ="id" value="1234">
</form>
<script>
    document.getElementById('myForm').submit();
</script>
```
受害者在打开恶意网页时自动提交表单，执行CSRF攻击。
如果使用POST请求提交JSON数据，也可以实现CSRF攻击：
`{"id":123}`
既符合name=value形式，又是合法的JSON数据。等号左边的内容属于input标签的name属性，等号右边的内容用于input标签的value属性。
构造的表单Payload如下：
```
<form id="myForm" method="POST" enctype="text/plain">
    <input type="hidden" name="{&quot;id&quot;:1234,&quot;dummy" value="&quot;:0}"> 
</form>
<script>
    document.getElementById('myForm').submit();
</script>
```
通过name和value属性在表单提交时会对其值用`=`进行连接。因此，将JSON数据进行拆分到name和value属性的值中去，使其传输的格式符合JSON格式。
在HTTP中携带的数据如下：
`{"id":123,"dummy=":0}`
通过id的值123来实现CSRF攻击。
传输XML数据同理，因为XML数据自带=因此无需插入无效内容：
```
<input type="hidden" name="&lt;?xml version" value="&quot;1.0&quot;?&gt;&lt;id&gt; 123&lt;/id&gt;">
```
在HTTP数据包中的数据如下：
`<?xml version="1.0"?><id>123</id>`
### CSRF蠕虫
实现蠕虫传播需要利用漏洞携带攻击代码自动扩散，如果CSRF漏洞满足两个条件就可以实现自动扩散

+ 在应用中发送其他用户可见内容的功能存在CSRF漏洞，发私信、个人状态、博客等
+ 发送的内容可嵌入链接，诱导用户点击，以触发CSRF漏洞
如一个可以发博客的功能存在CSRF漏洞，博客中又能嵌入链接，那么攻击者可以构造恶意页面http://evil.site/csrf.html，如下：
```
<form id="myForm" action="http://example.com/new_blog" method="POST">
    <input name="content" value='<a href="http://evil.site//csrf.html">click me </a>'>
</form>
<script>
    document.getElementById('myForm').submit();
</script>
``` 
更特殊的场景，比如应用中有个发布内容的功能存在CSRF漏洞，并且这个内容可以嵌入URL，用户访问应用时该URL会自动加载，这样无须搭建恶意页面也无须用户点击恶意链接就能实现CSRF蠕虫。条件更苛刻，效率更高。
前面介绍过有些论坛允许用户设置一个图片的URL作为头像，如果设置头像功能存在CSRF漏洞，攻击者就可以设置自己头像URL为"更新头像的URL"如：
`http://example.com/updateAvatar?url=[IMAGE_URL]`
所有访问了攻击者页面的用户，都会将自己头像的URL设置为攻击者指定的URL。
## 防御CSRF攻击
CSRF攻击有标准的防御方案，能够彻底解决CSRF漏洞。
### 验证码
验证码强制用户必须与应用进行交互才能完成请求，能有效避免CSRF攻击
### Referer校验
Referer校验在互联网中最常见的应用就是"防止图片盗链"，同理，Referer校验也可以用来检查请求是否来自合法的" 源"。
互联网应用，页面与页面之间都有一定的逻辑关系，因此每个正常请求的Referer会有一定规律，可以通过规律来识别是否是CSRF攻击。
Referer校验的缺陷是，服务器并非什么时候都能获取Referer头，不少保护个人隐私的浏览器扩展会限制Referer的发送。某些情况下浏览器本身就不会发送Referer，比如从HTTPS跳转到HTTP，出于保护全链路数据的考虑，浏览器不会发送Referer到明文的HTTP请求中。
如果应用忽略了Referer为空的情况，对这种请求不做拦截，那么CSRF防御功能将完全失效。攻击者通过iframe加载Data协议的URL，或者设置Referrer-Policy，都可以不发送Referer。
另一个问题是，web应用功能复杂，开发者如果对每个功能都校验完整的Referer URL，代码将很难维护。所以大部分校验Referer的CSRF防御中都只校验Referer中的域名。
Referer检验作为监控CSRF攻击倒是可行，不能作为主要手段来防御CSRF。
### Cookie的SameSite属性
Cookie的SameSite属性，可以控制Cookie在跨站点请求是否生效。
一般情况，不会将应用中的Cookie的SameSite设置为Strict，用户体验不好，而且，如果允许站内发送链接，在站内点击链接时会携带Cookie，也防御不了GET请求的CSRF攻击。
当SameSite被设置为LAX时，网站导航跳转和GET请求的表单都会携带Cookie，如果应用中的重要功能是以GET方式进行操作的，那么也存在CSRF漏洞。
总的来说不能依靠该属性进行防御
## Anti-SCRF Token
业界针对CSRF攻击的防御，一致做法就是使用一个随机的Token
### 原理
CSRF攻击能成功，原因是所有重要操作都能被攻击者猜到。攻击者如果能猜出URL的所有参数与参数值，就能成功构造一个伪造的请求，反之，将无法攻击成功。
一个解决方案是：参数加密或者签名，让攻击者无法生成正确的参数值。
比如：一个删除操作的URL是：
`http://host/path/delete?username=abc&item=123`
将其中username参数改为哈希值：
`http://host/path/delete?username=md5(salt+abc)&item=123`
服务器可以从Session或Cookie中获取"username=abc"的值，再结合salt对整个请求进行验证，正常请求就会被认为是合法的。
但是也存在一些问题：加密或签名后的URL将非常难读，对用户不友好。如果每次加密参数都会变，则某些URL将无法被用户收藏。普通参数也被加密或哈希，将会给数据分析工作带来很大困扰。
一个更加通用的方案就是Anti-CSRF Token，在URL中保持原参数不变，新增一个参数Token。这个Token是随机不可预测的。
Token的值 需要足够随机，必须采用足够安全的随机数生成算法，或者采用真随机数生成器来生成。Token应为一个"秘密"，为用户和服务器持有，不能被第三者知晓。实际应用时，可以将Token放在用户的Session中，或者浏览器的Cookie中。
Token也需要被放在表单中，在提交请求时，服务器只需验证表单中的Token与用户的Session中的Token是否一致，一致则认为合法，不一致则不合法。
JavaScript代码发起请求时虽然也可以往POST数据中添加Token，但是需要每次都添加，很麻烦。更常见的做法是在HTTP头中插入Token字段，如在jQuery中可以注册所有请求发送Token头，而无需每个请求都插入Token：
```
<meta content="{{csrf_token}}" name="csrf-token"/>
<script type="text/javascript">
    var csrf_token = document.querySelector('meta[name="csrf-token"]').content;
    $.ajaxSetup({
        if(!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(setting.type)&& !this.crossDomain){ //增则匹配条件
            xhr.setRequestHeader("X-CSRFToken",csrf_token);
        }
    });
</script>
```
可以在渲染HTML页面时将Token放在\<meta>标签中，JavaScript代码要使用时再读取内容，或者读取Cookie中的Token也行。
### 使用原则
使用Anti-CSRF Token时，有值得注意事项：
+ 需要足够安全的随机数生成器Token
+ Token不是为了防止重复提交。可以在会话周期内都是用同一个Token
+ 每次操作使用一个新的Token可能会导致其他页面的表单再次提交时，会出现Token错误
+ 在web应用执行敏感操作时应使用form表单或通过AJAX以POST方式提交。
+ XSS仅能对抗CSRF，若还存在XSS漏洞则该方案会变得无效。
在现代web框架应用中，可以使用统一的中间件来实现服务端的CSRF Token校验，其基本做法时对所有POST请求做Token校验，校验通过才会进入后续业务逻辑处理。例如Flask框架中，可以简单使用CSRFProtect为整个应用做全局CSRF防御：
`from flask_wtf.csrf import CSRFProtect
csrf= CSRFProtect(app)`
然后，在渲染表单时加入Token字段：
```
<form method="post">
    {{form.csrf_token}}
</form>
 ```
 即使有这种全局校验机制，如果开发者使用不当还是会存在CSRF漏洞。最常见的案例是开发者使用GET请求来实现敏感操作，这将完全不受CSRF校验保护。另一种情况是，后端没有指明从POST请求体中获取参数。PHP中$_REQUEST就会导致能从GET中获取参数，导致受到CSRF攻击。