# 第五章
# 深入同源策略
## 同源策略详解（SOP）
在web应用中，前端网页的本质是web引用的UI（用户交互界面），它与web服务端应用是一个整体，二者共同组成了web引用。
web应用的特殊之处在于，当用户访问A和B两个完全不相关的web应用时，它们的前端页面运行在同一个浏览器中，或者A的页面内部嵌入了B的页面，而且它们的前端页面还能给对方的服务器发送HTTP请求。同源策略保证二者的数据隔离的安全性。
同源策略的目的是防止它们读取用户在当前网站上的数据，或以用户的身份向当前网站发起操作。
"同源"实质上是一种安全区域的划分。A和B同源表示它们处于一个应用中，可以相互信任。对于不同源的两个网页，浏览器会限制它们无法访问对方的内部数据。
"源"是由（protocol，host，port）三元组定义的，必须协议、主机名、端口号都相同才是同源的。
加载资源和读取资源的区别

+ 加载资源是将静态资源从服务端下载到客户端，浏览器请求资源，服务器返回资源，浏览器再将资源加载到内存中进行解析和渲染
+ 读取资源通常是指从已加载到客户端浏览器中的资源中读取数据或信息。这是可以通过JavaScript代码访问和操纵DOM元素，读取表单输入值，发送Ajax请求获取服务器数据等。

同源策略是限制JavaScript代码读取另一个域的数据。SOP是允许一个站点加载另一个域的资源（CSS加载字体文件除外），而不允许该站点当前页面的JavaScript代码进行读取。网页跨域嵌入资源，甚至跨域“写”操作都是被允许的，不允许“读”操作。
SOP实现了

+ 能够加载B站点的JavaScript文件，但无法获取源代码
+ 加载B站点的CSS样式文件，无法获取该文件内容
+ 加载B站点的图像文件，无法获取图像的像素值
+ 能通过iframe嵌入来自B站点的网页，无法读取B网页的原始内容

从而达到数据隔离的目的。
注意：JavaScript代码执行环境是当前页面URL所在的域，与JavaScript代码是直接内嵌在HTML页面中还是从外部加载的没有关系，所以嵌入的外部JavaScript代码在执行时可以读取当前域的数据。
同源策略中还有很多安全机制保证了站点的隔离，不仅仅是限制JavaScript跨域访问服务端资源。在浏览器中，两个不同源的网页被同时打开，或者一个页面通过iframe嵌套在另一个页面中，它们之间也不能通过JavaScript相互访问。
本地的存储localStorage也受同源策略影响，一个源的JavaScript代码不能跨域读取另一个源的localStorage。Cookie较为特殊。
## 跨域DOM互访问
同源策略有严格的跨域访问限制，但很多场景需要网站跨域。有一种方法前端两个页面之间DOM互访问，不涉及服务端。
### 子域名应用互访问
如果一个公司的域名存在多个子域名，不同子域名的网页需要进行交互，该场景下同源策略限制可以放宽。通过修改两个子域名页面的document.domain属性使其在同一个源内，从而实现DOM互访。
document.domain是JavaScript的一个属性，用于获取或设置当前文档的域名。
两个域名sub.example.com和example.com 通过都将页面的document.domain设置为相同父域名则可实现互访
document.domain="example.com"
限制条件：

+ 在document.domain中都设置相同的父域名
+ 任何对document.domain的赋值操作，包括 document.domain = document.domain 都会导致端口号被重写为 null

