Docker 官网：https://docs.docker.com/ 
Docker 的 github 地址：https://github.com/moby/moby 
 
Dockerhub 官网 
https://registry.hub.docker.com 
 
如果 docker 官方 registry 拉取镜像速度很慢，可以尝试 daocloud 提供的加速器服务 
https://dashboard.daocloud.io/mirror 
 
 
1、docker 是什么？ 
Docker 是一个开源项目，诞生于 2013 年初，最初是 dotCloud 公司内部的一个业余项目。它基于 
Google 公司推出的 Go 语言实现。项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代
码在 GitHub 上进行维护。 
 
Docker 是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开
发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何支持 docker 的机器上运
行。容器是完全使用沙箱机制，相互之间不会有任何接口调用。 
 
Docker logo： 
 
Docker 的思想来自于集装箱，集装箱解决了什么问题？在一艘大船上，可以把货物规整的摆放起来。并
且各种各样的货物被装在集装箱里，集装箱和集装箱之间不会互相影响。那么我就不需要专门运送蔬菜
的船和专门运送货物的船了。只要这些货物在集装箱里封装的好好的，那我就可以用一艘大船把他们都
运走。 
docker 就是类似的理念。云计算就好比大货轮。docker 就是集装箱。 
2、docker 的优点 
1）快 
运行时的性能快，管理操作(启动，停止，开始，重启等等) 都是以秒或毫秒为单位的。 
2）敏捷 
像虚拟机一样敏捷，而且会更便宜，在 bare metal(裸机)上布署像点个按钮一样简单。 
3）灵活 
将应用和系统“容器化”，不添加额外的操作系统 
4）轻量 
在一台服务器上可以布署 100~1000 个 Containers 容器。 
5）便宜 
开源的，免费的，低成本的。 
 
docker-ce：社区版 
docker-ee: 商业版 
 
 
3、docker 缺点 
所有容器共用 linux kernel 资源，资源能否实现最大限度利用，所以在安全上也会存在漏洞。 
4、安装 Docker 
主机 ip：192.168.40.180 
Centos7.6-centos7.9 
 
4Gib/4vCPU 
 
配置主机名： 
[root@ localhost ~]# hostnamectl set-hostname  localhost && bash 
 
关闭防火墙 
[root@ localhost ~]# systemctl stop firewalld && systemctl disable firewalld 
 
关闭 iptables 防火墙 
[root@ localhost ~]# yum install iptables-services -y #安装 iptables 
禁用 iptables 
[root@ localhost ~]# service iptables stop && systemctl disable iptables 
清空防火墙规则 
[root@ localhost~]# iptables -F 
 
关闭 selinux 
[root@ localhost ~]# setenforce 0 
[root@ localhost ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' 
/etc/selinux/config 
注意：修改 selinux 配置文件之后，重启机器，selinux 才能永久生效 
 
[root@ localhost ~]# getenforce 
显示 Disabled 表示 selinux 关闭成功 
 
#配置时间同步 
[root@ localhost ~]# yum install -y ntp ntpdate 
[root@ localhost ~]# ntpdate cn.pool.ntp.org 
#编写计划任务 
[root@ localhost ~]# crontab -e 
* */1 * * * /usr/sbin/ntpdate cn.pool.ntp.org 
重启 crond 服务使配置生效： 
[root@ localhost ~]# systemctl restart crond 
 
安装基础软件包 
[root@ localhost ~]# yum install -y wget net-tools nfs-utils lrzsz gcc gcc-c++ 
make cmake libxml2-devel openssl-devel curl curl-devel unzip sudo ntp libaio-devel wget vim 
ncurses-devel autoconf automake zlib-devel python-devel epel-release openssh-server 
socat ipvsadm conntrack 
 
#安装 docker-ce 
配置 docker-ce 国内 yum 源（阿里云） 
[root@ localhost ~]# yum-config-manager --add-repo 
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
 
 
安装 docker 依赖包 
[root@ localhost ~]# yum install -y yum-utils device-mapper-persistent-data lvm2 
 
安装 docker-ce 
[root@ localhost ~]# yum install docker-ce -y 
 
 
#启动 docker 服务 
[root@ localhost ~]# systemctl start docker && systemctl enable docker 
[root@ localhost ~]# systemctl status docker 
● docker.service - Docker Application Container Engine 
 Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: 
disabled) 
 Active: active (running) since Thu 2021-07-01 21:29:18 CST; 30s ago 
 Docs: https://docs.docker.com 
 
看到 running，表示 docker 正常运行 
 
#查看 Docker 版本信息 
[root@ localhost ~]# docker version 
 
5、开启包转发功能和修改内核参数 
内核参数修改：br_netfilter 模块用于将桥接流量转发至 iptables 链，br_netfilter 内核参数需要开
启转发。 
[root@ localhost ~]# modprobe br_netfilter 
[root@ localhost ~]# cat > /etc/sysctl.d/docker.conf <<EOF 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
net.ipv4.ip_forward = 1 
EOF 
 
#使参数生效 
[root@ localhost ~]# sysctl -p /etc/sysctl.d/docker.conf 
 
