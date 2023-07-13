## Docker概述

## Docker为什么会出现？

开发和运行 两个环境

版本更新	开发运维

环境配置	每个机器都要部署

发布一个项目（）

跨平台

开发打包部署上线，一套流程做完！



Docker给出了解决方案！



java  --  apk  --  发布（商店） --  下载即可用

java  --  环境  --  打包项目带上环境（镜像）  --  docker仓库：商店  --  下载发布镜像  --  直接运行



Docker

核心思想：打包装箱

通过隔离机制，把服务器用到极致



## Docker的历史

2010	dotCloud

pass的云计算服务	LXC有关的容器技术

容器化技术	命名为 Docker

2013	开源

十分轻巧

出来之前用的都是 **虚拟机**

VMware：虚拟出了一台或者多台电脑 笨重			几个G

虚拟机也属于虚拟化技术，Docker也是虚拟化技术	小巧	几个M	镜像



> 聊聊Docker

基于Go语言开发的

文档地址：[Docker Docs: How to build, share, and run applications | Docker Documentation](https://docs.docker.com/)

仓库地址：https://hub.docker.com/



> ​	虚拟机

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710163146756.png" alt="image-20230710163146756" style="zoom:50%;" />



虚拟机技术缺点：

1、资源占用十分多

2、冗余步骤多

3、启动慢



>  	Docker

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710163321098.png" alt="image-20230710163321098" style="zoom:50%;" />

Docker和虚拟机的不同：

- 传统虚拟机，虚拟出一套硬件，运行一个完整的系统，然后在这个系统上安装和运行
- 容器内的应用直接运行在宿主机内，没有自己的内核，也没有虚拟化硬件，cpu利用率
- 每个容器间是互相隔离的，每个容器内部有自己的容器系统

![image-20230710171037014](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171037014.png)



**DevOps（开发运维）**

**更快速的交付和部署**

传统：帮助文档，安装程序

Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩缩容**

部署应用和搭积木一样

项目打包为镜像

**更简单的系统运维**

开发，测试环境高度一致

**更高效的计算资源利用**

内核级别的虚拟化，可以在一个物理机上运行很多容器实例，压榨服务器性能

**安全性劣势**

类似于线程进程吧

## Docker安装

### 基本组成

![image-20230710171001192](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171001192.png)

**镜像（images）**

好比一个模板，可以创建容器服务

一个镜像可以创建多个容器，最终服务运行就是在容器中

**容器（container）**

独立运行一个或者一组应用，通过镜像创建

启动，停止，删除，基本命令

理解为一个建议的linux系统

**仓库（repository）**

存放镜像的地方，分为公有和私有

### 安装Docker

- 常用命令


```shell
docker --version
# 查看镜像
docker images
# 查看正在运行的容器
docker ps 
# 查看所有的容器
docker ps -a 
# 拉取最新的mysql软件包
docker pull mysql
# 启动docker -d是后台运行 -p是指定端口 --name是改名 -e是设置登录密码
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql -d
# 启动已有容器
docker start 995cbe722379
# 通过 exec 命令对指定的容器执行 bash
docker exec -it 995cbe722379 /bin/sh
# 退出
exit
# 进入mysql操作数据库
mysql -uroot -p123456
# 在mysql命令行中查看版本信息
status
# 删容器
docker rm [container_id/container_name]
# 删除镜像: 要把镜像关联的所有容器删完了才能删除这个镜像，否则要用强制删除
docker rmi -f [image_id/image_name]
docker rmi [image_id/image_name] 
docker image rm [image_id/image_name]
# 启动镜像
docker start mysql
# 重名了efeb4214cfc4为hello
docker tag efeb4214cfc4 hello
# 然后将原来的image名称删除
docker rmi hello:1.0
```

###  运行流程

类比着dns的查找。。

![image-20230710171424414](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171424414.png)

### 镜像命令

![image-20230710171524903](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171524903.png)

#### docker images

![image-20230710171600893](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171600893.png)

#### docker search

![image-20230710171818411](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171818411.png)

#### docker pull

![image-20230710171855218](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171855218.png)

#### docker rmi tag

![image-20230710171932084](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710171932084.png)

docker tag 镜像id 命名容器

### 容器命令

#### docker run

-interact	-port

![image-20230710172119263](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710172119263.png)

![image-20230710172154961](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710172154961.png)

#### docker ps 查看容器

present

![image-20230710172443238](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710172443238.png)

#### docker rm 删除容器

remove

![image-20230710172538477](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710172538477.png)

![image-20230710172625587](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710172625587.png)

#### docker exec进入容器

![image-20230710172745460](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710172745460.png)

![image-20230710172859285](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710172859285.png)

### 小结

![image-20230710173007362](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710173007362.png)

![image-20230710173117000](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710173117000.png)

![image-20230710173129486](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710173129486.png)

> ​	Docker部署Nginx

docker search nginx	搜索image，去docker hub上

docker pull nginx         拉取下载

docker images              查看

docker run -d  --name   nginx01   -p    3344:80  nginx				启动：后台运行	起名字	暴露端口	宿主机端口:容器内部端口

docker ps					  查看

curl localhost:3344		本机自测

docker exec -it nginx01 /bin/bash

cd /etc/nginx

docker stop nginx		停止

**端口暴露**

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710195421406.png" alt="image-20230710195421406" style="zoom:50%;" />

每次改动nginx配置文件都要重新进入容器内部？

如果可以在容器外部提供一个映射路径，达到在容器外部修改文件，容器内部就可以自动修改。？

**数据卷**



### 可视化

**portainer**

Docker的图形化界面管理工具，提供后台面板

```shell
docker run -d -p 8088:9000 \
... portainer/portainer
```

访问测试：

ip:8088

## Docker镜像原理

### **镜像原理之联合文件系统**

UnionFS

包含所有代码，运行时库，环境变量和配置文件

所有应用，直接打包成docker镜像

**分层轻量级且高性能的文件系统**

![image-20230710201015335](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710201015335.png)

![image-20230710201235135](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710201235135.png)

![image-20230710201323146](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710201323146.png)

黑屏----开机进入系统：公用的（bootfs

容器就是一个小的linux环境

![image-20230710201523992](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710201523992.png)

### **分层的理解**

![image-20230710201630020](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710201630020.png)

![image-20230710201722643](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710201722643.png)

![image-20230710201930917](C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710201930917.png)

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710202013398.png" alt="image-20230710202013398" style="zoom:50%;" />



<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710202123784.png" alt="image-20230710202123784" style="zoom:50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710202227081.png" alt="image-20230710202227081" style="zoom:50%;" />

pull来run镜像层，**我的所有操作都是基于容器层的**

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710202655668.png" alt="image-20230710202655668" style="zoom:50%;" />

### commit镜像

```shell
docker commit  提交容器成为一个新的副本

命令和git
docker commit -a="作者" -m="发布信息" 容器id 目标镜像名:TAG
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710203510204.png" alt="image-20230710203510204" style="zoom:50%;" />



## 容器数据卷

### 什么是容器数据卷

**docker**的理念

将应用与环境打包成一个镜像image

数据？如果数据都在容器中，那么容器删除后，数据就会丢失！

**新需求：数据可以持久化**

MySQL，容器删了，删库跑路？希望MySQL数据可以存储在容器之外

**容器之间的数据共享技术**

在Docker容器中产生的数据，同步到本地

目录的挂载，将容器内的目录，挂载到Linux上

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710204115364.png" alt="image-20230710204115364" style="zoom:50%;" />

**总结：容器的持久化和同步操作！容器间的数据共享！**

使用数据卷

```she
用命令来挂在
docker run -it -v /home/ceshi:/home centos /bin/bash		#主机目录:容器内目录

# 启动后通过inspect查看 
docker inspect container_name 		# 查看mount的地方
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710204813056.png" alt="image-20230710204813056" style="zoom:50%;" />

内部有，外部同步了？？容器内的操作，会同步到容器外部

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710205058951.png" alt="image-20230710205058951" style="zoom:50%;" />

再来测试：

1. 停止容器
2. 宿主机（服务器）修改文件
3. 再次启动容器，数据依旧同步

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710205357511.png" alt="image-20230710205357511" style="zoom:50%;" />

**服务器上操作就好了，无需进去容器**

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710205553306.png" alt="image-20230710205553306" style="zoom:67%;" />

### 实战：安装MySQL

MySQL**的数据持久化问题**

```shell
docker search mysql

docker pull mysql:5.7

docker images

# mysql需要配置密码
# -d 后台运行  -p 端口映射  -v 卷挂载  -e 环境配置  --name 容器名字
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123455 --name mysql="mysql01" mysql:5.7

# 启动成功后，本地使用sqlyog
```

假设将容器删除，发现挂载到本地的数据卷依旧没有丢失

### **具名挂载和匿名挂载**

```shell
# 匿名挂载  不指定主机目录  容器内目录必须写
docker run -d -P --name xxxx -v /etc/nginx nginx
# 具名
docker run -d -P --name xxxx -v juming:/etc/nginx nginx

# 查看所有卷的情况
docker volume ls
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710211317508.png" alt="image-20230710211317508" style="zoom:50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710211746142.png" alt="image-20230710211746142" style="zoom:50%;" />

挂载卷不指定目录，都是在`/var/lib/docker/volumes/xxxx/_data`目录下

xxxx就是那个名字

通过额外操作 ro rw：只读，读写————针对容器的权限

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710212237838.png" alt="image-20230710212237838" style="zoom:50%;" />

默认rw，，看到ro，说明文件只能通过宿主机操作，容器内部不能操作

### DockerFile

用DockerFile来构建docker镜像的构建文件！命令脚本！

通过脚本可以生成镜像，镜像是一层层的，脚本就是一个个命令

dockerfile1

```bash
FROM CENTOS

VOULUM ["volume01", "volume02"]

CMD echo "---end---"
CMD /bin/bash
```

-f : 文件目录

-t：生成的文件名

```shell
docker build -f /home/xxx/dockerfile1 -t kuangshen/centos:tag_ .
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710213448674.png" alt="image-20230710213448674" style="zoom: 50%;" />

```shell
# 启动自己的容器
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710213824543.png" alt="image-20230710213824543" style="zoom:67%;" />

生成镜像时，自动挂载的数据卷目录

这个卷一定有一个同步的目录

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710215759804.png" alt="image-20230710215759804" style="zoom:50%;" />

找的话一定是一段很长的乱码，符合要求的路径

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710215932255.png" alt="image-20230710215932255" style="zoom:50%;" />

### 数据卷容器

多个mysql共享数据

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710220144705.png" alt="image-20230710220144705" style="zoom:50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710220529295.png" alt="image-20230710220529295" style="zoom: 50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710220707340.png" alt="image-20230710220707340" style="zoom:50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710220850345.png" alt="image-20230710220850345" style="zoom:50%;" />

**结论**

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有使用容器为止。

但是一旦持久化到了本地，这个时候，本地的不会随容器的删除而删除。

## DockerFile

用来构建docker镜像的文件！命令参数脚本

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710221211353.png" alt="image-20230710221211353" style="zoom: 50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710221309217.png" alt="image-20230710221309217" style="zoom:50%;" />

### DockerFile的构建过程

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710221431568.png" alt="image-20230710221431568" style="zoom:50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710221404815.png" alt="image-20230710221404815" style="zoom: 50%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710221632477.png" alt="image-20230710221632477" style="zoom:50%;" />

### DockerFIle的基础命令

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710221852712.png" alt="image-20230710221852712" style="zoom:50%;" />

```shell
FROM				# 基础镜像 
MAINTAINER			# 谁写的（姓名+邮箱
RUN					# 构建时需要运行的命令
ADD					# 步骤，添加内容
WORKDIR				# 镜像的工作目录
VOLUME				# 挂载到哪个目录
EXPOSE				# 保留端口配置
CMD					# 指定容器启动时运行的命令，只有最后一个生效
ENTRYPOINT			# 指定启动时要运行的命令，可以追加命令
ONBUILD				# 构建一个被继承的 DockerFile，就会运行，触发指令
COPY				# 类似ADD，将文件拷贝到镜像中
ENV					# 构建的时候设置环境变量
```

### 实战测试

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710222936124.png" alt="image-20230710222936124" style="zoom:50%;" />

```shell
# 1、编写dockerfile配置文件
# 创建一个自己的centos
FROM centos
MAINTAINER kaungshen<huiafhiu@qq.com>

ENV MYPATH /usr/local
# 工作目录
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "---end---"
CMD /bin/bash

# 2、通过这个文件构建镜像
docker build -f mydockerfile -t mycen:0.1 .
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710223654577.png" alt="image-20230710223654577" style="zoom:50%;" />

```shell
docker history dockerfilename
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710224002258.png" alt="image-20230710224002258" style="zoom:50%;" />

> ​	CMD 和 ENTRYPOINT 区别

```shell
CMD					# 指定容器启动时运行的命令，只有最后一个生效
ENTRYPOINT			# 指定启动时要运行的命令，可以追加命令
```

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710224552365.png" alt="image-20230710224552365" style="zoom: 67%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710224822935.png" alt="image-20230710224822935" style="zoom:67%;" />

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710224925960.png" alt="image-20230710224925960" style="zoom:67%;" />

DockerFile中很多命令十分相似，需要了解区别

### 实战：Tomcat镜像

1. 准备好镜像文件tomcat压缩包，jdk压缩包

   <img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710225202718.png" alt="image-20230710225202718" style="zoom:50%;" />

2. 编写dockerfile文件，官方命名Dockerfile，build会自动寻找这个文件

   ```shell
   FROM centos
   MAINTAINER kuangshen<sdfsf@qq.com>
   
   COPY readme.txt /usr/local/readme.txt
   
   ADD jdk-xxx.gz	/usr/local/		# 自动解压
   ADD apa.tar.gz	/usr/local/
   
   RUN yum -y install vim
   
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   ENV JAVA_HOME /usr/local/jdk1.8_11
   ENV CLASSPATH $JAVAHOME/lib/xxxxxx:$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/app.22
   ENV CATALINA_BASH /usr/local/app.22
   ENV PATH $JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
   
   EXPOSE 8080
   
   CMD /usr/local/app.22/bin/startup.sh && tail -F /url/local/app.22/bin/logs/catalina.out
   ```

3. 构建镜像

   ```shell
   docker build -t diydocker
   ```

4. 启动

   ```shell
   docker run -d -p 9090:8080 --name kuangshen -v /home/kuangshen/build/tomcat/test:/usr/local/app.22/webapps/test -v /home/kuangshen/build/tomcat/logs:/usr/local/app.22/log diytomcat
   
   docker exec -it dockername /bin/bash
   
   # 测试
   
   ```

5. 访问测试

6. 发布项目（由于做了卷挂载）

   <img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710231113501.png" alt="image-20230710231113501" style="zoom:67%;" />

掌握dockerfile的编写！之后的一切都是使用docker镜像来发布运行！

### 发布自己的镜像

DockerHub

1. 注册账号

   

阿里云



### Docker流程小结

<img src="C:\Users\zzy\AppData\Roaming\Typora\typora-user-images\image-20230710231914263.png" alt="image-20230710231914263" style="zoom:50%;" />

[33、Docker所有流程小结_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1og4y1q7M4?p=33&vd_source=f45ef0e6dcb694562d9daa795d12407e)



## Docker网络