注意：修改document.domain的同时会把当前源的端口置为"null"，解除了端口的限制。该方法已不推荐，更安全的做法是通过window.postMessage来实现，更通用，更安全，更灵活。
### 通过window.name跨域
window.name属性用来保存当前窗口的名称，通过window.name跨域是利用浏览器窗口的特性：一个窗口设置好window.name值后，即使窗口跳转到了不同域，其值依旧会保留
如果一个也面通过iframe加载一个跨域的页面，则该iframe中的页面会修改window.name属性的值，再跳转到与父页面同域的页面，父页面就可以读取这个window.name的值，从而实现跨域数据传输
window.name并非设计用来数据传输，因此有数据泄漏的风险。现代部分浏览器中，当跳转到不同域的站点时，会将window.name的值置为空。
### window.postMessage方案
window.postMessage用于一个页面向另一个窗口进行消息交互，窗口可以是通过window.open打开，或者是iframe嵌入的窗口。发送方和接收方都会进行措施保证其来源可信。
window.postMessage发送方式如下：
```
targetWindow.postMessage(message, targetOrigin, [transfer]);
```
接收方设置消息监听器，就可接收到消息：
```
window.addEventListener("message",(event)=>{
    if(event.origin !=="http://example.com:8080")
    return;
    //消息处理
},false);
```
发送方target.Origin不建议设置成*，防止消息发送给恶意网站
接收方需校验消息来源后再进行消息处理，最好再进行一次数据格式校验，防止DOM型XSS漏洞
## 跨域访问服务端
之前是不同源的前端页面互访问，此外前端页面需要从跨域的服务端获取数据
### JSONP方案
不涉及跨域时，前端通常以JSON格式从服务端获取数据，但跨域时该方案不行，会受到同源策略限制。对JSON数据进行简单处理后就可跨域共享，这就是JSONP（JSON with Padding）。将数据封装成函数。
```
callback({
     "key":"value",
     "key":"value"   
});
```
因为`<script>`标签的特殊性，其src属性可以用来加载远程脚本文件，加载跨域的JavaScript代码来执行是不受限制的，所以跨域加载JSONP代码时就把JSON数据当作参数传递给当前JavaScript执行环境所在的源，即当前页面的源。
通过函数参数传递JSON数据时，需要提前定义好回调函数(Callback)，在通过`<script src="JSONP地址"></script>`载入JSONP代码，这样回调函数就得到了JSON数据
回调函数：将函数作为参数传给另一个函数进行调用

大致过程：
A站点发起JSONP请求并指定Callback函数名称和JSONP的URL地址为B站点，服务端将A站点发送的数据包装成JSONP并发送给B站点，B站点加载该代码中的数据从而实现跨域请求。

另一种方式是通过赋值语句把数据赋给一个变量，这样也能把JSON数据引入当前JavaScript执行环境
```
var data={
    "key":"value",
    "key":"value"
}
```    
注意事项：

+ 不要在JSONP中 包含敏感数据。在不加限制的情况下，任何网站都可以载入JSONP的URL，从而使敏感数据泄漏给其他站点，这种攻击叫“JSONP劫持”。在服务端要严格校验Referer，确保只有可信的源可以跨域访问数据，或者使用随机Token方案
+ 注意校验Referer为空的场景。为防止不可信来源，拒绝没有Referer字段的JSONP请求
+ JSONP实际上是在当前域的页面执行了另一个域的JavaScript代码，确保另一个域是安全可信的，才能使用JSONP跨传输
+ JSONP的回调函数通常由URL的参数指定，JSONP本质上是一段JavaScript代码，所以在响应头需要设置Content-Type为"application/javascript"，否则默认的"text/html"，恶意构造回调函数可导致XSS风险
+ 在一些不用于跨域访问，只响应JSON/XML数据的接口中，添加上了JSONP需求可能导致安全问题。有经验的攻击者会尝试在URL中添加参数"callback=func"，或者将已有的参数"format=json"或"format=xml"改成"format=jsonp"，来挖掘JSONP劫持漏洞
JSONP只能实现单向的读操作（只支持GET请求），写操作则需要借助其他方案才能实现
### 跨域资源共享（CORS）
跨域资源共享是HTML5的新特性。在CORS方案中，Web服务器可以指定哪些域能访问自己的资源。
允许浏览器跨域访问，前端可以通过XMLHttpRequest或者Fetch API方式发起CORS请求
条件：

+ 浏览器支持CORS（主流浏览器都支持）
+ 服务要实现CORS接口

通过JavaScript跨域访问数据时（XMLHttpRequest或Fetch API），浏览器会在请求中带上一个名为Origin的头，用于指示自己当前的源。服务端对此请求的响应中需要带上一个名为Access-Control-Allow-Origin的头，用于指示哪些资源可以访问自己。
有两种场景：
简单请求（GET、POST、文件上传请求）：
客户端发送GET请求数据带上Origin字段，服务器返回带有Access-Control-Allow-Origin字段和数据的响应
复杂请求（除简单请求外）：
浏览器在发送真正的请求之前，会使用OPTIONS方法先发送一个预检请求，其中包含Access-Control-Request-Method字段包含所需要的方法以及Origin字段
服务器返回Access-Control-Allow-Origin字段和Access-Control-Allow-Method
允许之后，用允许的方法请求数据，并带上Origin字段，服务器返回带有Access-Control-Allow-Origin字段和数据的响应
"简单请求"是指通过普通的HTML表单就可以发送出去的请求的，其他的都属于"复杂请求"
"复杂请求"属于有副作用的请求，因此需要确认后才能进行传输。"简单请求"也并非没有副作用，HTML表单可以提交POST请求修改服务端数据。
请求头中的Origin头完全由浏览器控制，网页不能通过JavaScript脚本来进行修改其值，所以恶意站点不能通过修改Origin来进行伪装
服务端的另一个可选的CORS响应头时Access-Control-Allow-Credentials，其值为"true"时，表明服务器允许客户端将凭证信息（Cookie、Authorization头或者客户端TLS证书）发送给服务器；如果不为"true"，客户端设置了XMLHttpRequest对象的withCredentials为"true"时，跨域请求还是会失败。
如果服务端的响应中包含如下头，则表明定义一个信任关系，信任https://example.com源能够合法访问当前服务器。
```
Access-Control-Allow-Origin:https://example.com
Access-Control-Allow-Credentials:true
```
在CORS中，与安全相关的主要时服务端响应的Access-Control-Allow-Origin头，相当于定义了白名单，值可以是"*" 、一个明确的源或者null。不安全的做法将会带来严重的后果。

