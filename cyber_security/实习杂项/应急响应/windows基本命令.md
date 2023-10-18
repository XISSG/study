---
title: windows基本命令
updated: 2023-07-21 07:42:06Z
created: 2023-07-21 05:06:34Z
latitude: 30.57281600
longitude: 104.06680100
altitude: 0.0000
---

# windows基本命令
## 文件和目录操作：

dir：列出文件和目录。-> ls (Linux)
cd：切换当前目录。 -> cd (Linux)
mkdir：创建目录。-> mkdir (Linux)
del：删除文件。-> rm (Linux)
copy：复制文件。-> cp (Linux)
move：移动文件或重命名文件。-> mv (Linux)
## 系统管理和配置：

ipconfig：显示网络适配器的 IP 配置信息。-> ifconfig (Linux)
netstat：显示网络连接和统计信息。-> netstat (Linux)
tasklist：列出当前运行的进程。 -> ps (Linux)
taskkill：结束指定的进程。 -> kill (Linux)
systeminfo：显示计算机的系统信息。 -> uname (Linux)
shutdown：关闭或重新启动计算机。-> shutdown (Linux)
## 网络管理：

ping：测试与另一个计算机的连接。 -> ping (Linux)
tracert：跟踪数据包的路径。 -> traceroute (Linux)
nslookup：查询域名的 IP 地址。 -> nslookup (Linux)
netsh：配置网络接口和防火墙设置。 -> ifconfig、iptables (Linux)
## 安全和权限管理：

sfc：扫描和修复系统文件。 -> fsck (Linux)
chkdsk：检查和修复磁盘错误。-> fsck (Linux)
gpupdate：更新组策略设置。-> sudo apt update (Linux)
cacls：修改文件或目录的安全访问权限。-> chmod、chown (Linux)
## 注册表编辑：

regedit：打开注册表编辑器。 -> vi、nano (Linux)
reg：在命令行中对注册表进行操作。 -> regedit (Linux)

## 文件、文件夹、字符串查找
### winodws
#### 查找文件和文件夹
dir：列出目录中的文件和文件夹。
dir /s：递归地列出目录及其子目录中的文件和文件夹。
where：查找给定命令的可执行文件路径。
tree：以树形结构显示目录的内容
#### 查找字符串
findstr：在文件中搜索指定的字符串模式。
type：显示文件的内容。
more 或 less：逐页显示文件的内容。
find：在文本文件中查找指定的字符串
## Linux
### 查找文件和文件夹：

ls：列出目录中的文件和文件夹。
ls -R：递归地列出目录及其子目录中的文件和文件夹。
find：按照指定的条件在文件系统中查找文件和文件夹。
tree：以树形结构显示目录的内容。
### 查找字符串：

grep：在文件中搜索指定的字符串模式。
cat：显示文件的内容。
less 或 more：逐页显示文件的内容。
find：在文本文件中查找指定的字符串。

awk：
+ 使用 awk '/pattern/ { action }' file 来在文件中查找包含指定模式的行，并对匹配行执行指定操作。
+ 使用 awk '{ print $1 }' file 来打印文件中每一行的第一个字段。
+ 使用 awk -F',' '{ print $2 }' file 来打印以逗号分隔的文件中的第二个字段。


sed：

+ 使用 sed 's/pattern/replacement/' file 来替换文件中的指定模式为指定的替换内容。
+ 使用 sed '/pattern/d' file 来删除文件中包含指定模式的行。
+ 使用 sed -n '10,20p' file 来打印文件中的第 10 行到第 20 行。
