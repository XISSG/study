# PUT漏洞
## IIS写入权限漏洞
条件：IIS服务器开启“写入”“脚本资源访问”和“WebDAV扩展”
通过HTTP协议的PUT方法写入ASP一句话木马，然后通过MOVE,COPY方法将该文件后缀改为asp
若未开启“脚本资源访问”则可通过IIS6.0解析漏洞，将文件后缀改为".asp;.jpg"
## Tomcat PUT漏洞
条件：当web.xml中的“readonly设置为false”时可以通过PUT/DELETE进行文件操控，上传jsp木马文件