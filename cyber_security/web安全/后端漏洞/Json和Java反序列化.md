---
title: Json和Java反序列化
updated: 2023-07-25 02:56:22Z
created: 2023-07-24 01:24:24Z
latitude: 30.57281600
longitude: 104.06680100
altitude: 0.0000
---

# Java基础
Java.io.ObjectOuptustream代表输出流，其中的writeObject()方法会将指定对象进行序列化，将得到的字节序列写入到目标输出流中
Java.io.ObjectInptustream代表输入流，其中的readObject()方法会将指定字节流进行反序列化为一个对象，并返回。
只有`实现了Serialize和Externalizable接口的类`才能被序列化和反序列化，该对象内的属性必须为public，用关键字`transient`，`static`指定了的属性不会被序列化
```
import java.io.*
//要序列化的类
class person implements Serializable{
	public String name;
	public transient int age; //不进行序列化
	
	public person(String name,int age){
		this.name=name;
		this.age=age;
	}
}

//进行序列化和反序列化
public class Serialize{
	public static void main(String args[]) throws IOException, ClassNotFoundException{
	
	//序列化对象
	FileOutpuStream fos = new FileOutputStream("person.ser");
	ObjectOuputStream oos = new ObjectOutputStream(fos);
	oos.writeObject(new person("John",30));
	oos.close();
	} 
}

	//反序列化字节流
	FileInputStream fis = new FileInputStream("person.ser");
	ObjectInputStream ois = new ObjectIputStream(fis);
	ois.readObject();
	ois.close();
```
## 常见序列化和反序列化协议
+ XML& SOAP
+ JSON 
+ Protobuf
## 利用形式
+ 入口类的readObject直接调用危险方法
+ 入口类参数中包含可控类，该类有危险方法，readObject时调用
+ 入口类参数中包含可控类，该类又调用其它危险方法的类，readOject时调用
+ 构造函数/静态代码块等类加载时隐式执行
## 利用过程
1、继承 Serializable类
2、入口类 source(重写readObject参数类型宽泛，最好jdk自带)
3、调用链 gadget chain 相同名称，相同类型
4、执行 sink(rce,ssrf,写文件等) 
## 入口类的选择
Map接口的HashMap类
HashMap类重写了readObject方法
## 工具
ysoserial(github)
## payload(URLDNS链)
```
public static void main(String[] args){
    Person person = new Person("aa",22);//Person 类继承Serializabale接口
    HashMap<URL,Integer> hashmap = new HashMap<URL,Integer>();
    
    //这里不要发起请求,反序列化的时候发起DNS请求
    URL url = new URL("http://");
    //通过反射改变已有对象的属性
    Class c = url.getClass();
    Field field=c.getDeclareField("hashCode");
    field.setAccessible(true);
    field.set(url,1234);
    hashmap.put(url,1); //添加键值对
    //这里把hashcode改为-1
    field.set(url,-1);
    serialize(hashmap);
 }
```
## Java反射
### 在执行时操作和修改类的属性和方法
作用：
+ 让Java具有动态性
+ 修改已有对象的属性
+ 动态生成对象
+ 动态调用方法
+ 操作内部类和私有方法

### 在反序列化中的利用：
+ 定制需要的对象
+ 通过invoke调用除了同名函数以外的函数
+ 通过Class类创建对象，引入不能序列化的类

### 反射能修改的部分：
设置访问权限
```
setAccessible();
```
获取和创建Class对象
```
//获取对象
Class.forName();
Person.class;
peson.getClass()
//创建对象 
Class.newInstance();
Counstructor.newInstance();
```
获取构造方法
```
Class a = person.getClass();
Constructor constructor = a.getConstructor();
constructor.newInstance();
```
获取和调用方法
```
Method method = Class.getMethod();
method.invoke();
```
获取和修改属性
```
Field filed = Class.getField("hashCode");
filed.set(url,1);
```
### 反射代码示
```
public static void main(String[] args){
    Person person = new Person();
    Class c= person.getClass();//几乎所有的对象在创建时，都会继承java.lang.Object类中的getClass()方法，返回一个Class对象
    //反射就是操作Class
    //从原型的Class中实例化对象
    Constructor personconstructor = c.getConstructor(String.class,int.class);//获取构造器
    Person p = (Person)personconstructor("abc",22);//传参，返回Object类进行了强转
    
    //获取类中的属性和更改属性值
    Fields[] personfileds = c.getFields();//获取非私有属性的所有属性
    Fields[] personfileds = c.getDeclareFields();//获取所有属性
    for(Filed f:personfields){
        f.setAccessible(true);    //获取更改所有属性的权限
        f.set(p,"abd");        //set()方法设置属性的值
    } 
    
    //操作类里面的方法
    Method[] personmethods = c.getMethods();//获取类中的方法
    Method action = c.getMethods("action"，String.class);//根据名字来获取,指定传参类型
    Method action = c.getDeclaredMethod("action"，String.class);//获取所有
    c.setAccessible(true);    
    action.invoke();//调用方法
    
}
```
## Java动态代理 
### 动态代理
在程序执行时，使用jdk的反射机制，创建代理类对象，并动态指定要代理的目标类
不用创建类，就能创建代理类对象
### 实现
主要部分：目标接口，目标类，InvocationHandler的实现类，主函数创建，调用
java.lang.reflect，三个类：InvocationHandler ,Method,Proxy
InvocationHandler接口：invoke();代理对象的要执行的功能代码
```
public Object invoke(Object proxy,Method method,Object[] args)
```
参数：Object proxy：jdk创建的代理对象，无需赋值
           Method method：目标类中的方法，无需赋值
           Object args：目标类中的参数，无需赋值