重启后模块失效，下面是开机自动加载模块的脚本 
在/etc/新建 rc.sysinit 文件 
cat /etc/rc.sysinit 
#!/bin/bash 
for file in /etc/sysconfig/modules/*.modules ; do 
[ -x $file ] && $file 
done 
 
在/etc/sysconfig/modules/目录下新建文件如下 
cat /etc/sysconfig/modules/br_netfilter.modules 
modprobe br_netfilter 
 
增加权限 
[root@ localhost ~]# chmod 755 /etc/sysconfig/modules/br_netfilter.modules 
 
重启机器模块也会自动加载 
[root@localhost ~]# lsmod |grep br_netfilter 
br_netfilter 22209 0 
bridge 136173 1 br_netfilter 
 
注： 
Docker 安装后出现：WARNING: bridge-nf-call-iptables is disabled 的解决办法： 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
 
net.ipv4.ip_forward = 1： 
将 Linux 系统作为路由或者 VPN 服务就必须要开启 IP 转发功能。当 linux 主机有多个网卡时一个网卡收
到的信息是否能够传递给其他的网卡 ，如果设置成 1 的话 可以进行数据包转发，可以实现 VxLAN 等功
能。不开启会导致 docker 部署应用无法访问。 
 
#重启 docker 
[root@ localhost ~]# systemctl restart docker 
 
6、配置 docker 镜像加速器 
登陆阿里云镜像仓库 
https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors 
如果没有开通，可开通阿里云的镜像服务 
 
 
 
找到镜像加速器，然后按照箭头方向操作 
修改/etc/docker/daemon.json，变成如下 
{ 
 "registry-mirrors":["https://y8y6vosv.mirror.aliyuncs.com","https://registry.docker￾cn.com","https://docker.mirrors.ustc.edu.cn","https://dockerhub.azk8s.cn","http://hub￾mirror.c.163.com"] 
} 
 
让配置文件生效 
sudo systemctl daemon-reload 
sudo systemctl restart docker 
 
 
 
7、docker 的基本用法 
7.1 镜像相关操作 
#从 dockerhub 查找镜像 
[root@ localhost ~]# docker search centos 
NAME DESCRIPTION STARS 
OFFICIAL AUTOMATED 
centos The official build of CentOS. 6639 
[OK] 
ansible/centos7-ansible Ansible on Centos7 134 
[OK] 
consol/centos-xfce-vnc Centos container with "headless" VNC session… 129 
[OK] 
jdeathe/centos-ssh OpenSSH / Supervisor / EPEL/IUS/SCL Repos - … 118 
[OK] 
 
解释说明： 
NAME: 镜像仓库源的名称 
DESCRIPTION: 镜像的描述 
OFFICIAL: 是否 docker 官方发布 
stars: 类似 Github 里面的 star，表示点赞、喜欢的意思。 
AUTOMATED: 自动构建。 
 
 
#下载镜像 
[root@ localhost ~]# docker pull centos 
 
#查看本地镜像 
[root@ localhost ~]# docker images 
 
 
#把镜像做成离线压缩包 
[root@ localhost ~]# docker save -o centos.tar.gz centos 
 
#解压离线镜像包 
[root@ localhost ~]# docker load -i centos.tar.gz 
 
#删除镜像 
[root@ localhost ~]# docker rmi -f centos:latest 
 
7.2 容器相关操作 
7.2.1 以交互式方式启动并进入容器 
[root@ localhost ~]# docker run --name=hello -it centos /bin/bash 
[root@09c4933b5cd7 /]# 
 
输入 exit，退出容器，退出之后容器也会停止，不会再前台运行 
 
#docker run 运行并创建容器 
--name 容器的名字 
-i 交互式 
-t 分配伪终端 
centos: 启动 docker 需要的镜像 
/bin/bash 说明你的 shell 类型为 bash，bash shell 是最常用的一种 shell, 是大多数 Linux 发行版默
认的 shell。 此外还有 C shell 等其它 shell。 
 
7.2.2 以守护进程方式启动容器 
[root@ localhost ~]# docker run --name=hello1 -td centos 
[root@ localhost ~]# docker ps |grep hello1 
1a2b73ba0ac2 centos "/bin/bash" hello1 
 
-d 在后台运行 docker 
[root@ localhost ~]# docker exec -it hello1 /bin/bash 
 
7.2.3 查看正在运行的容器 
docker ps 
 
 
 
[root@ localhost ~]# docker ps -a #查看所有容器，包括运行和退出的容器 
 
7.2.4 停止容器 
docker stop hello1 
7.2.5 启动已经停止的容器 
docker start hello1 
7.2.6 进入容器 
docker exec -it hello1 /bin/bash 
 
[root@ localhost ~]# docker rm -f hello1 #删除容器 
 
[root@ localhost ~]# docker --help 
#查看 docker 帮助命令 
 
8、通过 docker 部署 nginx 服务 
[root@ localhost ~]# docker run --name nginx -p 80 -itd centos 
 
#-p 把容器端口随机在物理机随机映射一个端口 
 
[root@ localhost ~]# docker ps | grep nginx 
ecfa046e9681 centos "/bin/bash" 
5 seconds ago Up 4 seconds 0.0.0.0:49153->80/tcp, :::49153->80/tcp nginx 
 
# 在 docker 里安装 nginx 
docker exec -it nginx /bin/bash 
[root@ecfa046e9681]# ip addr 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000 
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 
 inet 127.0.0.1/8 scope host lo 
 valid_lft forever preferred_lft forever 
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000 
 link/ipip 0.0.0.0 brd 0.0.0.0 
21: eth0@if22: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group 
default 
 link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0 
 inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0 
 valid_lft forever preferred_lft forever 
#通过上面可以看到容器的 ip 是 172.17.0.3 
 
#yum 安装 nginx 
rm -rf /etc/yum.repos.d/* 
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-
8.5.2111.repo 
 
 
yum install wget -y 
yum install nginx -y 
 
安装文本编辑器 vim 
yum install vim-enhanced -y 
 
创建静态页面 
mkdir /var/www/html -p 
cd /var/www/html/ 
cat index.html 
<html> 
 <head> 
 <title>nginx in docker</title> 
 </head> 
 <body> 
 <h1>hello,My Name is xianchao</h1> 
 </body> 
</html> 
 
修改 nginx 配置文件中的 root 路径，如下 
vim /etc/nginx/nginx.conf 
root /var/www/html/; 
 
启动 nginx 
/usr/sbin/nginx 
 
访问 docker 里的 nginx 服务,复制一个终端窗口，执行如下命令 
[root@ localhost ~]# docker ps | grep nginx 
ecfa046e9681 centos "/bin/bash" 
12 minutes ago Up 12 minutes 0.0.0.0:49153->80/tcp, :::49153->80/tcp nginx 
 
#能查看到 nginx 容器在物理机映射的端口是 49153 
 
[root@ localhost ~]# curl http://192.168.40.180:49153 
<html> 
 <head> 
 <title>nginx in docker</title> 
 </head> 
 <body> 
 <h1>hello,My Name is xianchao</h1> 
 </body> 
</html> 
 
 
 
也可以直接访问容器的 ip:port 
[root@ localhost ~]# curl 172.17.0.3:80 
<html> 
 <head> 
 <title>nginx in docker</title> 
 </head> 
 <body> 
 <h1>hello,My Name is xianchao</h1> 
 </body> 
</html> 
 
流量走向： 
访问物理节点 ip:port（容器在物理节点映射的端口）--→容器 ip:port（容器里部署的服务的端口）->
就可以访问到容器里部署的应用了 
 
9、dockerfile 语法详解 
Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。 
 
基于 Dockerfile 构建镜像可以使用 docker build 命令。docker build 命令中使用-f 可以指定具体的
dockerfile 文件 
 
FROM centos 
MAINTAINER xianchao 
RUN rm -rf /etc/yum.repos.d/* 
COPY Centos-vault-8.5.2111.repo /etc/yum.repos.d/ 
RUN yum install wget -y 
RUN yum install nginx -y 
COPY index.html /usr/share/nginx/html/ 
EXPOSE 80 
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"] 
 
#创建 index.html 
vim index.html 
<html> 
<head> 
 <title>page added to dockerfile</title> 
</head> 
<body> 
 <h1>i am in df_test </h1> 
</body> 
</html> 
 
dockerfile 构建过程： 
 
从基础镜像运行一个容器 
执行一条指令，对容器做出修改 
执行类似 docker commit 的操作，提交一个新的镜像层 
再基于刚提交的镜像运行一个新的容器 
执行 dockerfile 中的下一条指令，直至所有指令执行完毕 
 
 
（1）FROM 
基础镜像，必须是可以下载下来的，定制的镜像都是基于 FROM 的镜像，这里的 centos 就是定制需要的
基础镜像。后续的操作都是基于 centos 镜像。 
（2）MAINTAINER 
指定镜像的作者信息 
 
（3）RUN：指定在当前镜像构建过程中要运行的命令 
包含两种模式 
1、Shell 
RUN <command> (shell 模式，这个是最常用的，需要记住) 
RUN echo hello 
 
2、exec 模式 
RUN [“executable”，“param1”，“param2”](exec 模式) 
RUN [“/bin/bash”,”-c”,”echo hello”] 
等价于/bin/bash -c echo hello 
 
（4）EXPOSE 指令 
仅仅只是声明端口。 
作用： 
1、帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。 
2、在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。 
 
3、可以是一个或者多个端口，也可以指定多个 EXPOSE 
 
格式：EXPOSE <端口 1> [<端口 2>...] 
 
（5）CMD 
类似于 RUN 指令，用于运行程序，但二者运行的时间点不同: 
1、CMD 在 docker run 时运行。 
2、RUN 是在 docker build 构建镜像时运行的 
 
作用：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被
docker run 命令行参数中指定要运行的程序所覆盖。
 
CMD[“executable”，“param1”，“param2”]（exec 模式） 
 
CMD command （shell 模式） 
CMD [“param1”,”param2”](作为 ENTRYPOINT 指令的默认参数) 
 
例：cd /root/dockerfile/test 
cat dockerfile 
#first dockerfile 
FROM centos 
MAINTAINER xianchao 
RUN yum clean all 
RUN yum install nginx -y 
EXPOSE 80 
CMD ["/usr/sbin/nginx","-g","daemon off;"] 
 
构建镜像： 
docker build -t="dockerfile/test-cmd:v1" . 
 
基于上面构建的镜像运行一个容器 
docker run -p 80 --name cmd_test2 -d dockerfile/test-cmd:v1 
（不需要跟 nginx -g “daemon off；”了） 
 
docker ps 可以看到下面信息 
b903d5a71279 dockerfile/test-cmd:v1 "/usr/sbin/nginx -
g …" 7 seconds ago Up 6 seconds 0.0.0.0:32770->80/tcp cmd_test2 
 
（6）ENTERYPOINT 
类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被
当作参数送给 ENTRYPOINT 指令指定的程序。 
但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 entrypoint 指令指定的程序。 
优点：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。 
注意：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。 
格式： 
 
ENTERYPOINT [“executable”,“param1”,“param2”](exec 模式) 
ENTERYPOINT command （shell 模式） 
 
可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下
示例会提到。 
示例： 
假设已通过 Dockerfile 构建了 nginx:test 镜像： 
FROM nginx 
ENTRYPOINT ["nginx", "-c"] # 定参 
CMD ["/etc/nginx/nginx.conf"] # 变参 
 
 
1、不传参运行 
$ docker run nginx:test 
 
容器内会默认运行以下命令，启动主进程。 
nginx -c /etc/nginx/nginx.conf 
 
2、传参运行 
$ docker run nginx:test -c /etc/nginx/new.conf 
容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件) 
nginx -c /etc/nginx/new.conf 
 
（7）COPY 
COPY<src>..<dest> 
COPY[“<src>”...“<dest>”] 
 
复制指令，从上下文目录中复制文件或者目录到容器里指定路径。 
 
格式： 
COPY [--chown=<user>:<group>] <源路径 1>... <目标路径> 
COPY [--chown=<user>:<group>] ["<源路径 1>",... "<目标路径>"] 
 
[--chown=<user>:<group>]：可选参数，用户改变复制到容器内文件的拥有者和属组。 
 
<源路径>：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 
规则。例如： 
 
COPY hom* /mydir/ 
COPY hom?.txt /mydir/ 
 
<目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。 
 
 
（8）ADD 
ADD <src>...<dest> 
ADD [“<src>”...“<dest>”] 
 
ADD 指令和 COPY 的使用格式一致（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下： 
 
ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，
会自动复制并解压到 <目标路径>。 
ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像
构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。 
 
 
 
 
ADD vs COPY 
ADD 包含类似 tar 的解压功能 
如果单纯复制文件，dockerfile 推荐使用 COPY 
 
 
例;替换/usr/share/nginx 下的 index.html 
cd /root/dockerfile/test1 
cat dockerfile 
FROM centos 
MAINTAINER xianchao 
RUN yum install wget -y 
RUN yum install nginx -y 
COPY index.html /usr/share/nginx/html/ 
EXPOSE 80 
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"] 
 
vim index.html 
<html> 
<head> 
 <title>page added to dockerfile</title> 
</head> 
<body> 
 <h1>i am in df_test </h1> 
</body> 
</html> 
 
docker build -t="dockerfile/copy:v1" . 
 
docker run -d -p 80 --name html3 dockerfile/copy:v1 
 
docker ps | grep html3 
显示如下： 
478868402ac4 dockerfile/copy:v1 "/usr/sbin/nginx -
g …" 15 seconds ago Up 12 seconds 0.0.0.0:32771->80/tcp html3 
 
curl 192.168.40.180:32771 
显示的就是替换后的页面 
<html> 
<head> 
 <title>page added to dockerfile</title> 
</head> 
<body> 
 
 
 <h1>i am in df_test </h1> 
</body> 
</html> 
 
（9）VOLUME 
定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。 
作用： 
1、避免重要的数据，因容器重启而丢失，这是非常致命的。 
2、避免容器不断变大。 
格式： 
VOLUME ["<路径 1>", "<路径 2>"...] 
VOLUME <路径> 
在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。 
 
VOLUME [“/data”] 
 
（10）WORKDIR 
指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作
目录，必须是提前创建好的）。 
docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才
会一直存在。 
格式： 
WORKDIR <工作目录路径> 
 
WORKDIR /path/to/workdir 
（填写绝对路径） 
 
(11 )ENV 
设置环境变量 
ENV <key> <value> 
ENV <key>=<value>... 
 
以下示例设置 NODE_VERSION =6.6.6， 在后续的指令中可以通过 $NODE_VERSION 引用： 
ENV NODE_VERSION 6.6.6 
 
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" 
\ 
 && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" 
 
（12）USER 
用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已
经存在）。 
格式： 
 
 
USER <用户名>[:<用户组>] 
 
USER daemon 
USER nginx 
USER user 
USER uid 
USER user:group 
USER uid:gid 
USER user:gid 
USER uid:group 
 
（13）ONBUILD 
用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜
像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM 
test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 
ONBUILD 指定的命令。 
格式： 
ONBUILD <其它指令> 
 
为镜像添加触发器 
当一个镜像被其他镜像作为基础镜像时需要写上 OBNBUILD 
会在构建时插入触发器指令 
 
例：演示 ONBUILD 指令 
cat index.html 
<html> 
<head> 
 <title>page added to dockerfile</title> 
</head> 
<body> 
 <h1>i am in df_test </h1> 
</body> 
</html> 
 
vim dockerfile 
FROM centos 
MAINTAINER xianchao 
RUN yum install wget -y 
RUN yum install nginx -y 
ONBUILD COPY index.html /usr/share/nginx/html/ 
EXPOSE 80 
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"] 
 
 
 
docker build -t="onbuild-nginx:v1" . 
docker run -d --name html4 -p 80 onbuild-nginx:v1 
docker ps | grep html4 
显示如下： 
65f4a5be9355 onbuild-nginx:v1 "/usr/sbin/nginx -
g …" 14 seconds ago Up 11 seconds 0.0.0.0:32772->80/tcp html4 
 
curl 192.168.40.180:32772 
 
显示还是以前 nginx 默认的内容，没有被替换，表示 ONBUILD 这个指令后面的 COPY 没有生效 
 
 
还是在刚在路径下构建新的镜像 
vim dockerfile 
FROM onbuild-nginx:v1 
MAINTAINER xianchao 
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"] 
EXPOSE 80 
 
docker build -t="onbuild-nginx1" . 
docker run -d --name htm5 -p 80 onbuild-nginx1 
docker ps | grep htm5 
显示如下： 
e56542310692 onbuild-nginx1 "/usr/sbin/nginx -
g …" 12 seconds ago Up 8 seconds 0.0.0.0:32773->80/tcp htm5 
 
curl 192.168.40.180:32773 
显示如下： 
<html> 
<head> 
 <title>page added to dockerfile</title> 
</head> 
<body> 
 <h1>i am in df_test </h1> 
</body> 
</html> 
 
显示的就是已经重新构建的镜像，页面就是替换之后的了，说明我们基于 ONBUILD 指令的镜像作为基础
镜像，在构建镜像，会触发 ONBUILD 后面的 COPY 命令运行 
 
（14）LABEL 
LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式，语法格式如下：
LABEL <key>=<value> <key>=<value> <key>=<value> ...
 
 
比如我们可以添加镜像的作者：
LABEL org.opencontainers.image.authors="xianchao"
 
（15）HEALTHCHECK 
用于指定某个程序或者指令来监控 docker 容器服务的运行状态。
格式：
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
（16）ARG 
构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就
是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。
构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。
格式：
ARG <参数名>[=<默认值>]
 
10、dockerfile 构建 nginx 镜像 
mkdir dockerfile 
cd dockerfile/ 
vim dockerfile 
FROM centos 
MAINTAINER xianchao 
RUN rm -rf /etc/yum.repos.d/* 
COPY Centos-vault-8.5.2111.repo /etc/yum.repos.d/ 
RUN yum install wget -y 
RUN yum install nginx -y 
COPY index.html /usr/share/nginx/html/ 
EXPOSE 80 
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"] 
vim index.html 
<html> 
<head> 
 <title>page added to dockerfile</title> 
</head> 
<body> 
 <h1>Hello，My Name is Xianchao,My wechat is luckylucky421302 </h1> 
</body> 
 
 
</html> 
构建镜像： 
docker build -t="xianchao/nginx:v1" . 
查看镜像是否构建成功： 
docker images | grep nginx 
显示如下说明镜像部署成功： 
dockerfile/nginx latest baee97a76499 About a minute ago 344MB 
基于刚才的镜像启动容器： 
docker run -d -p 80 --name html2 xianchao/nginx:v1 
查看容器具体信息可按如下命令: 
[root@ localhost dockerfile]# docker ps | grep html 
bdbe140d5dc9 xianchao/nginx:v1 "/usr/sbin/nginx -g …" 
17 seconds ago Up 15 seconds 0.0.0.0:49154->80/tcp, :::49154->80/tcp html2 
 
查看容器里部署的 nginx 网站的内容： 
curl http://192.168.40.180:49154 
显示如下： 
<html> 
<head> 
 <title>page added to dockerfile</title> 
</head> 
<body> 
 <h1>Hello，My Name is Xianchao,My wechat is luckylucky421302 </h1> 
</body> 
</html> 
 
注意：ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"] 
表示容器运行时，自动启动容器里的 nginx 服务 
 
11、dockerfile 构建 tomcat 镜像 
下面需要的压缩包在下面需要的压缩包在课件 
 
mkdir tomcat8 
cd tomcat8 
把 apache-tomcat-8.0.26.tar.gz 和 jdk-8u45-linux-x64.rpm 传到这个目录下 
ls 可以查看到 tomcat8 下面有如下几个文件 
 
cat dockerfile 
FROM centos 
MAINTAINER xianchao 
RUN rm -rf /etc/yum.repos.d/* 
COPY Centos-vault-8.5.2111.repo /etc/yum.repos.d/ 
 
RUN yum install wget -y 
ADD jdk-8u45-linux-x64.rpm /usr/local/ 
ADD apache-tomcat-8.0.26.tar.gz /usr/local/ 
RUN cd /usr/local && rpm -ivh jdk-8u45-linux-x64.rpm 
RUN mv /usr/local/apache-tomcat-8.0.26 /usr/local/tomcat8 
EXPOSE 8080 
 
开始构建镜像 
docker build -t="tomcat8:v1" . 
运行一个容器 
docker run --name tomcat8 -itd -p 8080 tomcat8:v1 
 
进入到容器 
docker exec -it tomcat8 /bin/bash 
启动 tomcat 
/usr/local/tomcat8/bin/startup.sh 
查看进程，看看是否启动成功 
ps -ef | grep tomcat 
打开新的终端窗口，查看刚才创建的 tomcat8 这个容器的详细信息： 
docker ps | grep tomcat 
显示如下信息 
4d4c91cff4b5 tomcat8:v1 "/bin/bash" 
About a minute ago Up About a minute 0.0.0.0:32776->8080/tcp tomcat8 
 
通过上面可以看到，tomcat 在宿主机上映射的端口是 32776 
这样我们请求 docker 节点的 ip:32776，就可以访问到 tomcat 的内容了 
 
通过这些步骤可以实现通过 dockerfile 构建 tomcat 镜像了 
 
刚才我们在构建 tomcat 镜像时候，基于镜像运行容器，但是需要进入到容器，手动启动 tomcat 服务，
那如果想要启动容器，tomcat 也自动起来，需要按照如下方法构建镜像： 
 
FROM centos 
MAINTAINER xianchao 
RUN rm -rf /etc/yum.repos.d/* 
COPY Centos-vault-8.5.2111.repo /etc/yum.repos.d/ 
RUN yum install wget -y 
ADD jdk-8u45-linux-x64.rpm /usr/local/ 
ADD apache-tomcat-8.0.26.tar.gz /usr/local/ 
RUN cd /usr/local && rpm -ivh jdk-8u45-linux-x64.rpm 
RUN mv /usr/local/apache-tomcat-8.0.26 /usr/local/tomcat8 
ENTRYPOINT /usr/local/tomcat8/bin/startup.sh && tail -F 
/usr/local/tomcat8/logs/catalina.out 
 
 
EXPOSE 8080 
 
 
作业：dockerfile 构建 PHP 镜像 
 或者把公司的项目做成镜像测试下 
 
 
12、Docker 容器的数据管理 
Docker 容器的数据卷 
什么是数据卷？ 
数据卷是经过特殊设计的目录，可以绕过联合文件系统（UFS），为一个或者多个容器提供访问，数据卷
设计的目的，在于数据的永久存储，它完全独立于容器的生存周期，因此，docker 不会在容器删除时删
除其挂载的数据卷，也不会存在类似的垃圾收集机制，对容器引用的数据卷进行处理，同一个数据卷可
以只支持多个容器的访问。 
 
数据卷的特点： 
1.数据卷在容器启动时初始化，如果容器使用的镜像在挂载点包含了数据，这些数据会被拷贝到新初始
化的数据卷中 
2.数据卷可以在容器之间共享和重用 
3.可以对数据卷里的内容直接进行修改 
4.数据卷的变化不会影像镜像的更新 
5.卷会一直存在，即使挂载数据卷的容器已经被删除 
 
数据卷的使用： 
1.为容器添加数据卷 
docker run -v /datavolume:/data -it centos /bin/bash 
 
如： 
docker run --name volume -v ~/datavolume:/data -itd centos /bin/bash 
 
注：~/datavolume 为宿主机目录，/data 为 docker 启动的 volume 容器的里的目录 
这样在宿主机的/datavolume 目录下创建的数据就会同步到容器的/data 目录下 
 
（1）为数据卷添加访问权限 
docker run --name volume1 -v ~/datavolume1:/data:ro -itd centos /bin/bash 
 
添加只读权限之后在 docker 容器的/data 目录下就不能在创建文件了，为只读权限；在宿主机下的
/datavolume1 下可以创建东西 
 
2.使用 dockerfile 构建包含数据卷的镜像 
dockerfile 指令： 
 
volume[“/data”] 
 
 
cat dockerfile 
FROM centos 
VOLUME ["/datavolume3","/datavolume6"] 
CMD /bin/bash 
 
使用如下构建镜像 
docker build -t="volume" . 
 
启动容器 
docker run --name volume-dubble -it volume 
 
会看到这个容器下有两个目录，/datavolume3 和/datavolume6 
 
13、Docker 的数据卷容器 
什么是数据卷容器： 
命名的容器挂载数据卷，其他容器通过挂载这个容器实现数据共享，挂载数据卷的容器，就叫做数据卷
容器 
挂载数据卷容器的方法 
docker run --volumes-from [container name] 
例： 
docker run --name data-volume -itd volume（volume 这个镜像是上面创建的带两个数据卷
/datavolume3 和/ddatavolume6 的镜像） 
 
docker exec -it data-volume /bin/bash（进入到容器中） 
 
touch /datavolume6/lucky.txt 
退出容器 exit 
 
创建一个新容器挂载刚才 data-volume 这个容器创建的数据卷 
docker run --name data-volume2 --volumes-from data-volume -itd centos /bin/bash 
 
进入到新创建的容器 
docker exec -it data-volume2 /bin/bash 
 
查看容器的/datavolume6 目录下是否新创建了 lucky.txt 文件 
cd /datavolume6 
可以看见有刚才在上一个容器创建的文件 lucky.txt 
 
14、docker 数据卷的备份和还原 
数据备份方法： 
docker run --volumes-from [container name] -v $(pwd):/backup centos tar czvf 
/backup/backup.tar [container data volume] 
 
 
 
例子： 
docker run --volumes-from data-volume2 -v /root/backup:/backup --name datavolume-copy 
centos tar zcvf /backup/data-volume2.tar.gz /datavolume6 
 
数据还原方法： 
docker run --volumes-from [container name] -v $(pwd):/backup centos tar xzvf 
/backup/backup.tar.gz [container data volume] 
 
例： 
docker exec -it data-volume2 /bin/bash 
cd /datavolume6 
rm -rf lucky.txt 
 
docker run --volumes-from data-volume2 -v /root/backup/:/backup centos tar zxvf 
/backup/data-volume2.tar.gz -C /datavolume6 
 
docker exec -it data-volum2 /bin/bash 
cd /datavolum6 
可以看到还原后的数据 
 
 
15、docker 容器互联 
 
docker run 创建 Docker 容器时，可以用--net 选项指定容器的网络模式，Docker 有以下 4 种网络模式： 
 bridge 模式：使--net =bridge 指定，默认设置； 
 host 模式：使--net =host 指定； 
 none 模式：使--net =none 指定； 
 container 模式：使用--net =container:NAME orID 指定。 
 
15.1 docker 容器的网络基础 
1、docker0： 
安装 docker 的时候，会生成一个 docker0 的虚拟网桥 
 
2、Linux 虚拟网桥的特点： 
可以设置 ip 地址 
相当于拥有一个隐藏的虚拟网卡 
 
docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
 link/ether 02:42:28:ae:c0:42 brd ff:ff:ff:ff:ff:ff 
 inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0 
 valid_lft forever preferred_lft forever 
inet6 fe80::42:28ff:feae:c042/64 scope link 
 
 
每运行一个 docker 容器都会生成一个 veth 设备对，这个 veth 一个接口在容器里，一个接口在物理机
上。 
 
 
3、安装网桥管理工具： 
yum install bridge-utils -y 
 
brctl show 可以查看到有一个 docker0 的网桥设备，下面有很多接口，每个接口都表示一个启动的
docker 容器，因为我在 docker 上启动了很多容器，所以 interfaces 较多，如下所示： 
 
 
 
15.2 docker 容器的互联 
下面用到的镜像的 dockerfile 文件如下： 
cd dockerfile/inter-image 
vim dockerfile 
FROM centos 
RUN yum install wget -y 
RUN yum install nginx -y 
RUN sed -i "7s/^/#/g" /etc/nginx/conf.d/default.conf 
EXPOSE 80 
CMD /bin/bash 
 
 
[root@ localhost]# docker build -t="inter-image" . 
 
允许所有容器间互联（也就是访问） 
第一种方法： 
例： 
（1）基于上面的 inter-image 镜像启动第一个容器 test1 
docker run --name test1 -itd inter-image 
进入到容器里面启动 nginx： 
/usr/sbin/ngnx 
 
 
（2）基于上面的 inter-image 镜像启动第二个容器 test2 
docker run --name test2 -itd inter-image 
 
 
（3）进入到 test1 容器和 test2 容器，可以看两个容器的 ip，分别是 
172.17.0.20 和 172.17.0.21 
 
docker exec -it test2 /bin/bash 
ping 172.17.0.20 可以看见能 ping 同 test1 容器的 ip 
 
curl http://172.17.0.20 
可以访问到 test1 容器的内容 
 
上述方法假如 test1 容器重启，那么在启动就会重新分配 ip 地址，所以为了使 ip 地址变了也可以访问
可以采用下面的方法 
 
15.2.1 docker link 设置网络别名 
第二种方法： 
可以给容器起一个代号，这样可以直接以代号访问，避免了容器重启 ip 变化带来的问题 
--link 
docker run --link=[CONTAINER_NAME]:[ALIAS] [IMAGE][COMMAND] 
例： 
1.启动一个 test3 容器 
docker run --name test3 -itd inter-image /bin/bash 
 
 
2.启动一个 test5 容器，--link 做链接，那么当我们重新启动 test3 容器时，就算 ip 变了，也没关系，
我们可以在 test5 上 ping 别名 webtest 
docker run --name test5 -itd --link=test3:webtest inter-image /bin/bash 
 
3.test3 和 test5 的 ip 分别是 172.17.0.22 和 172.17.0.24 
 
4.重启 test3 容器 
docker restart test3 
发现 ip 变成了 172.17.0.25 
 
5.进入到 test5 容器 
docker exec -it test5 /bin/bash 
 
ping test3 容器的 ip 别名 webtest 可以 ping 通，尽管 test3 容器的 ip 变了也可以通 
 
16、docker 容器的网络模式 
 
 
docker run 创建 docker 容器时，可以用--net 选项指定容器的网络模式，Docker 有以下 4 种网络模式： 
 bridge 模式：使--net =bridge 指定，默认设置； 
 host 模式：使--net =host 指定； 
 none 模式：使--net =none 指定； 
 container 模式：使用--net =container:NAME orID 指定。 
 
16.1 none 模式 
Docker 网络 none 模式是指创建的容器没有网络地址，只有 lo 网卡
 
[root@ localhost ~]# docker run -itd --name none --net=none --privileged=true centos 
[root@ localhost ~]# docker exec -it none /bin/bash 
 
[root@05dbf3f2daaf /]# ip addr 
#只有本地 lo 地址 
 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000 
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 
 inet 127.0.0.1/8 scope host lo 
 valid_lft forever preferred_lft forever 
 
16.2 container 模式 
Docker 网络 container 模式是指，创建新容器的时候，通过--net container 参数，指定其和已经存
在的某个容器共享一个 Network Namespace。如下图所示，右方黄色新创建的 container，其网卡共享左
边容器。因此就不会拥有自己独立的 IP，而是共享左边容器的 IP 172.17.0.2,端口范围等网络资源，两
个容器的进程通过 lo 网卡设备通信。
#和已经存在的 none 容器共享网络
[root@ localhost ~]# docker run --name container2 --net=container:none -it --
privileged=true centos
 
 
[root@05dbf3f2daaf /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
 inet 127.0.0.1/8 scope host lo
 valid_lft forever preferred_lft forever
16.3 bridge 模式 
默认选择 bridge 的情况下，容器启动后会通过 DHCP 获取一个地址
#创建桥接网络
[root@ localhost~]# docker run --name bridge -it --privileged=true centos bash
[root@a131580fb605 /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 
1000
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
 inet 127.0.0.1/8 scope host lo
 valid_lft forever preferred_lft forever
64: eth0@if65: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group 
default 
 link/ether 02:42:ac:11:00:0d brd ff:ff:ff:ff:ff:ff link-netnsid 0
 inet 172.17.0.13/16 brd 172.17.255.255 scope global eth0
 valid_lft forever preferred_lft forever
16.4 host 模式 
Docker 网络 host 模式是指共享宿主机的网络
#共享宿主机网络
[root@ localhost~]# docker run --name host -it --net=host --privileged=true centos 
bash
 
 
17、docker 资源配额 
17.1 docker 容器控制 cpu 
Docker 通过 cgroup 来控制容器使用的资源限制，可以对 docker 限制的资源包括 CPU、内存、磁盘
17.1.1 指定 docker 容器可以使用的 cpu 份额 
#查看配置份额的帮助命令： 
[root@ localhost ~]# docker run --help | grep cpu-shares 
cpu 配额参数：-c, --cpu-shares int 
CPU shares (relative weight) 在创建容器时指定容器所使用的 CPU 份额值。cpu-shares 的值不能
 
 
保证可以获得 1 个 vcpu 或者多少 GHz 的 CPU 资源，仅仅只是一个弹性的加权值。 
 
默认每个 docker 容器的 cpu 份额值都是 1024。在同一个 CPU 核心上，同时运行多个容器时，容器的
cpu 加权的效果才能体现出来。
 
例： 两个容器 A、B 的 cpu 份额分别为 1000 和 500，结果会怎么样？ 
情况 1：A 和 B 正常运行，占用同一个 CPU，在 cpu 进行时间片分配的时候，容器 A 比容器 B 多一倍
的机会获得 CPU 的时间片。 
 
情况 2：分配的结果取决于当时其他容器的运行状态。比如容器 A 的进程一直是空闲的，那么容器 B
是可以获取比容器 A 更多的 CPU 时间片的； 比如主机上只运行了一个容器，即使它的 cpu 份额只有
50，它也可以独占整个主机的 cpu 资源。
 
cgroups 只在多个容器同时争抢同一个 cpu 资源时，cpu 配额才会生效。因此，无法单纯根据某个容
器的 cpu 份额来确定有多少 cpu 资源分配给它，资源分配结果取决于同时运行的其他容器的 cpu 分
配和容器中进程运行情况。 
 
 
例 1：给容器实例分配 512 权重的 cpu 使用份额 
参数： --cpu-shares 512 
[root@ localhost ~]# docker run -it --cpu-shares 512 centos /bin/bash 
[root@df176dd75bd4 /]# cat /sys/fs/cgroup/cpu/cpu.shares #查看结果： 
512
 
注：稍后，我们启动多个容器，测试一下是不是只能使用 512 份额的 cpu 资源。单独一个容器，看
不出来使用的 cpu 的比例。 因没有 docker 实例同此 docker 实例竞争。
总结：
通过-c 设置的 cpu share 并不是 CPU 资源的绝对数量，而是一个相对的权重值。某个容器最终能
分配到的 CPU 资源取决于它的 cpu share 占所有容器 cpu share 总和的比例。通过 cpu share 
可以设置容器使用 CPU 的优先级。
比如在 host 中启动了两个容器：
docker run --name "container_A" -c 1024 ubuntu
docker run --name "container_B" -c 512 ubuntu
container_A 的 cpu share 1024，是 container_B 的两倍。当两个容器都需要 CPU 资源时，
container_A 可以得到的 CPU 是 container_B 的两倍。
需要注意的是，这种按权重分配 CPU 只会发生在 CPU 资源紧张的情况下。如果 container_A 处于空
闲状态，为了充分利用 CPU 资源，container_B 也可以分配到全部可用的 CPU。
 
 
 
17.1.2 CPU core 核心控制 
参数：--cpuset 可以绑定 CPU 
对多核 CPU 的服务器，docker 还可以控制容器运行限定使用哪些 cpu 内核和内存节点，即使用--
cpuset-cpus 和--cpuset-mems 参数。对具有 NUMA 拓扑（具有多 CPU、多内存节点）的服务器尤其有
用，可以对需要高性能计算的容器进行性能最优的配置。如果服务器只有一个内存节点，则--
cpuset-mems 的配置基本上不会有明显效果。
 
扩展： 
服务器架构一般分： SMP、NUMA、MPP 体系结构介绍 
从系统架构来看，目前的商用服务器大体可以分为三类： 
1. 即对称多处理器结构(SMP ： Symmetric Multi-Processor) 例： x86 服务器，双路服务
器。主板上有两个物理 cpu 
2. 非一致存储访问结构 (NUMA ： Non-Uniform Memory Access) 例： IBM 小型机
pSeries 690 
3. 海量并行处理结构 (MPP ： Massive ParallelProcessing) 。 例： 大型机 Z14 
 
17.1.3 CPU 配额控制参数的混合使用 
在上面这些参数中，cpu-shares 控制只发生在容器竞争同一个 cpu 的时间片时有效。 
如果通过 cpuset-cpus 指定容器 A 使用 cpu 0，容器 B 只是用 cpu1，在主机上只有这两个容器使用
对应内核的情况，它们各自占用全部的内核资源，cpu-shares 没有明显效果。 
如何才能有效果？
 
容器 A 和容器 B 配置上 cpuset-cpus 值并都绑定到同一个 cpu 上，然后同时抢占 cpu 资源，就可以
看出效果了。 
 
例 1：测试 cpu-shares 和 cpuset-cpus 混合使用运行效果，就需要一个压缩力测试工具 stress 来让
容器实例把 cpu 跑满。 
如何把 cpu 跑满？
如何把 4 核心的 cpu 中第一和第三核心跑满？可以运行 stress，然后使用 taskset 绑定一下 cpu。 
 
先扩展：stress 命令 
概述：linux 系统压力测试软件 Stress 。
 
 
[root@ localhost ~]# yum install -y epel-release 
[root@ localhost ~]# yum install stress -y 
 
stress 参数解释 
-? 显示帮助信息 
-v 显示版本号 
 
 
-q 不显示运行信息 
-n 显示已完成的指令情况 
-t --timeout N 指定运行 N 秒后停止 
--backoff N 等待 N 微妙后开始运行 
-c 产生 n 个进程 ：每个进程都反复不停的计算随机数的平方根，测试 cpu 
-i 产生 n 个进程 ：每个进程反复调用 sync()，sync()用于将内存上的内容写到硬
盘上，测试磁盘 io 
-m --vm n 产生 n 个进程,每个进程不断调用内存分配 malloc（）和内存释放 free（）函
数 ，测试内存 
--vm-bytes B 指定 malloc 时内存的字节数 （默认 256MB） 
--vm-hang N 指定在 free 栈的秒数 
-d --hadd n 产生 n 个执行 write 和 unlink 函数的进程 
-hadd-bytes B 指定写的字节数 
--hadd-noclean 不 unlink 
注：时间单位可以为秒 s，分 m，小时 h，天 d，年 y，文件大小单位可以为 K，M，G 
 
例 1：产生 2 个 cpu 进程，2 个 io 进程，20 秒后停止运行 
[root@ localhost]# stress -c 2 -i 2 --verbose --timeout 20s 
#如果执行时间为分钟，改 20s 为 1m 
 
查看：
top 
 
例 1：测试 cpuset-cpus 和 cpu-shares 混合使用运行效果，就需要一个压缩力测试工具 stress 来让
容器实例把 cpu 跑满。 当跑满后，会不会去其他 cpu 上运行。 如果没有在其他 cpu 上运行，说明
cgroup 资源限制成功。 
实例 3：创建两个容器实例:docker10 和 docker20。 让 docker10 和 docker20 只运行在 cpu0 和
cpu1 上，最终测试一下 docker10 和 docker20 使用 cpu 的百分比。实验拓扑图如下： 
 
 
 
运行两个容器实例 
[root@ localhost ~]# docker run -itd --name docker10 --cpuset-cpus 0,1 --cpu-shares 
512 centos /bin/bash 
#指定 docker10 只能在 cpu0 和 cpu1 上运行，而且 docker10 的使用 cpu 的份额 512 
#参数-itd 就是又能打开一个伪终端，又可以在后台运行着 docker 实例 
[root@ localhost ~]# docker run -itd --name docker20 --cpuset-cpus 0,1 --cpu-shares 
1024 centos /bin/bash 
#指定 docker20 只能在 cpu0 和 cpu1 上运行，而且 docker20 的使用 cpu 的份额 1024，比 dcker10 多
一倍 
 
测试 1： 进入 docker10，使用 stress 测试进程是不是只在 cpu0,1 上运行： 
[root@ localhost ~]# docker exec -it docker10 /bin/bash 
[root@d1a431815090 /]# yum install -y epel-release #安装 epel 扩展源 
[root@d1a431815090 /]# yum install stress -y #安装 stress 命令 
[root@d1a431815090 /]# stress -c 2 -v -t 10m #运行 2 个进程，把两个 cpu 占满 
在物理机另外一个虚拟终端上运行 top 命令，按 1 快捷键，查看每个 cpu 使用情况： 
 
可看到正常。只在 cpu0,1 上运行 
 
测试 2： 然后进入 docker20，使用 stress 测试进程是不是只在 cpu0,1 上运行，且 docker20 上运行
的 stress 使用 cpu 百分比是 docker10 的 2 倍 
[root@ localhost ~]# docker exec -it docker20 /bin/bash 
[root@d1a431815090 /]# yum install -y epel-release #安装 epel 扩展源 
[root@d1a431815090 /]# yum install stress -y 
 
 
[root@f24e75bca5c0 /]# stress -c 2 -v -t 10m 
 
在另外一个虚拟终端上运行 top 命令，按 1 快捷键，查看每个 cpu 使用情况： 
 
注：两个容器只在 cpu0,1 上运行，说明 cpu 绑定限制成功。而 docker20 是 docker10 使用 cpu 的 2
倍。说明--cpu-shares 限制资源成功。 
 
 
17.2 docker 容器控制内存 
Docker 提供参数-m, --memory=""限制容器的内存使用量。 
例 1：允许容器使用的内存上限为 128M： 
[root@ localhost ~]# docker run -it -m 128m centos 
查看： 
[root@40bf29765691 /]# cat /sys/fs/cgroup/memory/memory.limit_in_bytes 
134217728 
注：也可以使用 tress 进行测试，到现在，我可以限制 docker 实例使用 cpu 的核心数和权重，可以
限制内存大小。 
 
例 2：创建一个 docker，只使用 2 个 cpu 核心，只能使用 128M 内存 
[root@ localhost ~]# docker run -it --cpuset-cpus 0,1 -m 128m centos 
 
17.3 docker 容器控制 IO 
[root@ localhost ~]# docker run --help | grep write-b 
 --device-write-bps value Limit write rate (bytes per second) to a device 
(default []) 
#限制此设备上的写速度（bytes per second），单位可以是 kb、mb 或者 gb。 
--device-read-bps value #限制此设备上的读速度（bytes per second），单位可以是 kb、mb 或
者 gb。
 
情景：防止某个 Docker 容器吃光你的磁盘 I / O 资源
 
例 1：限制容器实例对硬盘的最高写入速度设定为 2MB/s。 
--device 参数：将主机设备添加到容器 
[root@ localhost ~]# mkdir -p /var/www/html/ 
 
 
[root@ localhost ~]# docker run -it -v /var/www/html/:/var/www/html --device 
/dev/sda:/dev/sda --device-write-bps /dev/sda:2mb centos /bin/bash
 
[root@bd79042dbdc9 /]# time dd if=/dev/sda of=/var/www/html/test.out bs=2M count=50 
oflag=direct,nonblock
 
注：dd 参数： 
direct：读写数据采用直接 IO 方式，不走缓存。直接从内存写硬盘上。 
nonblock：读写数据采用非阻塞 IO 方式，优先写 dd 命令的数据 
50+0 records in 
50+0 records out 
52428800 bytes (52 MB) copied, 50.1831 s, 2.0 MB/s 
 
real 0m50.201s 
user 0m0.001s 
sys 0m0.303s 
注： 发现 1 秒写 2M。 限制成功。
 
18、docker 容器运行结束自动释放资源 
[root@ localhost ~]# docker run --help | grep rm 
 --rm 参数： Automatically remove the container when it exits 
作用：当容器命令运行结束后，自动删除容器，自动释放资源 
 
例： 
[root@ localhost ~]# docker run -it --rm --name xianchao centos sleep 6 
物理上查看： 
[root@ localhost ~]# docker ps -a | grep xianchao 
6c75a9317a6b centos "sleep 6" 6 seconds ago Up 4 
seconds mk 
等 5s 后，再查看： 
[root@ localhost ~]# docker ps | grep xianchao #自动删除了
 
18、docker 私有镜像仓库 harbor 
Harbor 介绍
Docker 容器应用的开发和运行离不开可靠的镜像管理，虽然 Docker 官方也提供了公共的镜像仓库，
但是从安全和效率等方面考虑，部署我们私有环境内的 Registry 也是非常必要的。Harbor 是由 VMware
公司开源的企业级的 Docker Registry 管理项目，它包括权限管理(RBAC)、LDAP、日志审核、管理界面、
自我注册、镜像复制和中文支持等功能。
官网地址：https://github.com/goharbor/harbor
实验环境：
 
安装 harbor 的机器，主机名设置成 harbor
机器需要的内存至少要 2G，我分配的是 4G
机器 ip：192.168.40.181
4vCPU/4G 内存/100G 硬盘
18.1 为 Harbor 自签发证书 
[root@192 ~]# hostnamectl set-hostname harbor && bash
[root@harbor ~]# mkdir /data/ssl -p
[root@harbor ~]# cd /data/ssl/
生成 ca 证书：
openssl genrsa -out ca.key 3072
#生成一个 3072 位的 key，也就是私钥
openssl req -new -x509 -days 3650 -key ca.key -out ca.pem
#生成一个数字证书 ca.pem，3650 表示证书的有效时间是 3 年，按箭头提示填写即可，没有箭头标注的为
空：
 
 
生成域名的证书： 
openssl genrsa -out harbor.key 3072
#生成一个 3072 位的 key，也就是私钥
openssl req -new -key harbor.key -out harbor.csr
#生成一个证书请求，一会签发证书时需要的，标箭头的按提示填写，没有箭头标注的为空：
 
签发证书：
openssl x509 -req -in harbor.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out harbor.pem -
days 3650
显示如下，说明证书签发好了：
18.2 安装 Harbor 
18.2.1 安装 Docker 
 
关闭防火墙 
[root@ harbor~]# systemctl stop firewalld && systemctl disable firewalld 
 
关闭 iptables 防火墙 
[root@ harbor~]# yum install iptables-services -y #安装 iptables 
禁用 iptables 
root@ harbor~]# service iptables stop && systemctl disable iptables 
清空防火墙规则 
[root@ harbor~]# iptables -F 
 
关闭 selinux 
[root@ harbor~]# setenforce 0 
[root@harbor~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config 
注意：修改 selinux 配置文件之后，重启机器，selinux 才能永久生效 
 
#配置时间同步 
[root@harbor~]# yum install -y ntp ntpdate 
[root@ localhost ~]# ntpdate cn.pool.ntp.org 
#编写计划任务 
 
 
[root@harbor~]# crontab -e 
* */1 * * * /usr/sbin/ntpdate cn.pool.ntp.org 
重启 crond 服务使配置生效： 
[root@ localhost ~]# systemctl restart crond 
 
 配置 hosts 文件 
