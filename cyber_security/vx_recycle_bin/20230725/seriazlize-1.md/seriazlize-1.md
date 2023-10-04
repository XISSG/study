---
title: seriazlize
updated: 2023-07-19 15:57:03Z
created: 2023-07-19 15:52:19Z
latitude: 30.57281600
longitude: 104.06680100
altitude: 0.0000
---

# seriazlize
PHP
用于输出的函数：
print：只能输出固定字符的字符串
echo: 能输出变量
printf:格式化输出字符串
print_r:输出输出数组，对象和标量（整型，浮点型，布尔型，字符串型）
var_dump:输出任意数据

类的结构、内容、实例化、赋值、类修饰符
echo(): 只能输出字符串
print_r():只能输出数组，对象和标量（整型，浮点型，布尔型，字符串型）
var_dump:可以输出所有变量类型 
权限修饰符：public ，protected，private
序列化，将对象信息转换成字符串的过程
格式：
&->R:[count];
null->N;
int ->i:value;
float->d:value;
boolean:true->b:1;
false->b:0;
String->s:length:”value”;
arry[]->a:para_num:{i:0;s:len[0];”str[0]_value”;i:1;i:value;;i:2;s:len[2]:”str[2]_value”...}
object->O:cls_len:”cls_name”:para_num:{s:var_len:”var_name”;s:value_len:”var_value”;}
O:5:”class”:2{s:3:”var”;s:5:”param”;s:3:”int”;i:4;}
private属性序列化时需要在变量名前加%00类名%00	(空字符)
若\$var为私有属性则：
O:5:”class”:2{s:10:”classvar”;s:5:”param”;s:3:”int”;i:4;}
protected属性序列化时需要在变量名前加%00*%00
若$var为protected属性则：
O:5:”class”:2{s:6:”*var”;s:5:”param”;s:3:”int”;i:4;}
对象里调用了对象序列化套娃即可

反序列化：将字符串还原成一个对象
反序列化的特性
反序列化之后的内容为一个对象
反序列化生成对象里的值，由反序列化里的值提供;与原有类预定义的值无关
反序列化不触发类的成员方法；需要调用方法后才能触发（魔术方法除外）
反序列化漏洞的形成原因：
反序列化的值可控，通过修改这个值，得到所需要的代码，生成对象的属性值。
通过调用方法（魔术方法）使代码触发
魔术方法：
预定好的，在特定情况下自动触发的行为方法
__construct():实例化对象时触发
new时会触发
__destruct():析构函数，对象引用被删除或者对象被显示销毁时触发
触发时机：在unserialize()执行之后
反序列化会生成一个对象，new也会生成一个对象，程序执行结束后，这些对象都会被销毁，因此会触发__destruct()方法
<?php
   class user{
       public function __destruct(){
           echo "触发";
       }
   }
   $a = new user();
   $b = serialize($a);
   unserialize($b);