创建类实现InvocationHandler接口，重写invoke方法。
Method类：执行目标类中的方法，通过Method.invoke()实现
```
Method.invoke(Object Class,String args)
```
Proxy类：创建代理对象。代替new创建对象，newProxyInstance()
```
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h);
```
参数：ClassLoader loader类加载器，利用反射获取对象的ClassLoader，a.getClass().getClassLoader(),目标对象的类加载器
           InvocationHandler h 要完成的功能
#### 实现步骤
1、创建接口，定义目标类要完成的功能
2、创建目标类实现接口
3、创建InvocationHandler接口的实现类，在invoke方法中完成代理类的功能
4、使用Proxy类的静态方法，创建代理对象。并把返回值转为接口类型
### 代码示例
```
import java.lang.reflect.*;
//创建接口
public interface makesence{
    int counter;
    public void counter();
}
//创建目标类
public class target implements makesence{
    int counter = 0;
    @override
    public void counter(){
           counter = counter++;
    }
}
//实现InvocationHandler类
public class myInvocationHandler implements InvocationHandler{
    private Object target = null;
    public myInvocationHandler(Object target){
        this.target = target;
    } 
    @override
     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = method.invoke(target, args);//反射获取对象的方法，然后调用
            //添加其他执行代码
        return result;
}

//主函数创建实现
public class Main{
    public static void main(String[] args){
        maksence myTarget = new target();//创建目标对象
        InvocationHandler invocationhandler = new myInvocationHandler(myTarget);//创建InvocationHandler的实现类对象
        makesence proxy = (makesnece)Proxy.newInstance(myTarget.getClass().getClassLoader(),myTarget.getClass().getInterfaces(),invocationhandler);//创建代理对象
       proxy.counter(); //调用对象中的方法
    }
}
```
## Java类加载机制和流程
### 概念
类加载机制是指将.class文件中的二进制数据读入到内存中，并对数据进行校验，解析和初始化
类的生命周期，经历七个阶段：加载，验证，准备，解析，初始化，使用，卸载
**类加载时会执行的代码：**
初始化：静态代码块
实例化：构造代码块，无参构造函数
**动态加载方法： **
ClassLoader.defineClass 字节码加载任意类    私有
Unsafe.defineClass 字节码加载任意类    public类不能直接生成，Spring中可以直接生成 
**获取ClassLoader **
```
ClassLoader loader =null;
loader = Thread.currentThread().getContextClassLoader();//通过当前线程获取
loader = ClassLoader.getSystemClassLoader();
loader = this.getClass().getClassLoader();//通过已加载的class获取
```
实现
### 三层类加载器
启动类加载器（Bootstrap ClassLoader）、扩展类加载器(Extension ClassLoader)、应用程序类加载器(Application ClassLoader)
双亲委派模型，任何类加载器接收到一个类加载请求时，都会让其父类进行加载，父类无法加载时，子类才会加载
**继承关系**
ClassLoader->SecureClassLoader->URLClassLoader->AppClassLoader
**调用关系**
loadClass->findClass(重写的方法)->defineClass(从字节码加载类)
### 实现任意类加载
URLClassLoader任意类加载：file/http/jar
```
URLClassLoader urlclasloader = new URLClassLoader(new URL[]{new URL("https://www./")});//可以用file协议，http，jar协议加载
Class<?> c = urlclassloader.loadclass("Test");    //加载类，不会进行初始化
c.newInstance();  //实例化
```
# Commons Collections
## 简介（利用版本3.2.1，JDK 8U65）
Commons Collections 维护一些集合类（List 列表,Set 集合, Map 映射, Queue 队列等）满足调用类的条件
Java反序列化利用流程
>接受任意类执行readObject方法-->该类调用其它方法-->其它类又调用其它方法-->...-->调用危险方法
条件：
**入口类**
* 入口类必须可序列化
* 重写readObject方法
* 接收任意对象作为参数
**调用类**
* 可序列化
* 集合类型/接受Object/接受Map
**不同类的同名函数任意方法调用**(transform)
* 反射
* 动态加载字节码
 
### shiro反序列化