[root@ localhost ~]# cat /etc/hosts 
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4 
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6 
192.168.40.180  localhost 
192.168.40.181 harbor 
 
[root@harbor harbor]# cat /etc/hosts 
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4 
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6 
192.168.40.180  localhost 
192.168.40.181 harbor 
 
安装基础软件包 
[root@ harbor~]# yum install -y wget net-tools nfs-utils lrzsz gcc gcc-c++ make cmake 
libxml2-devel openssl-devel curl curl-devel unzip sudo ntp libaio-devel wget vim ncurses￾devel autoconf automake zlib-devel python-devel epel-release openssh-server 
socat ipvsadm conntrack 
 
#安装 docker-ce 
配置 docker-ce 国内 yum 源（阿里云） 
[root@ harbor~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker￾ce/linux/centos/docker-ce.repo 
 
安装 docker 依赖包 
[root@ harbor~]# yum install -y yum-utils device-mapper-persistent-data lvm2 
 
安装 docker-ce 
[root@ harbor~]# yum install docker-ce -y 
 
 
#启动 docker 服务 
[root@ harbor~]# systemctl start docker && systemctl enable docker 
[root@ harbor~]# systemctl status docker 
● docker.service - Docker Application Container Engine 
 Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: 
disabled) 
 Active: active (running) since Thu 2021-07-01 21:29:18 CST; 30s ago 
 
 
 Docs: https://docs.docker.com 
 
