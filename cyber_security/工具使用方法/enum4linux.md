# enum4linux
smb服务信息收集工具
## 大致探测原理
匿名访问：enum4linux首先尝试使用匿名访问（guest账户）连接到目标主机的共享资源，以获取公开可访问的信息。

用户枚举：如果匿名访问失败，enum4linux会尝试枚举目标主机上的用户账户信息，包括用户名、SID（Security Identifier）和用户权限等。

组枚举：enum4linux还会尝试枚举目标主机上的组信息，包括组名、SID和组成员等。

共享枚举：enum4linux会列举目标主机上的共享资源，包括共享名称、路径和访问权限等。

密码策略：enum4linux可以获取目标主机上的密码策略信息，如密码最小长度、密码复杂性要求和帐户锁定策略等。

注册表枚举：enum4linux还可以枚举目标主机的注册表信息，包括安全标识符（Security Identifiers，SIDs）和注册表键值等。

## smb服务概述
用于文件用于文件共享服务，通常在内网中进行使用，由于其明文传输的特点可以使用工具 进行枚举爆破

### 默认端口
TCP端口445：这是SMB协议的主要端口，用于文件共享、打印共享和其他SMB相关服务的通信。

UDP端口137和138：这些端口用于NetBIOS名称服务（NetBIOS Name Service）和NetBIOS数据报服务（NetBIOS Datagram Service），这些服务与SMB密切相关。

TCP端口139：这个端口也用于NetBIOS会话服务（NetBIOS Session Service），它是SMB的一部分
## 工具使用

## NTML服务