# docker
## 镜像
```
docker images
	-a	显示所有
	-q	只显示id
docker search
	
docker pull
docker rmi	删除镜像
	-f 指定id
docker rmi -f $(docker images -aq)删除所有镜像

docker run [OPTIONS] IMAGE [COMMAND] [ARG...]	创建并启动一个新的容器
	--name =<image_name>	
	-d 后台运行
	-it 交互式运行，进入容器查看内容
	-p 指定容器端口 端口映射
		-p 8080:80 将容器的80端口映射到主机的8080端口
		-p 192.168.0.1:8080:80 将容器192.168.0.1:80端口映射到主机的8080端口
	-P 随机指定端口（不推荐使用）
docker ps 显示正在运行的镜像
	-a 显示出所有运行过的镜像
	-n 显示最近显示的n个镜像
```

## 容器操作
```
退出容器
	exit
 Ctrl+P+Q后台运行
删除容器
	docker rm <id>删除指定的容器，不能删除正在运行的容器
		-f	强制删除容器
		docker rm -f $(docker ps -aq) 删除所有容器
		docker ps -aq|xargs docker rm
	
启动和停止容器的操作
	docker start <id> 启动容器（该容器已存在）
	docker retart <id>重启容器
	docker stop <id> 停止当前正在运行的容器
	docker kill <id>强制停止当前容器
```
## 查看信息
```
查看日志
	docker logs 
查看进程信息
	docker top <id>
	
查看镜像元数据
	docker inspect <id>
进入当前正在运行的容器
	docker exec -it <id> /bin/bash
进入正在运行的容器
    docker attach
将容器的内容复制到主机
	docker cp <容器id>:<容器路径> <主机路径>
将主机内容复制到到容器
	docker cp 主机路径 <容器id>:<容器路径>

 查看CPU状态
	docker stats
提交容器成为镜像
	Docker commit -m=”描述信息” -a=”作者” 容器id 镜像名:[TAG]
```
## 容器数据卷
```
将容器产生的数据同步到本地，将目录挂载到系统中	
	-v （volume）映射主机目录:容器目录
docker volume ls 查看挂载的数据卷	
	具名挂载
	-v 挂载名:容器目录
	匿名挂载
	-v 容器目录
	-e	环境配置

dockerfile构建docker镜像
	创建dockerfile，内置docker脚本命令
	使用docker build 创建镜像
	docker push 推送到远程地址
--volumes-from	实现容器间数据共享
```
## 流程
+ 编写dockerfile
+ 生成docker images
+ 运行docker run 或保存到远程仓库 docker push 
……

## dockerfile指令
```
	FROM 基础镜像
	MAINTAINER	作者
	RUN 运行的命令
	ADD 添加文件，会自动解压压缩包
	WORKDIR 工作目录
	VOLUME 挂载卷路径
	EXPOSE 对外开放端口
	CMD 指定容器启动是运行的命令，覆盖命令
	ENTRYPOINT 追加命令
	ONBUILD 构建一个被继承的dockerfile就会触发该命令
	COPY 将文件复制到镜像中
	ENV 构建时设置环境变量
```
例如：
```
FROM centos
MAITAINER xissg
ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yumm -y install net-tools

EXPOSE 80

CMD ["ls","-a"]
CMD echo $MYPATH
CMD /bin/bash 
```
## 创建镜像
```docker build -f <file_path> -t <tag_name>```
查看镜像的创建过程(镜像配置信息)
	```docke history <id>```
## 发布镜像
```
docker login 
docker push 
docker tag 添加标签（push前需要添加标签）
docker save
docker load
 
docker network 网络相关操作
```