看到 running，表示 docker 正常运行 
 
#查看 Docker 版本信息 
[root@ harbor~]# docker version 
 
18.2.2 开启包转发功能和修改内核参数 
内核参数修改：br_netfilter 模块用于将桥接流量转发至 iptables 链，br_netfilter 内核参数需要开
启转发。 
[root@ harbor~]# modprobe br_netfilter 
[root@ harbor~]# cat > /etc/sysctl.d/docker.conf <<EOF 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
net.ipv4.ip_forward = 1 
EOF 
 [root@harbor ~]# sysctl -p /etc/sysctl.d/docker.conf 
 
注： 
Docker 安装后出现：WARNING: bridge-nf-call-iptables is disabled 的解决办法： 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
 
net.ipv4.ip_forward = 1： 
将 Linux 系统作为路由或者 VPN 服务就必须要开启 IP 转发功能。当 linux 主机有多个网卡时一个网卡收
到的信息是否能够传递给其他的网卡 如果设置成 1 的话 可以进行数据包转发，可以实现 VxLAN 等功
能。不开启会导致 docker 部署应用无法访问。 
 
#重启 docker 
[root@ localhost ~]# systemctl restart docker 
 
[root@ localhost ~]# scp /etc/docker/daemon.json 192.168.40.181:/etc/docker/
[root@harbor ~]# systemctl daemon-reload
[root@harbor ~]# systemctl restart docker
18.2.4 安装 harbor 
创建安装目录
[root@harbor ssl]# mkdir /data/install -p
[root@harbor ssl]# cd /data/install/
安装 harbor
/data/ssl 目录下有如下文件：
ca.key ca.pem ca.srl harbor.csr harbor.key harbor.pem
 
