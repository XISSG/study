# SSI注入漏洞
通过SSI指令可实现系统命令执行
Nginx,Apache,IIS都支持SSI功能
开启了SSI功能的文件后缀通常为stm、shtm、shtml
上传一个shell.shtml文件，内容为如下SSI指令
```
<!--#exec cmd="whoami"-->
```
即可实现远程命令执行