+ 当服务端响应了Access-Control-Allow-Origin:https://example.com和 Access-Control-Allow-Credentials:true时，实际上是将安全风险扩大了。example.com上的页面可以带着凭证跨域访问当前站点，如果example.com存在XSS漏洞，也会威胁当前站点。
+ Access-Control-Allow-Origin:\* 表示允许任何源访问，根据CORS定义，该字段为"*"是不允许客户端带上凭证信息。也就是说如果客户端指定了withCredentials=true来发送Cookie信息，这个请求就会失败。CORS如此定义避免了很多错误配置的安全风险。该配置只能实现跨域获取到匿名能访问的数据
+ Access-Control-Allow-Origin:null 将产生巨大风险，因为本地文件系统加载的网页(file://协议)、Data URL加载的网页(data:text/html)、沙箱化的iframe页面，浏览器会为它们指定一个新的源"null"，它跨域请求发出的Origin值为"null"正好与服务器的匹配，因此任意本地文件，恶意网站通过Data URL或者沙箱化的iframe载入的页面都能够访问当前源
+ 应用多个子域名时Access-Control-Allow-Origin并不支持通配符，通过简单探测判断目标站点是否存在漏洞：
```
curl -I https://example.com -H "Origin:https://evilsite.com"
```
如果响应中Access-Control-Allow-Origin为https://evilsite.com,并且Access-Control-Allow-Credentials为true，则存在漏洞
+ 如需支持多个源进行跨域访问，需对请求中的Origin进行白名单校验。
CORS本质上是一个跨域授权策略，服务端通过CORS策略定义了哪些资源可以访问、能否带有凭证，哪些是被HTTP请求和头是被允许的，然后由浏览器来执行这些策略，并阻止违反策略的行为 
### 私有网络访问
Google在Chrome中加入了私有网络访问限制。如果一个公网的非加密网站向私有IP网段的网站发起请求，浏览器会直接拒绝该请求。为了防止外部网站对内部网络应用发起CSRF攻击。HTTPS协议由于混合内容问题而无法实施CSRF攻击，浏览器会阻止HTTPS网站加载HTTP协议的资源
后续更新中，浏览器会根据IP地址将网站的网络区域分为三类：公共网络、私有网络、本地设备
向私密性更高的网络区域发起请求时，浏览器会发起预检请求。
RFC1918定义通过CORS访问私有网络的方式，浏览器访问私有网络前会发起预检请求并带上如下请求头：
```
Access-Control-Request-Private-Network:true
```
只有响应了Access-Control-Allow-Private-Network:true，浏览器才会继续访问目标应用。
攻击者在浏览器中实施DNS重绑定攻击时，会更改DNS记录让网页访问内部网络地址，有了该限制就能阻止这种攻击。
### WebSocket跨域访问
浏览器的同源策略并不会约束WebSocket的跨域访问，需要开发者在服务端实施安全策略。如果有WebSocket应用会返回敏感数据，或者不希望被其他网站的脚本访问，有以下措施来保护：

+ 在应用服务端校验WebSocket握手请求中的Origin头，判断是否在可信白名单上
+ 在每个会话中生成一个随机Token，客户端在WebSocket握手协议请求中带上这个Token，服务端校验。 
### 其他跨域访问
除了DOM、XMLHttpRequest、Fetch API受同源策略限制，其他浏览器插件，如Flash和Silverlight也有自己的同源策略，分别是通过服务端的策略文件crossdomain.xml和clientaccesspolicy.xml实现的