[root@harbor install]# cd /data/install/
#把 harbor 的离线包 harbor-offline-installer-v2.3.0-rc3.tgz 上传到这个目录，离线包在课件
里提供了
下载 harbor 离线包的地址：
https://github.com/goharbor/harbor/releases/tag/
解压：
[root@harbor install]# tar zxvf harbor-offline-installer-v2.3.0-rc3.tgz
[root@harbor install]# cd harbor
[root@harbor harbor]# cp harbor.yml.tmpl harbor.yml
[root@harbor harbor]# vim harbor.yml
修改配置文件：
hostname: harbor 
#修改 hostname，跟上面签发的证书域名保持一致
#协议用 https
certificate: /data/ssl/harbor.pem
private_key: /data/ssl/harbor.key
邮件和 ldap 不需要配置，在 harbor 的 web 界面可以配置
其他配置采用默认即可
修改之后保存退出
注：harbor 默认的账号密码：admin/Harbor12345
安装 docker-compose
上传课件里的 docker-compose-Linux-x86_64 文件到 harbor 机器
[root@harbor harbor]# mv docker-compose-Linux-x86_64.64 /usr/bin/docker-compose
[root@harbor harbor]# chmod +x /usr/bin/docker-compose
注： docker-compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。
Docker-Compose 的工程配置文件默认为 docker-compose.yml，Docker-Compose 运行目录下的必要有一个
docker-compose.yml。docker-compose 可以管理多个 docker 实例。
安装 harbor 需要的离线镜像包 docker-harbor-2-3-0.tar.gz 在课件，可上传到 harbor 机器，通过
docker load -i 解压
[root@harbor install]# docker load -i docker-harbor-2-3-0.tar.gz 
[root@harbor install]# cd /data/install/harbor
[root@harbor harbor]# ./install.sh
看到下面内容，说明安装成功：
 