?>
__sleep() 和__serialize()
触发时机：serialize()函数执行之前
序列化serialize()函数会检查类中是否存在魔术方法__sleep(),__serialize()
存在就会先被调用再执行序列化操作
不同：__sleep()不能返回父类私有成员名，__serialize()可以并且优先级更高
作用：
用于清理对象，并返回一个包含对象中所有应被序列化的变量名称的数组。
如果该方法没有返回任何内容则会序列化NULL，并产生一个E_NOTICE级别的错误
__wakeup()和__unserialize()
触发时机: unserialize()函数执行之前
预先准备对象所需要的资源，返回void
注：__unserialize()优先级更高
作用：
用于反序列操作中重建数据库连接和执行其它初始化操作
__toString()
触发时机：把对象当作字符串调用($obj为一个对象)
echo $obj;
__invoke()
触发时机：把对象当作函数进行调用时
$obj()->name()
__call()
触发时机：调用一个不存在的方法
两个参数：__call($args1,$args2)
返回不存在的方法的名称和参数
$obj->unkown(a)
触发__call()时会返回unkown和a这两个参数
__callStatic()
触发时机：静态调用或调用静态常量时的方法使用的不存在
参数：同上
返回值：同上
$obj::unkown(a)
__get()
触发时机：调用的成员属性不存在
参数：__get($args1)
返回值：返回不存在的成员属性名	($var不存在)
$obj->$var;
__set()
触发时机：给不存在的成员属性赋值
参数：__set(args1，args2)
返回值：返回不存在的属性成员名称和赋值
$obj->$var=name;
__isset()
触发时机：对不可访问的属性使用isset()或empty()时会被调用
isset():判断变量是否声明并赋值，存在非null返回true否则返回false
empty():判断是否变量为空，不存在，值为0，空字符串，null，false或空数组都返回false，否则返回true
empty()会将0判为空，isset()不会
参数：__isset(args1)
返回值：返回不存在的成员属性名	($var为private或不存在时都会触发)
isset($obj->var);
__unset()
触发时机：对不可访问的属性使用unset()时
unset():销毁变量，只能销毁非私有属性的变量,销毁private成员变量时会自动触发__unset()
参数：__usnet(args1)
返回值：返回不存在的成员属性名称
unset($obj->var);
__clone()
触发时机：当使用clone关键字copy完成一个对象后，新对象会自动调用__clone()
$obj2 = clone $obj1;或	$obj2 = clone($obj1);

__wakeup()绕过
	当unsirialize()之前会触发__wakeup()序列化字符串中的属性个数大于实际的个数就会__wakeup()就不会执行（版本限制php5<5.6.25,php7<7.0.10）

字符逃逸
凑数值长度

php反序列化漏洞
当session_start()被调用或者php.ini中的session.auto_start为1时，PHP内部调用会话管理器，访问用户session被序列化以后，存储到指定目录（默认/tmp）
产生原因：写入格式和读取格式不一致

php存储session的三种方式：
php_serialize				经过serialize()函数序列化的数组
例如：s:6:”123456”
php						键名+竖线+经过serialize()函数处理的值
例如：benben|s:6:”123456”;
php_binary				键名的长度对应的ascii字符+键名+serialize()函数序列化的值
例如：06benbens:6:”123456”;
对于php_serilize读取a:1:{s:3:”benben”;s:39:”|O:1:”D”:1:{s:1:”a”;s:10:”phpinfo();”;}”;}时：
结果为：
Arry{Benben=>” |O:1:”D”:1:{s:1:”a”;s:10:”phpinfo();”;}”;}
用php读取值
结果为：
a:1:{s:3:”benben”;s:39:”=D{
	a=” phpinfo();”;
}
不同的读取方式就有可能导致php反序列化漏洞

Phar反序列化漏洞
Php通过用户定义和内置的“流包装器”实现复杂的文件处理功能。内置包装器可用于文件系统函数。
在打包文件的内容字段，会将其序列化，使用文件操作函数时会对内容自动进行反序列化。若后端代码包含特定魔术方法，就会通过上传phar文件并，在调用该文件时文件操作函数就会触发魔术方法，从而实现特定目的。
利用条件：php.ini中需要设置phar.readonly=Off（白盒审计）
Phar的结构：
	stub phar文件标识，格式为xxx<?php xxx;)__HALT_COMPILER();?>头部信息
	manifest 压缩文件的属性等信息，以序列化存储
	contents 压缩文件的内容
	signature 签名，放在文件尾
phar协议在解析文件时，会自动触发对manifest字段的序列化字符串进行反序列化
常见的文件系统函数：
	fopen() - 打开文件或URL		fclose() - 关闭文件		fread() - 读取文件内容
