# python反序列化
## 基础知识
### 反序列化方式
json、dbm、pickle、shelve
反序列化漏洞常用pickle模块
### 反序列化的对象
以下类型都是可以使用pickle进行反序列化的
+ None，True，False
+ 整数，浮点数，复数
+ 字符串，字节，字节组
+ 元组，列表，集合和仅包含可pickle序列化对象的字典
+ 在模块顶层定义的函数（使用def定义，lamda函数不可以）
+ 在模块顶层定义的类
+` __dict__`属性或`__getstate__()`函数的返回值
pickle.dumps()函数将python对象进行序列化得到对应的bytes-like对象，对应的pickle.loads()进行反序列化
pickle.dump()函数将python对象序列化写入一个文件中,对应的pickle.load()进行反序列化
## 漏洞原理
python程序对来源不可信的数据调用了pickle.load()或pickle.loads()进行反序列化是就会造成反序列化漏洞
python也内置magic函数
主要利用magic函数：`__reduce__()`
返回值为一个元组，（callbale_func,(parmeter1,parmeter2...)）
```
import pickle
import os 

class A(object):
    def __reduce__(self):
        return (os.system(),('cmd',))

a=A()   //实例化对象
test=pickle.dumps(a) //序列化对象
pickle.loads(test) //反序列化，触发__reduce__()方法
```