[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating registryctl ... done
Creating harbor-db ... done
Creating redis ... done
Creating registry ... done
Creating harbor-portal ... done
Creating harbor-core ... done
Creating harbor-jobservice ... done
Creating nginx ... done
✔ ----Harbor has been installed and started successfully.----
在自己电脑修改 hosts 文件
在 hosts 文件添加如下一行，然后保存即可
192.168.40.181 harbor
扩展：
如何停掉 harbor：
[root@harbor harbor]# cd /data/install/harbor
[root@harbor harbor]# docker-compose stop 
如何启动 harbor：
[root@harbor harbor]# cd /data/install/harbor
[root@harbor harbor]# docker-compose start
如果 docker-compose start 启动 harbor 之后，还是访问不了，那就需要重启虚拟机
18.3 Harbor 图像化界面使用说明 
在浏览器输入：
https://harbor
 
接收风险并继续，出现如下界面，说明访问正常
账号：admin
密码：Harbor12345
输入账号密码出现如下：
所有基础镜像都会放在 library 里面，这是一个公开的镜像仓库
新建项目->起个项目名字 test（把访问级别公开那个选中，让项目才可以被公开使用）
 
18.4 测试使用 harbor 私有镜像仓库 
#修改 docker 配置
[root@ localhost ~]# vim /etc/docker/daemon.json
{ "registry-mirrors": ["https://rsbud4vc.mirror.aliyuncs.com","https://registry.docker￾cn.com","https://docker.mirrors.ustc.edu.cn","https://dockerhub.azk8s.cn","http://hub￾mirror.c.163.com"],
"insecure-registries": ["192.168.40.181","harbor"]
}
修改配置之后使配置生效：
[root@ localhost ~]# systemctl daemon-reload && systemctl restart docker
#查看 docker 是否启动成功
[root@ localhost ~]# systemctl status docker
#显示如下，说明启动成功：
Active: active (running) since Fri … ago
 
注意：
配置新增加了一行内容如下：
"insecure-registries":["192.168.40.181"], 
上面增加的内容表示我们内网访问 harbor 的时候走的是 http，192.168.40.181 是安装 harbor 机器
的 ip
登录 harbor：
[root@ localhost]# docker login 192.168.40.181
Username：admin
Password: Harbor12345
输入账号密码之后看到如下，说明登录成功了：
Login Succeeded
#导入 tomcat 镜像，tomcat.tar.gz 在课件里
[root@ localhost ~]# docker load -i tomcat.tar.gz
#把 tomcat 镜像打标签
[root@ localhost ~]# docker tag tomcat:latest 192.168.40.181/test/tomcat:v1
执行上面命令就会把 192.168.40.181/test/tomcat:v1 上传到 harbor 里的 test 项目下
[root@ localhost ~]# docker push 192.168.40.181/test/tomcat:v1
执行上面命令就会把 192.168.40.181/test/tomcat:v1 上传到 harbor 里的 test 项目下
18.5 从 harbor 仓库下载镜像 
在  localhost 机器上删除镜像
[root@ localhost ~]# docker rmi -f 192.168.40.181/test/tomcat:v1
拉取镜像
[root@ localhost ~]#docker pull 192.168.40.181/test/tomcat:v1
 
 
 
 