fwrite() - 写入文件内容		file_get_contents() - 读取整个文件内容到一个字符串中
file_put_contents() - 将一个字符串写入文件			fgets() - 从文件中读取一行
feof() - 检查文件指针是否到达文件末尾				fseek() - 在文件中定位指针位置
file_exists() - 检查文件或目录是否存在				is_file() - 判断指定路径是否为文件
is_dir() - 判断指定路径是否为目录					mkdir() - 创建目录
rmdir() - 删除目录				unlink() - 删除文件	rename() - 重命名文件或目录
copy() - 复制文件				scandir() - 扫描目录并返回文件列表
glob() - 根据模式匹配文件路径	realpath() - 返回规范化的绝对路径名 

常见的PHP流包装器：
	file:// - 用于访问本地文件系统中的文件
http:// - 用于通过HTTP协议访问远程文件
https:// - 用于通过HTTPS协议访问远程文件
ftp:// - 用于通过FTP协议访问远程文件
data:// - 用于访问数据流，如内存中的字符串或变量
php://input - 用于访问HTTP请求的原始数据
php://output - 用于访问HTTP响应的输出流
php://stdin - 用于从标准输入读取数据
php://stdout - 用于向标准输出写入数据
php://stderr - 用于向标准错误输出写入数据
后端phar的使用流程：
``<?php

\$phar = new Phar("a.phar");   //文件名
\$phar->startBuffering();//启动Phar缓冲区，用于在创建或修改Phar存档时将文件内容写入缓冲区而不立即写入磁盘
/*设置结尾以<?php __HALT__COMPILER();?>结尾。"<?php __HALT_COMPILER();?>"是一个特殊的PHP标记，它告诉PHP解释器在这里停止解析代码*/
\$phar->setStub("<?php __HALT_COMPILER();?>");//添加要压缩的文件
\$phar->addFromString("test.txt","contents");
\$phar->stopBuffering();
?>``
主文件：
``<?php
echo file_get_contents('phar://a.phar/test.txt');//用file_get_contents文件操作函数，phar流包装器读取文件a.phar/test.txt的内容
?>
若代码为：
<?php
    class Test{
        public $num=2;
        public function __destruct()
        {
          if($this->num===1){
              echo 'flag{sss}';
          }   // TODO: Implement __destruct() method.
        }
    }
    echo file_get_contents($_GET['path']);
?>``

则可以生成特殊的phar文件：
``	<?php
class Test{
    public \$num=2;
}

\$test= new Test();
\$phar = new Phar("a.phar");   //文件名
\$phar->startBuffering();//启动Phar缓冲区，用于在创建或修改Phar存档时将文件内容写入缓冲区而不立即写入磁盘
//设置结尾以<?php __HALT__COMPILER();?>结尾。"<?php __HALT_COMPILER();?>"是一个特殊的PHP标记，它告诉PHP解释器在这里停止解析代码*/
\$phar->setStub("<?php __HALT_COMPILER();?>");

\$phar->setMetadata($test);  //设置自定义内容，序列化存储

/*添加要压缩的文件*/
\$phar->addFromString("test.txt","contents");
\$phar->stopBuffering();
?>``

然后在URL中引用上传文件路径即可：
``?path=’phar://a.phar/test.txt’``

可操作性：若后端代码有__destruct(),__wakeup()魔术方法就会因为触发
受文件操作影响相关函数：
	``fileatime/filectime/filemtime
	stat/filenode/fileowner/filegroup/fileperms
	file/file_get_contents/readfile/fopen
	file_exists/is_dir/is_excutable/is_file/is_link/is_readable/is_writeable
	parse_ini_file
	unlink/copy
其它触发函数：
image：
	exif_thumbnail
	exif_imagetype
	imageloadfont
	imagecreatefrom***
	getimagesize
	geimagesizefromstring
hash
	hash_hmac_file
	hash_file
	hash_update_file
	md5_file
	sha1_file
file_url
	get_meta_tags
	get_headers
	
绕过不允许以phar://开头
compress.bzip://phar://a.phar/test.txt
compress.bzip2://phar://a.phar/test.txt
compress.zlib://phar://a.phar/test.txt
php://filter/resource=phar://a.phar/test.txt
php://filter/read=convert.base64-encode/resource=phar://a.phar/test.txt ``
