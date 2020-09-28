# shiku
视酷即时通讯前后端源码分享，另有酷信全开源新版，更新及二次开发需要扣扣149145308
=========源码构架=======
服务端：
spring、spring-boot		项目基础架构
spring-mvc	Http请求接入
jedis		Redis缓存访问
morphia、mongo-java-driver	Mongodb数据库访问

客户端：
android：android studio 原生开发
ios：object-c xcode 12 原生自绘
pc：vs2019 c#
h5:vscode vue
web:js,htm5
macos:xcode12
Wechat Mini Program：js,wxss







========================================以下是部署文档，需要开发文档或详细部署教程请加扣群524992751===========================
视酷服务端安装部署文档


目录

一、 部署方式一直接安装	3
1) Linux系统下直接安装IM相关服务	3
1、 安装MongoDB	3
2、 安装Redis	4
3、 安装Jdk1.8+	5
4、安装Spring-boot-imapi  api 接口服务	6
5、安装Tigase-Server   xmpp 通讯服务	7
6、 安装 shiku-push 推送服务	10
7、 安装Upload文件上传服务	11
8、 安装FastDFS分布式文件储存系统	14
9、安装Nginx，配置文件访问	23
2) Windows 系统下安装IM 相关服务	25
1、安装MongoDB	25
2、安装Redis	26
3、安装jdk1.8+	27
4、安装Spring-boot-imapi	30
5、安装Tigase	32
6、安装Upload文件上传服务	33
7、配置Nginx文件访问	33
二、部署方式二（通过docker部署）	33
1. 安装docker（以centos 为例）	33
2. 下载、导入镜像	34
3.通过镜像启动容器（container）	35
4.远程连接容器	36
5.修改相关配置（主要是ip）	36
三、服务器维护	40
1.修改调整服务器最大连接数	40
2.查看监听的端口	40
3. 设置防火墙	41
4. 查看日志	42
5. 服务的启动与关闭	43
四．https 配置说明	43
3.Tigase-server 配置https	43
四、创建自定义的docker镜像	45
1.首先确保已经安装 docker	45
2.编写dockerfile 文件	45
3. 通过dockerfile 文件构建镜像	46
4.通过镜像运行容器及相关说明	46
5.在容器里安装相应的服务	46
6. 将相关服务设置为开机自启动	47
1) Mongodb	47
2) Redis	48
3) Tomcat	49
4) Nginx	50
5) Spring-boot-imapi	51
6) Tigase-server	52
7.测试 / 提交容器	53
8.保存/导出最新的镜像	53
五、IM相关服务自启动方案说明：	54
Windows系统：	54
1.找到此压缩包解压：	54
2. 修改路径	54
3. 一键启动	55
4. 实现开机后IM相关服务自行启动	56
Linux 系统：	56
六、端口映射方案说明：	57
1.通过nginx 做端口转发	59
2. 通过(iptables /fireWall)做端口转发	60
1) 开启服务器的ip 转发功能，默认是关闭的	60
2） 配置端口转发	61
说明：将 47.75.89.57 的8088 端口 转至 47.91.255.114 下的 5222 端口	61
3.windows 系统下的端口映射	62
七、数据库加密	63
MongoDB加密说明：	63
Redis 加密说明：	65
八、代码本地导入编译（以Eclipse为例）	66
1. 导入mianshi-parent项目	66
2.编译并运行 mianshi-im-api 项目	71
3.编译并运行tigase 项目	73
4.upload 项目	76





端口说明

服务	用途	使用端口	说明
Mongodb	数据库	28018	
Redis	缓存	6379	
Spring-boot-imapi	Api 接口	8092	
Tigase-server	Xmpp 通讯	Web端使用5280端口，其它端使用5222端口	
Upload 	文件上传	8088	
Nginx	静态文件访问	默认8089 ，根据不同部署情况会有调整	


一、部署方式一直接安装
1)  Linux系统下直接安装IM相关服务

1、安装MongoDB
○1下载并解压缩mongoDb安装包：
[root@shiku~]# cd  /opt
[root@shiku~]# wget  http://47.75.89.57/soft/mongodb-linux-x86_64-3.4.0.tgz
[root@shiku~]# tar  -zxvf  mongodb-linux-x86_64-3.4.0.tgz
[root@shiku~]# mv  mongodb-linux-x86_64-3.4.0  mongodb-3.4.0
○2在/optl/mongodb目录下创建mongo.conf文件内容如下：
[root@shiku~]#cd   mongodb-3.4.0
[root@shiku~]#vim  mongo.conf

旧版配置：
dbpath=/data/mongodb
logpath=/opt/mongodb-3.4.0/logs/mongodb.log
port = 28018
fork = true


新版配置：
systemLog:
   destination: file
   path: "/opt/mongodb-3.4.0/logs/mongodb.log"
   logAppend: true
storage:
   dbPath: "/data/mongodb"
   journal:
      enabled: true
   mmapv1:
     smallFiles: true
   wiredTiger:
      engineConfig:
        configString: cache_size=1G
processManagement:
      fork: true
net:
   #bindIp: 127.0.0.1
   port: 28018
setParameter:
   enableLocalhostAuthBypass: false
security:
   authorization: enabled


○3然后创建mongodb数据目录，和日志目录 
[root@shiku~]# mkdir  -p  /data/mongodb
[root@shiku~]# mkdir  logs 
○4在/opt/mongodb-3.4.0目录下创建start启动脚本内容如下：
 /opt/mongodb-3.4.0/bin/mongod --config=/opt/mongodb-3.4.0/mongo.conf
执行start脚本，出现如下图所示内容则启动成功
 
Stop脚本：
ps -ef|grep mongo.conf|grep -v grep|awk '{printf $2}'|xargs kill -9

（新版本会自动创建）
注意事项：mongodb安装完毕后请将  sql文件夹下mongodb/imapi中的数据导入
Linux：将imapi文件夹拷贝到 /opt 目录下
然后执行以下命令：
cd  /opt/mongodb-3.4.0/bin
./mongo  -port  28018
use imapi
exit
./mongorestore  -h  127.0.0.1:28018 -d imapi  --dir /opt/imapi


2、安装Redis

○1 下载、解压、安装
[root@shiku~]# cd  /opt
[root@shiku~]# wget  http://47.75.89.57/soft/redis-4.0.1.tar.gz
[root@shiku~]# tar  -xvf  redis-4.0.1.tar.gz
[root@shiku~]# cd  redis-4.0.1
[root@shiku~]# make && make  install

○2 修改/opt/redis-4.0.1目录下redis.conf文件中配置项：
daemonize yes（进程后台运行）

○3 在/opt/redis-4.0.1目录下创建start启动脚本内容如下：
/opt/redis-4.0.1/src/redis-server  /opt/redis-4.0.1/redis.conf

○4 执行sh start 命令启动脚本，查看redis是否启动成功
 
Stop脚本：

ps -ef|grep  /opt/redis-4.0.1/src/redis-server|grep -v grep|awk '{printf $2}'|xargs kill -9
ps -ef|grep  /opt/redis-4.0.1/src/redis-server

3、安装Jdk1.8+
[root@shiku~]# cd  /opt
[root@shiku~]# wget  http://47.75.89.57/soft/jdk-8u131-linux-x64.tar.gz
[root@shiku~]# tar -zxvf  jdk-8u131-linux-x64.tar.gz
[root@shiku~]# mkdir  java
[root@shiku~]# mv  jdk1.8.0_131  ./java

设置jdk环境变量
这里采用全局设置方法，就是修改etc/profile，它是是所有用户的共用的环境变量
[root@shiku~]# vim  /etc/profile 
打开之后在末尾添加
JAVA_HOME=/opt/java/jdk1.8.0_131
JRE_HOME=/opt/java/jdk1.8.0_131/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export PATH JAVA_HOME CLASSPATH
使环境变量生效
[root@shiku~]# source /etc/profile
○5 检验是否安装成功
在终端执行：  java  -version 命令，看看是否安装成功，成功则显示如下
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)

 

○6可能出现的问题：
若出现 bash: /usr/bin/java: /lib/ld-linux.so.2: bad ELF interpreter: No such file or directory
执行：sudo yum install glibc.i686 命令安装glibc


4、安装Spring-boot-imapi  api 接口服务
[root@shiku~]# cd  /opt
[root@shiku~]# wget  http://47.75.89.57/soft/spring-boot-imapi.tar
[root@shiku~]# tar  -xvf  spring-boot-imapi.tar
[root@shiku~]# cd  spring-boot-imapi
[root@shiku~]# vim  application.properties

2)修改application.properties配置文件

 


Apikey=13824309221p

Mongoconfig.mappackage=cn.xyz.mianshi.vo
 














	Redis链接地址可能有问题
redis：//192.168.0.168：6379
 
3) 在spring-boot-imapi目录下执行sh start命令运行imapi接口服务
 

4) 修改后台配置

启动完成后，在浏览器打开链接“http://localhost:8092/console/login”,出现如下图所示内容即酷聊接口部署成功：
 
注：超级管理员账号：1000  初始密码：1000

将系统配置---> 客户端配置里的地址修改为部署的服务器的地址，这些地址用于在客户端启动时返回给客户端使用，请参考下图
 



5、安装Tigase-Server   xmpp 通讯服务
[root@shiku~]# cd  /opt
[root@shiku~]# wget  http://47.75.89.57/soft/tigase-server-7.1.3-b4482.tar
[root@shiku~]# tar  -xvf  tigase-server-7.0.1.tar
[root@shiku~]# cd  tigase-server-7.0.1
[root@shiku~]# vim  etc/ tigase.conf
修改tigase.conf配置文件中JAVA_HOME参数为本机JDK安装目录
 
修改 init.properties配置文件：
[root@shiku~]#vim /etc/init.prperties

配置项修改说明：
--admins=admin@im.shiku.co,10005@im.shiku.co（管理员列表）
--virt-hosts=im.shiku.co
请参照如下标注进行修改配置
 
 


○4 在tigase-server-7.0.1目录下执行 sh start 命令启动tigase、查看tigase是否启动成功

附：  tigase 客户端Spark（用于测试）下载地址：
http://www.igniterealtime.org/downloads/index.jsp

 

注意事项：
○1 使用spark注册账号时，用户名请填写数字，如：100011
○2 Spark是一个xmpp客户端，可以用来测试；App可以发消息给Spark，但APP解析不了Spark发来的内容，除非符合协议标准。

6、安装 shiku-push 推送服务
整体说明：tigase-server 服务为生产者，rocketmq  为消息队列，shiku-push 服务为消费者
生产者产生需要推送的消息放入消息队列中缓存起来，消费者从队列中读取需要推送的消息
然后去调用Apns（IOS）、小米、华为（Android）等平台的推送将消息推送给离线用户。

1、首先安装rocketmq  消息队列
下载地址：http://mirror.bit.edu.cn/apache/rocketmq/4.3.2/rocketmq-all-4.3.2-bin-release.zip
新http://mirror.bit.edu.cn/apache/rocketmq/4.6.1/rocketmq-all-4.6.1-bin-release.zip


wget  http://mirror.bit.edu.cn/apache/rocketmq/4.3.2/rocketmq-all-4.3.2-bin-release.zip

unzip  rocketmq-all-4.3.2-bin-release.zip

cd rocketmq-all-4.3.2-bin-release

调整rocketMq 的内存值

注意：这里请根据机器的实际情况进行调整，如机器内存较大可适当配置高一点

vim  bin/runbroker.sh

 

vim bin/runserver.sh

 

vim startSrv

写入如下内容：
nohup sh ./bin/mqnamesrv  >  ./logs/rocketmqlogs/namesrv.log  &
tail  -f  ./logs/rocketmqlogs/namesrv.log

执行startSrv 脚本启动nameServer
sh  startSrv  
 

vim startBroker
写入如下内容：
nohup sh bin/mqbroker -n  localhost:9876  >  ./logs/rocketmqlogs/broker.log   &
tail  -f  ./logs/rocketmqlogs/broker.log

执行startBroker脚本启动borker
sh  startBroker
 

执行jps 命令 查看正常应该能看到NamesrvStaup 和 BrokerStartup
 

注册推送消息、用户状态话题

sh  bin/mqadmin updateTopic -n localhost:9876  -c DefaultCluster  -t  pushMessage
sh  bin/mqadmin updateTopic -n localhost:9876  -c DefaultCluster  -t  xmppMessage
sh  bin/mqadmin updateTopic -n localhost:9876  -c DefaultCluster  -t  userStatusMessage

集群：命令
mqadmin deleteTopic -n localhost:9876 -c rmq-cluster -t pushMessage
 

附：停止命令

sh bin/mqshutdown namesrv

sh bin/mqshutdown broker


2、安装shiku-push 推送服务

部署包下载地址：wget  http://47.75.89.57/soft/shiku-push.tar

 
1.首先在部署文件目录中找到linux 版本的shiku-push部署文件上传至服务器
      
shiku-push 部署包各个文件说明
 

2.替换ios   apns推送证书，个人版和企业版选择一个使用即可，需要注意在申请ios  apns 证书的时候必须要设置密码。

3.修改application.properties 配置文件
将申请好的小米、华为、极光 等平台的key   secret 配置到对应的位置
 

4.	修改完配置后使用 sh start 命令启动shiku-push服务



7、安装message-push服务
[root@shiku~]# cd  /opt
[root@shiku~]# wget  http://47.75.89.57/soft/message-push.tar
[root@shiku~]# tar -xvf message-push.tar
[root@shiku~]# cd  message-push
[root@shiku~]# vim  application.properties

参照如下示例，修改配置文件
 
启动 message-push 服务
修改start里面的配置文件路径，不然跑不起来。
[root@shiku~]#sh start
8、安装Upload文件上传服务
[root@shiku~]# cd  /opt
[root@shiku~]# wget  http://47.75.89.57/soft/upload.tar
[root@shiku~]# tar -xvf upload.tar
[root@shiku~]# cd  upload
[root@shiku~]# vim  application.properties


2）参照如下示例修改配置：
说明：/data/www/resources：文件上传成功后存储的根目录
beginIndex：根目录“/data/www/resources”的字符串长度（从0开始）用于分隔字符串生成文件链接用（需要注意，如果修改了相关路径，beginIndex 的数值也要相应修改）

 
 





3）在文件上传服务所在机器创建存储目录（例如“/data/www/resources”）并初始化目录结构
可以直接将以下命令全部拷贝到Linux服务器一次性执行
 
mkdir  -p  /data/www/resources
cd /data/www/resources
mkdir audio
mkdir avatar
mkdir avatar/o
mkdir avatar/t
mkdir avatar_r
mkdir avatar_r/o
mkdir avatar_r/t
mkdir gift
mkdir image
mkdir image/o
mkdir image/t
mkdir other
mkdir preview
mkdir temp
mkdir u
mkdir video

4） 启动 upload 

[root@shiku~]# Cd  /opt/upload
[root@shiku~]# sh  start
 
9、安装FastDFS分布式文件储存系统

   注：FastDFS 主要用于在多个节点间上传文件，同时会将文件同步到各个节点（如搭建单个节点的测试环境，无需安装FastDFS）
      多个节点上均按如下步骤进行安装
以下步骤会使用到 unzip，请确保已经安装了unzip，
如没有安装使用如下命令安装unzip：
 [root@shiku~]# yum install -y unzip 
8.1下载安装 libfastcommon 
[root@shiku~]# cd  /opt 
[root@shiku~]# wget  http://47.75.89.57/soft/libfastcommon-master.zip
[root@shiku~]# unzip libfastcommon-master.zip
[root@shiku~]#cd  libfastcommon-master
[root@shiku~]# ./make.sh
[root@shiku~]# ./make.sh install



显示这样的画面，说明 libfastcommon 安装成功

 

8.2 下载安装 FastDFS
[root@shiku~]# cd  /opt 
[root@shiku~]# wget  http://47.75.89.57/soft/fastdfs-master.zip
[root@shiku~]# unzip fastdfs-master.zip
[root@shiku~]# cd  fastdfs-master
[root@shiku~]# ./make.sh
[root@shiku~]# ./make.sh  install

显示这样的画面，说明安装 FastDFS 成功。
 


注：可能遇到的问题
 

错误原因
libfastcommon 在这个/usr/local/include 目录 而不是 
/usr/include 目录
解决办法
把libfastcommon   复制到 /usr/include/目录下
cp -r /usr/local/include/fastcommon  /usr/include/

8.3配置 Tracker 服务
上述安装成功后，在/etc/目录下会生成一个fdfs的目录。
.sample后缀的文件，是示例配置文件
[root@shiku~]# cd  /etc/fdfs 
[root@shiku~]# mv  tracker.conf.sample  tracker.conf
[root@shiku~]# vim tracker.conf

打开tracker.conf文件，只需要找到你只需要该这两个参数就可以了。

# the base path to store data and log files
base_path=/data/fastdfs 

# HTTP port on this tracker server
http.server_port=8080 

修改完成后保存退出

[root@shiku~]# mkdir  -p  /data/fastdfs  #创建目录

[root@shiku~]#/usr/bin/fdfs_trackerd  /etc/fdfs/tracker.conf  start  #启动tracker

注：如果觉得这个启动命令不够优雅，可以使用ln -s 建立软链接：
[root@shiku~]# ln -s /usr/bin/fdfs_trackerd /usr/local/bin
[root@shiku~]# ln -s /usr/bin/stop.sh /usr/local/bin
[root@shiku~]# ln -s /usr/bin/restart.sh /usr/local/bin
 
这时候我们就可以使用service  fdfs_trackerd  start来优雅地启动 Tracker服务了。
你也可以启动过服务看一下端口是否在监听，命令：

启动服务：service fdfs_trackerd start 
查看监听：netstat -unltp|grep fdfs  

 

看到22122端口正常被监听后，说明 Tracker服务启动成功
8.4 配置 Storage 服务
现在开始配置 Storage 服务，它有 Group(组)的概念，同一组内服务器互备同步。
[root@shiku~]# cd  /etc/fdfs
[root@shiku~]# mv  storage.conf.sample  storage.conf
[root@shiku~]# vim  storage.conf
 


对下图标注的几个地方进行修改：
 
 
 

stroage的port=23000这个端口参数也不建议修改，默认就好，除非你已经占用它了。
修改完成保存并退出 vim 

[root@shiku~]# /usr/bin/fdfs_storaged  /etc/fdfs/storage.conf start  #启动storaged

这里还是使用ln -s建立软链接：

[root@shiku~]#ln -s /usr/bin/fdfs_storaged /usr/local/bin
 
建立软连接后就可以使用如下命令启动storaged服务了：

[root@shiku~]# service fdfs_storaged start


 

查看一下端口监听，正常的话22122 和 23000端口都处于监听状态
[root@shiku~]# netstat -unltp|grep fdfs
 

我们安装配置并启动了 Tracker 和 Storage 服务，也没有报错了。我们可以使用如下命令监视一下， Tracker 和 Storage 服务是不是在正常通信。

[root@shiku~]# /usr/bin/fdfs_monitor  /etc/fdfs/storage.conf


8.5 安装 Nginx 和 fastdfs-nginx-module
首先安装 nginx 所需的依赖  
 [root@shiku~]# yum install -y  pcre  pcre-devel  zlib  zlib-devel openssl  openssl-devel  

下载解压 fastdfs-nginx-module 
[root@shiku~]# cd /opt
[root@shiku~]# wget  http://47.75.89.57/soft/fastdfs-nginx-module-master.zip
[root@shiku~]# unzip  fastdfs-nginx-module-master.zip

安装 Nginx-1.9.11
[root@shiku~]# cd /opt
[root@shiku~]# wget  http://47.75.89.57/soft/nginx-1.9.11.tar.gz
[root@shiku~]# tar  -zxvf  nginx-1.9.11.tar.gz
[root@shiku~]# mv  nginx-1.9.11  nginx-install
[root@shiku~]# cd nginx-install 
[root@shiku~]#./configure--prefix=/opt/nginx-1.9.11--add-module=/opt/fastdfs-nginx-module-master/src  --with-http_ssl_module  --with-stream
[root@shiku~]# make && make install 
[root@shiku~]# rm  -rf  ../nginx-install
在/opt/nginx-1.9.11目录下创建start、stop脚本：
start脚本： vim  start ，写入如下内容
./sbin/nginx
ps -ef|grep nginx

stop脚本： vim  stop 写入如下内容 
./sbin/nginx -s stop
ps -ef|grep nginx


8.6配置 fastdfs-nginx-module 和 Nginx

配置mod-fastdfs.conf，并拷贝到/etc/fdfs文件目录下

[root@shiku~]# cd  /opt/fastdfs-nginx-module-master/src/
[root@shiku~]# vim  mod_fastdfs.conf
[root@shiku~]# cp  mod_fastdfs.conf  /etc/fdfs 

注： 修改mod-fastdfs.conf配置只需要修改标注的这几个地方就行了，其他不需要也不建议改变。
tracker_server  必须是内网ip，可以配置多个，根据节点部署情况进行配置，
 
 

接着我们需要把fastdfs-master 中的一些文件中拷贝到 /etc/fdfs 目录中

[root@shiku~]# cd  /opt/fastdfs-master/conf/
[root@shiku~]# cp  anti-steal.jpg  http.conf  mime.types  /etc/fdfs/ 


配置 Nginx 编辑nginx.conf文件：
[root@shiku~]# cd /opt/nginx-1.9.11/conf
[root@shiku~]# vim  nginx.conf 

在配置文件中加入：
#FastDFS 图片、音频、视频、等文件访问
    server{
       listen            8089;
       server_name       localhost;
       location ~* /\.(html|htm|jsp|php|js)$ {
          deny  all;
       }
       location /group1/M00{
           root /data/fastdfs;
           ngx_fastdfs_module;
       }
}
由于我们配置了group1/M00的访问，我们需要建立一个group1文件夹，并建立M00到data的软链接。

[root@shiku~]# mkdir  /data/fastdfs/group1
[root@shiku~]# ln  -s  /data/fastdfs/data   /data/fastdfs/group1/M00
 


最后在作为负载均衡入口的机器上配置多个节点负载均衡访问文件：

说明：只需要在作为入口的机器上配置即可
修改 nginx 的配置文件 nginx.conf 加入以下内容：

将如下配置加到 http{} 内

#配置多个节点负载均衡访问文件
upstream  file{
      server   172.31.77.82:8089;
      server   172.31.77.83:8089;
    }

    server{
       listen  8090;
       server_name  43.97.25.94:8090;

       location / {
          proxy_pass        http://file;
          #proxy_set_header   Host             $host;
          #proxy_set_header   X-Real-IP        $remote_addr;
          #proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
       }
    }



10、安装Nginx，配置文件访问
1） 首先请确保已经安装 Nginx，如没有安装请参考上面的 “安装 Nginx-1.9.11
”中的步骤进行安装。

2）在upload 所在的机器上 配置头像访问
说明：由于FastDfs 不支持对上传文件的自定义命名，而目前用户头像是根据userId 命名的，所以目前用户头像没有使用FastDfs 储存。

修改Nginx 配置文件 nginx.conf ，添加如下内容（域名配置示例）
#用户头像访问
server{
    listen            80;
    server_name       head.shiku.co;
#拒接访问html 等类型的文件避免受到脚本攻击
location ~* /\.(html|htm|jsp|php|js)$ {
          deny  all;
    }

    location /{
            root        /data/www/resources;
            expires     4d;
    }
}



（ ip 配置示例 ）
    server{
       listen             8088;
       server_name       localhost;
        #拒接访问html 等类型的文件避免受到脚本攻击
        location ~ /\.(html|htm|jsp) {
                deny  all;
        }

        location ~* /{
           root    /data/www/resources;
           expires      4d;
        }
     }

3) 执行start、stop脚本，查看nginx是否启动、停止成功：
 

4）可能遇到的问题
 1.若出现以下问题，说明软连接没有设置好：
 
解决方法：
cd  /lib64
ls *pcre*
ln -s  /usr/local/lib/libpcre.so.1  /lib64/
参考：http://blog.csdn.net/ystyaoshengting/article/details/50504746

11、将相关服务设置为开机自启动
1)  Mongodb
在/lib/systemd/system/目录下新建mongodb.service文件，内容如下：                 
[Unit]
Description=mongodb
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/opt/mongodb-3.2.4/bin/mongod --config /opt/mongodb-3.2.4/mongo.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/opt/mongodb-3.2.4/bin/mongod  --shutdown  --config /opt/mongodb-3.2.4/mongo.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
如图所示：
 
#设置权限
[root@shiku~]# chmod 754 mongodb.service

#启动服务  
[root@shiku~]#systemctl start mongodb.service    
#关闭服务    
[root@shiku~]#systemctl stop mongodb.service    
#开机启动    
[root@shiku~]#systemctl enable mongodb.service  
2)  Redis
在/lib/systemd/system/目录下新建redis.service文件，内容如下：
[Unit]
Description=redis
After=syslog.target network.target

[Service]
Type=forking
PIDFile=/var/run/redis_6388.pid
ExecStart=/opt/redis-4.0.1/src/redis-server  /opt/redis-4.0.1/redis.conf
ExecReload=/bin/kill -USR2 $MAINPID
ExecStop=/bin/kill -SIGINT $MAINPID

[Install]
WantedBy=multi-user.target

如下图所示：
 

#设置权限
[root@shiku~]# chmod 754 redis.service

#启动服务  
[root@shiku~]#systemctl  start  redis.service    
#关闭服务    
[root@shiku~]#systemctl  stop  redis.service    
#开机启动    
[root@shiku~]#systemctl  enable  redis.service  

3)  Tomcat
在/lib/systemd/system/目录下新建tomcat.service文件，内容如下：
[Unit]
Description=Tomcat
After=syslog.targetnetwork.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/opt/apache-tomcat-8.0.48/tomcat.pid
ExecStart=/opt/apache-tomcat-8.0.48/bin/startup.sh
ExecReload=/bin/kill-s HUP $MAINPID
ExecStop=/bin/kill-s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target

如下图所示：
 
#设置权限
[root@shiku~]# chmod 754 tomcat.service

#启动服务  
[root@shiku~]#systemctl  start  tomcat.service    
#关闭服务    
[root@shiku~]#systemctl  stop  tomcat.service    
#开机启动    
[root@shiku~]#systemctl  enable  tomcat.service  


4)  Nginx
在/lib/systemd/system/目录下新建 nginx.service文件，内容如下：
[Unit]
Description=nginx
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/nginx_1.7.9/sbin/nginx
ExecReload=/opt/nginx_1.7.9/sbin/nginx -s reload
ExecStop=/opt/nginx_1.7.9/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target

如下图所示：
 

#设置权限
[root@shiku~]# chmod 754 nginx.service

#启动服务  
[root@shiku~]#systemctl  start  nginx.service    
#关闭服务    
[root@shiku~]#systemctl  stop  nginx.service    
#开机启动    
[root@shiku~]#systemctl  enable  nginx.service  

5)  Spring-boot-imapi
在/lib/systemd/system/目录下新建 imapi.service文件，内容如下：
[Unit]
Description=imapi
After=syslog.target network.target remote-fs.target nss-lookup.target mongodb.service  redis.service

[Service]
Type=simple
PIDFile=/opt/spring-boot-imapi/imapi.pid
ExecStart=/opt/spring-boot-imapi/startup.sh
ExecReload=/bin/kill-s HUP $MAINPID
ExecStop=/opt/spring-boot-imapi/stop
PrivateTmp=true
#SuccessExitStatus=143

[Install]
WantedBy=multi-user.target

如下图所示：
 
#设置权限
[root@shiku~]# chmod 754 imapi.service 

#启动服务  
[root@shiku~]#systemctl  start  imapi.service    
#关闭服务    
[root@shiku~]#systemctl  stop  imapi.service    
#开机启动    
[root@shiku~]#systemctl  enable  imapi.service  

6)  Tigase-server
在/lib/systemd/system/目录下新建 tigase.service文件，内容如下：
[Unit]
Description=tigase
After=syslog.target network.target remote-fs.target nss-lookup.target mongodb.service  redis.service

[Service]
Type=forking
ExecStart=/opt/tigase-server-7.0.1/scripts/tigase.sh start  /opt/tigase-server-7.0.1/etc/tigase.conf
ExecReload=/bin/kill-s HUP $MAINPID
ExecStop=/opt/tigase-server-7.0.1/scripts/tigase.sh stop /opt/tigase-server-7.0.1/etc/tigase.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target


如下图所示：
 
#设置权限
[root@shiku~]# chmod  754  tigase.service  

#启动服务  
[root@shiku~]# systemctl  start  tigase.service    
#关闭服务    
[root@shiku~]# systemctl  stop  tigase.service    
#开机启动    
[root@shiku~]# systemctl  enable  tigase.service  

2)  Windows 系统下安装IM 相关服务
1、安装MongoDB
○1 下载Windowns版MongoDB 	https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-3.0.7.zip
○2 解压mongodb-win32-x86_64-3.0.7.zip并打开mongodb-win32-x86_64-3.0.7目录
○3 将mongodb-win32-x86_64-3.0.7目录重命名为mongodb，并在该目录下创建 data/db目录
○4 在mongodb/bin目录下按住Shift键，右键打开菜单，点击在此处打开命令窗口 输入如下命令启动mongodb：
mongod --dbpath E:\IM\mongodb\data\db
○5 出现以下内容表示启动成功
 

2、安装Redis
○1 下载Windows版本Redis
https://github.com/MSOpenTech/redis/releases/download/win-2.8.2104/Redis-x64-2.8.2104.zip

○2 解压Redis-x64-2.8.2104.zip并打开Redis-x64-2.8.2104目录，在目录按住Shift键，右键
打开菜单，点击在此处打开命令窗口 输入如下命令启动redis：
redis-server.exe  redis.windows.conf
或者redis-server.exe  redis.windows.conf    --maxheap 200m


○3 出现如下图所示的内容说明redis启动成功
 


3、安装jdk1.8+
○1 首先到官网下载 jdk-8u144-windows-x64.exe
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

○2下载完成后点击进行安装
 

○3配置环境变量


 
 


在新弹出窗口上，点系统变量区域下面的新建按钮，弹出新建窗口，变量名为JAVA_HOME，变量值填JDK安装的最终路径，我这里装的地址是D:\Program Files\Java\jdk1.7.0_51，所以填D:\Program Files\Java\jdk1.7.0_51，点确定完成。

 


下面需要设置Path变量，由于系统本身已经存在这个变量，所以无需新建，在原本基本上添加JDK相关的，找到Path变量双击编辑，由于每个值之间用;符号间断，所以先在末尾加上;（注意是英文格式的，不要输其他符号空格等），加上;符号后在末尾加入%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin，点确定完成。
 

○4验证安装是否成功
打开windows 命令窗口 输入 java -version   显示如下内容这说明安装成功。
 

4、安装Spring-boot-imapi 
○1 打开酷聊服务端-Windows-Final 目录下的spring-boot-imapi文件夹

                       

○2 参照Linux安装步骤中的第○3歩修改 application.properties配置文件
 

○3 右键执行start.bat脚本，即可启动imapi
 


○4 执行脚本后会出现如下窗口，表示启动成功
 
5、安装Tigase
○1 打开酷聊服务端-Windows-Final 目录下的tigase-server-7.0.1-b3810-win32 文件夹

                           

○2 参照Linux安装步骤中的第○3歩修改 init.properties
○3 右键执行Run.bat脚本，即可启动tigase
 
○4 执行脚本后会出现如下窗口，看到 红框中的内容表示启动成功，需要注意此窗口必须要保持打开状态，
将其关闭后，tigase也就关闭了。

 
 




6、安装Upload文件上传服务
请参看 linux 下 Upload 安装步骤

7、配置Nginx文件访问
打开http://nginx.org/download/nginx-1.9.11.zip 下载并解压，按照Linux安装步骤下“修改配置文件（nginx-1.9.11/conf/nginx.conf）”步骤修改配置，然后双击nginx-1.9.6目录下nginx.exe启动nginx

8、Windows下将服务自启动说明：


1.找到此压缩包解压：


解压后会有一个IMStart_BAT 文件夹,和一个IMAllStart脚本，
其中IMAllStart.bat脚本是总启动脚本，执行后会将所有的IM 
相关服务启动，IMStart_BAT文件夹下存放的是IM的各个分程序的启动脚本，如下图：

 
2.修改路径
选中上图的脚本，右键 ——》编辑，根据自己的安装位置修改脚本文件中的盘符路径

○1 编辑FreeSwitchStart.bat 脚本，将红线框出的部分修改为自己的FreeSWITCH安装路径
 
○2编辑mongoStart.bat 脚本，将红线框出的部分修改为自己的mongoDB安装路径
 
注：这里的mongodb目录下的  data\db 目录是自己创建的，安装步骤中有相应说明。
○3 编辑nginxStart.bat 脚本，将红线框出的部分修改为自己的Nginx安装路径
 
○4 编辑redisStart.bat 脚本，将红线框出的部分修改为自己的Redis安装路径
 
○5 编辑spring.bat 脚本，将红线框出的部分修改为自己的Spring-boot-imapi项目的解压路径
 
○6 编辑tigase.bat 脚本，将红线框出的部分修改为自己的tigase-server项目的解压路径
 
○7 编辑tomcat.bat 脚本，将红线框出的部分修改为自己的Tomcat安装路径
 
3.一键启动
在将IMStart_BAT 文件夹中所有脚本的路径都修改、保存好之后将IMStart_BAT 文件夹拷贝到E盘，然后双击执行IMAllStart.bat脚本，即可实现IM相关服务一键启动。




4.实现开机后IM相关服务自行启动
来到Windows开机启动项目录：
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs
将MAllStart.bat脚本拷贝到该目录中即可实现开机自启动。



二、部署方式二（通过docker部署）
注：如习惯使用Docker 容器部署项目，并具有相应的使用、维护经验，则可以使用Docker 容器的方式部署。
1.	安装docker（以centos7.X 为例）
注：Docker 要求 CentOS 系统的内核版本高于 3.10 ，通过 uname -r 命令查看你当前的内核版本验证你的CentOS 版本是否支持 Docker 。
[root@shiku ~]# uname  -r 
 

安装 docker 服务
[root@shiku ~]#  yum -y install docker-io 
 

启动docker 服务
[root@shiku ~]#service docker start 

 

2.	Docker 创建Redis 容器
1、从官方仓库拉取redis 4.0.1 的镜像
[root@shiku ~]# docker pull redis:4.0.1 
 

2、在宿主机创建 /data/redis 目录
mkdir  /data/redis

3、通过镜像启动容器
docker run -p 6388:6379 --name redis-4.0.1  --hostname redis  --restart=always -v /data/redis:/data  -d redis:4.0.1  redis-server --appendonly yes

 


3.	Docker 创建Mongodb容器
1、使用命令从官方仓库拉取3.4.0 版本的镜像
[root@shiku ~]# docker  pull  mongo:3.4.0

2、在宿主机创建目录
mkdir  -p  /data/mongodb/db

3、使用镜像启动容器
docker run -d -p 28018:27017  -v  /data/mongodb/db:/data/db  --restart=always --name mongodb  mongo:3.4.0  


注：IP地址请根据实际部署机器的内网ip进行调整
4.	Docker 创建imapi-server容器
1、在imapi 的部署文件spring-boot-imapi中创建 imapi_dockerfile文件 ，写入如下内容
FROM java:8
VOLUME  /opt/logs
COPY application.properties /opt/application.properties
ADD mianshi-im-api-0.0.1-SNAPSHOT.war /opt/mianshi-im-api-0.0.1-SNAPSHOT.war
RUN bash -c 'touch /opt/mianshi-im-api-0.0.1-SNAPSHOT.war'
EXPOSE 8092
ENTRYPOINT ["java","-jar","/opt/mianshi-im-api-0.0.1-SNAPSHOT.war","--spring.config.location=/opt/application.properties"]

说明：这里的application.properties 从 conf_file/imapi 目录里拷贝，conf_file 下载地址：
http://47.75.89.57/soft/conf_file.tar

mianshi-im-api-0.0.1-SNAPSHOT.war 为imapi 服务的部署包

2、通过dockerfile 文件构建镜像
docker build  -t  imapi  -f  imapi_dockerfile  ./

 

3、通过镜像启动容器
docker run -d -p 8092:8092 -e "UPLOAD_ADDR=192.168.0.152:8088" -e "MONGODB_ADDR=192.168.0.151:28018"    -e "NAMESRV_ADDR=192.168.0.155:9876"  -e "XMPP_HOST=192.168.0.155" -e "XMPP_SERVERNAME=im.shiku.co" -e "REDIS_ADDR=redis://192.168.0.155:6388" --name  imapi-server  imapi

5.	Docker 创建Tigase-server容器
1、修改  tigase-server-7.1.3-b4482/etc/tigase.conf  配置文件
 
修改 JAVA_HOME的值为"${JAVA_HOME}"，修改TIGASE_HOME的值为"/opt/tigase-server"
JAVA_HOME="${JAVA_HOME}"
TIGASE_HOME="/opt/tigase-server"


2、修改 tigase-server-7.1.3-b4482/etc/init.properties 配置文件zhanghaomima
admin@im.qishiyong.com 10005@im.qishiyong.cn
 
  
3、编辑tigase-dockerfile 文件，写入如下内容：
（说明：tigase-server-7.1.3-b4482 为tigase-server 的部署文件的目录 ）

注：5222 端口用于Android、Ios、PC 等客户端连接使用，5280 端口用于WEB 版连接使用，如没有 WEB 版5280 端口可不用

FROM java:8
VOLUME  /opt/logs
ADD ./tigase-server-7.1.3-b4482/  /opt/tigase-server
RUN chmod  -R 755 /opt
EXPOSE 5222
EXPOSE 5280
ENTRYPOINT cd /opt/tigase-server; java -version; ./scripts/tigase.sh run etc/tigase.conf; wait $!

4、通过dockerfile 文件构建镜像
docker  build  -t  tigase-server -f  tigase-dockerfile ./

5、通过镜像启动容器
docker run -d -p 5222:5222  -p 5280:5280  --name  tigase-server  tigase-server

6.	Docker 创建RocketMq容器
1、拉取rocketmq-4.3.2 的镜像
docker pull rocketmqinc/rocketmq:4.3.2
 
2、在宿主机创建目录
mkdir  -p  /data/namesrv/logs
mkdir  -p  /data/namesrv/store
mkdir  -p  /data/broker/logs
mkdir  -p  /data/broker/store

3、使用镜像启动 NameServer 容器

docker run -d -p 9876:9876 -v /data/namesrv/logs:/root/logs -v  /data/namesrv/store:/root/store --name rmqnamesrv  rocketmqinc/rocketmq:4.3.2 sh mqnamesrv


4、使用镜像启动broker 容器（说明：-Xms512m 等内存值可根据机器实际内存值进行调整）


docker run -d -p 10911:10911 -p 10909:10909 -v /data/broker/logs:/root/logs -v /data/broker/store:/root/store --name rmqbroker --link rmqnamesrv:namesrv -e "NAMESRV_ADDR=namesrv:9876"  -e "JAVA_OPT=${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:PermSize=128m -XX:MaxPermSize=128m" rocketmqinc/rocketmq:4.3.2  sh  mqbroker


7.	Docker 创建Shiku-push容器
1、在shiku-push的部署文件shiku-push中创建 shiku-push_dockerfile文件 ，写入如下内容

FROM java:8
VOLUME  /opt/shiku-push/logs
COPY application.properties /opt/shiku-push/application.properties
COPY *.p12  /opt/shiku-push/
COPY sixth-hawk-164509-firebase-adminsdk-342gn-465351f0ef.json /opt/shiku-push/sixth-hawk-164509-firebase-adminsdk-342gn-465351f0ef.json
ADD shiku-push-1.0.war /opt/shiku-push/shiku-push-1.0.war
RUN bash -c 'touch /opt/shiku-push/shiku-push-1.0.war'
ENTRYPOINT ["java","-jar","/opt/shiku-push/shiku-push-1.0.war","--spring.config.location=/opt/shiku-push/application.properties"]

说明：这里的application.properties 从 conf_file/shiku-push目录里拷贝
.p12 文件为ios 推送证书文件
sixth-hawk-164509-firebase-adminsdk-342gn-465351f0ef.json  为Google 推送需要的文件
shiku-push-1.0.war 为 shiku-push 服务部署包

2、通过dockerfile 文件构建镜像
docker build -t shiku-push  -f shiku-push_dockerfile  ./

3、通过镜像启动容器
docker run -d -e "MONGODB_ADDR=192.168.0.151:28018"    -e "NAMESRV_ADDR=192.168.0.155:9876"  -e "XMPP_HOST=192.168.0.155" -e "XMPP_SERVERNAME=im.shiku.co" -e "REDIS_ADDR=redis://192.168.0.155:6379" --name  shiku-push-server  shiku-push


8.Docker 创建message-push 容器
1、在message-push的部署文件shiku-push中创建 message-push_dockerfile文件 ，写入如下内容

FROM java:8
VOLUME  /opt/message-push/logs
COPY application.properties /opt/message-push/application.properties
ADD  xmpp-push-0.0.1-SNAPSHOT.war /opt/message-push/xmpp-push-0.0.1-SNAPSHOT.war
RUN bash -c 'touch /opt/message-push/xmpp-push-0.0.1-SNAPSHOT.war'
ENTRYPOINT ["java","-jar","/opt/message-push/xmpp-push-0.0.1-SNAPSHOT.war","--spring.config.location=/opt/message-push/application.properties"]

说明：这里的application.properties 从 conf_file/message-push目录里拷贝
xmpp-push-0.0.1-SNAPSHOT.war 为 message-push 服务的部署包


2、通过dockerfile 文件构建镜像
docker  build  -t  message-push  -f  message-push_dockerfile  ./

3、通过镜像启动容器
docker run -d -e "MONGODB_ADDR=192.168.0.151:28018"    -e "NAMESRV_ADDR=192.168.0.151:9876"  -e "XMPP_HOST=192.168.0.151" -e "XMPP_SERVERNAME=im.shiku.co" -e "REDIS_ADDR=redis://192.168.0.151:6379" --name  message-push-server  message-push


9.Docker 创建Upload容器

1、在upload的部署文件upload中创建 upload_dockerfile文件 ，写入如下内容

FROM java:8
VOLUME  /opt/upload/logs
COPY application.properties /opt/upload/application.properties
ADD  upload-2.0.war /opt/upload/upload-2.0.war
RUN bash -c 'touch /opt/upload/upload-2.0.war'
RUN mkdir  -p  /data/www/resources
ENTRYPOINT ["java","-jar","/opt/upload/upload-2.0.war","--spring.config.location=/opt/upload/application.properties"]

说明：这里的application.properties 从 conf_file/upload目录里拷贝
upload-2.0.war 为 upload 服务的部署包

2、通过dockerfile 文件构建镜像
docker build -t upload  -f upload_dockerfile  ./

3、在宿主机创建文件存储目录，挂载到容器(可将下面命令复制后一次执行)

mkdir  -p  /data/www/resources
cd  /data/www/resources 
mkdir audio
mkdir avatar
mkdir avatar/o
mkdir avatar/t
mkdir avatar_r
mkdir avatar_r/o
mkdir avatar_r/t
mkdir gift
mkdir image
mkdir image/o
mkdir image/t
mkdir other
mkdir preview
mkdir temp
mkdir u
mkdir video

 
4、通过镜像启动容器
docker run -d  -p 8088:8088 -v /data/www/resources:/data/www/resources -e "FILE_URL=http://file.shiku.co" -e "MONGODB_URL=mongodb://192.168.0.151:28018" --name upload-server  upload

10.	Docker 创建Nginx容器
说明：这里Nginx 主要提供文件访问服务，需要和 Upload 放在同一宿主机，以便能读取到上传的文件

1、拉取官方镜像
docker  pull  nginx

2、在宿主机创建目录，编辑配置文件
mkdir  -p  /data/nginx/logs
mkdir  -p  /data/nginx/conf
cd   /data/nginx/conf

vim nginx.conf

写入如下内容：
user  root;
worker_processes  1;

pid        /run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    #文件访问
server{
   listen        8089;  
   charset utf-8;
   location / {
       if ($http_referer ~* ".php") {
             return 403;
        }
        root     /www/resources;
        expires  5d;
   }
}

}

3、通过镜像启动容器
docker run -p 8089:8089 --name nginx -v /data/www/resources:/www/resources -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /data/nginx/logs:/var/log/nginx   -d nginx




三、配置第三方短信服务 
说明：项目目前集成了阿里云短信和天天国际短信

四、配置消息离线推送	







五、配置微信充值与提现
1.配置微信充值
注意：首先确保拥有微信开放平台的账号，没有的话请注册一个
微信开放平台网址：https://open.weixin.qq.com/

1 ) 登录微信开放平台，点击管理中心，然后点击创建移动应用按照提示填写应用资料，
提交审核。
        
 

 
 
2.配置微信提现
 

1) 填写完成后，提交微信审核，应用审核通过后只有登录、分享等基础接口的权限，支付权限需要申请开通


 

2) 点击申请开通，在打开的页面填写商户信息，提交微信审核

 

6) 当微信审核通过后，该应用就具备了支付权限
 

7) 进行配置
在 spring-boot-imapi 项目下打开 application.properties  配置文件，找到
微信支付相关配置,然后参照如下修改配置
 

im.wxConfig.appid  ： 应用的 AppId          
im.wxConfig.secret  ：应用的AppSecret

im.wxConfig.mchid  ： 微信商户号 
im.wxConfig.apiKey  ： 用于接口鉴权的key，自定义设置，但要和客户端保持一致。
im.wxConfig.callBackUrl ：支付成功后的回调接口url
im.wxConfig.pkPath ：微信商户平台下发的证书路径


注：微信商户平台网址：https://pay.weixin.qq.com/index.php/core/home/login
说明：微信提现权限需要在微信商户平台获取，目前申请条件为 1、已获得微信支付权限达到三个月  2. 有连续一个月的流水 
当达到这两个条件后会自动开通提现接口，无需操作什么
当获取提现权限后，从微信商户平台下载证书，配置上即可




六、管理后台使用说明
1.修改管理员自己的密码
点击右上角的下拉框，在展开的菜单中点击修改密码

 
2.	系统配置说明
系统配置项下面有三个子菜单

服务器配置：主要是服务器使用的一些配置项（包含部分服务端和客户端共同使用的配置项）

客户端配置：主要是返回给客户端使用的数据

集群配置：可配置不同地区的节点，服务器给客户端返回连接配置时根据客户端所在地区，选择最近的节点返回
 
服务器配置

 
Xmpp 心跳超时值 : 
设置成15秒即可，表示服务器在大于等于15s的时间内没有收到某个客户端的心跳数据，就将该客户端设置离线。 

是否启用消息回执：
这里的消息回执是指tigase 的消息回执插件，用于给客户端发送回执，目前启用了消息流管理，和回执插件的用途一样，所以默认不开启消息回执。
》是否开启短信验证码:
这里是指用户注册时是否使用短信验证码，可以根据需要选择开启或关闭，关闭后Android 、Ios 、Web 等客户端注册时时将不会出现短信验证码。
是否开启集群模式：
这里的集群模式是指，系统配置 ----> 集群配置
》HTTP是否安全验证：

手机号搜索用户、昵称搜索用户：
有三个选项 关闭、精确搜索、模糊搜索。设置说明如下：

手机号搜索-->关闭      昵称搜索--> 关闭 ：无法搜索到好友
手机号搜索-->精确搜索  昵称搜索--> 关闭 ：根据用户输入信息去精确匹配手机号
手机号搜索-->模糊搜索  昵称搜索--> 关闭 ：根据用户输入信息去模糊匹配手机号
手机号搜索-->关闭      昵称搜索--> 精确搜索 ：根据用户输入信息去精确匹配用户昵称
手机号搜索-->精确搜索  昵称搜索--> 精确搜索 ：用户手机号或昵称只要有一个能够被精确匹配到，就可以被搜索到
手机号搜索-->模糊搜索  昵称搜索--> 精确搜索 ：根据用户输入信息去模糊匹配手机号，精确匹配用户昵称
手机号搜索-->关闭      昵称搜索--> 模糊搜索 ：根据用户输入信息去模糊匹配昵称
手机号搜索-->精确搜索  昵称搜索--> 模糊搜索 ：根据用户输入信息去精确匹配手机号
手机号搜索-->模糊搜索  昵称搜索--> 模糊搜索 ：根据用户输入信息去模糊匹配手机号



 
手机号登录：
开启或关闭使用手机号登录
用户Id登录：
开启或关闭使用用户userId 登录
使用手机号或者用户名注册：
默认为手机号注册
选择手机号表示需要使用手机号注册用户
选择用户名表示使用用户名注册（用户名：字母或数字组合的字符串）
注册邀请码：
有三个选项：
关闭：关闭邀请码
开启一对一邀请（一码一用，必须填写邀请码才能注册）：该模式为邀请制模式，注册时必须填写邀请码，一个邀请码只能使用一次。
开启一对多邀请（一码多用，邀请码为选填项）：该模式为推广模式，注册时邀请码为选填项，每个用户有一个推广型邀请码，可使用多次。
敏感词过滤：
开启或者关闭后需要重启Tigase-server服务才能生效，该功能开启后，加到敏感词列表的词语将会被屏蔽，用户发送包含敏感词的信息，服务器不会传递给接收者。
如下图所示，包含骂人的消息，对方接受不到。
 
 
保存单聊聊天记录：
开启则在服务器的数据库中保存单聊消息记录，关闭则不保存。
修改后需要重启tigase-server 才会生效
保存群聊聊天记录：
开启则在服务器的数据库中保存群聊聊消息记录，关闭则不保存。
修改后需要重启tigase-server 才会生效

强制同步消息发送时间：
ios 推送平台：
这里的推送是指Ios手机App不在线时有消息发送过来，推送通知提醒。
注意：这里需要和服务器配置保持一致，服务器配置的时apns推送，这里就选apns推送，配置的是百度，这里就选百度。
可以选择苹果官方apns 推送或者百度推送。建议优先使用apns推送。
是否开启voip 推送：
这里的voip 推送是指进行语音电话、视频电话对方不在线时发送的推送通知，属于音视频模块。
需要在服务器配置好voip推送证书，然后在这里开启。

 

短信服务支持：
目前系统集成了阿里云短信和天天国际短信，这里需要和配置文件保持一致，配置文件配置的是阿里云短信这里则选择阿里云，反之则选择天天国际。
通讯录自动添加好友：
该功能开启后，会将手机通讯录中注册了本软件的用户，自动加为好友。
是否保存接口请求日志：
该功能开启后会在数据库保存软件调用http接口的记录，主要包含调用时间，消耗时间等信息，关闭则不保存。
保存后可以在管理后台接口日志中查看到对应的记录
 
直播赠送礼物分成比率：
这里是指直播模块中用户赠送礼物后主播和平台之间的分成比率，
如这里填写0.4，那么平台将分得40%
在线咨询链接：
 
这里可以配置一个url链接，pid是参数名，配置好后在角色管理里面点击客服
 
然后在打开的页面点击新增客服，填写手机号码
 
新增成功后，会在上面配置的在线咨询连接后面拼接上客服的userId，形成该客服的推广链接
 
注册默认成为好友的用户手机号：
该功能是在新用户注册时，加该手机号对应的用户为好友，可以填写多个用逗号分割
 
注意：下方的设置为用户默认隐私设置，在新用户注册时使用这些数据作为默认值。
默认漫游时长：
这里的漫游是指服务器储存的聊天消息数据，同步到手机App本地数据库。
可以选择不漫游、永久、一小时等选项。
 

默认过期销毁时长：
这里的过期销毁是指在聊天过程中发送的图片、文件语音等文件，超过过期时间就把文件从服务器删除掉。可以理解为文件在服务器留存的时间，永久则表示不删除。
 
客户端默认语种：
可以选择中文、英文、繁体
是否需要好友验证：
加好友需要对方同意后才能加好友。默认为开启
XMPP是否加密传输：
开启后发送消息时消息内容会使用密文进行传输，对方收到后进行解密，然后在显示出来。
是否支持多点登录：
多个不同的设备可以登录同一个账号
消息来时是否振动：
收到消息后手机振动一下
让对方知道我正在输入：
和对方聊天时如果对方正在编辑消息，对方那边会显示正在输入，该功能主要用于手机端（Android和ios）
使用Google 地图：
开启后手机端将使用Google 地图，默认使用百度地图。

 
注意：下方的设置项为建群时群组的默认设置，建群后群主可以在软件中修改这些设置项
》是否私密群组：
是否显示已读人数：
开启后在群组内发消息会显示该消息的已读人数，点击已读人数会显示已读用户列表
是否开启群组邀请确认：
开启后邀请用户加群需要群主确认后才能加入
》群组减员发送通知：
开启后当有用户退出或者
是否允许显示群成员：
开启、关闭群成员列表的显示
是否允许普通成员私聊：
关闭后群内的普通成员之间无法进行私聊。开启后则可以进行私聊。
是否允许普通成员邀请好友：
是：允许普通成员邀请好友
否：普通成员无法邀请好友
是否允许普通成员上传群共享文件：
是：群内普通成员可以上传群共享文件
否：群内普通成员不能上传群共享文件
是否允许普通成员发起会议：
是：群内普通成员可以发起视频、语音会议
否：群内普通成员无法发起视频、语音会议
是否允许普通成员发起讲课：
是：群内普通成员可以在群组里发起讲课
否：群内普通成员不能在群组里发起讲课


客户端配置
 
注意：下面的配置是通过/config 接口返回给客户端使用的
XMPP 主机host：
该地址用于连接tigase-server 。
配置Tigase-Server 所在服务器的IP地址，或者能够指向那台服务器的域名
XMPP 虚拟域名：
和tigase-server 配置文件里的virt-host 保持一致
接口url：
调用http 接口的地址
头像下载url：
说明：头像地址没有保存服务器，是通过userId 产生一个唯一的路径。
客户端通过这里配置的头像url拼接用userId 得到的头像路径，得到完整的头像下载url
资源下载url:
聊天中发送的图片、文件、语音等资源的下载url
资源上传url：
图片、文件、语音等资源的上传地址
视频服务器url：
语音通话、视频通话对应的服务器地址，属于音视频模块，没有音视频模块可以不用配置。
直播服务器url：
直播对应的流媒体服务器地址，用于推流和拉流，属于直播模块，没有直播模块可以不用配置。
Ios 应用AppleId：
公司下载页网址：
这里配置的公司下载页网址用于生成用户的二维码，主要目的是有时候用户使用微信、QQ等其它软件扫描二维码时可以跳转到该网址，从而避免出现无法识别的情况。


 
是否开放Ios 零钱：
关闭后ios App 中我的钱包将会隐藏，该设置主要用途为：在IOS App 上架审核时我的钱包可能会导致App 上架审核失败，可以在上架审核时将此设置关闭，等审核通过后在开启。
是否启用已读消息回执：
该设置目前没有用到，可以忽略
启用手机联系人：
开启或者关闭Android和IOS App的手机联系人功能
是否开启注册：
开启或者关闭Android和IOS的用户注册功能（做UI层面的屏蔽）
是否开启好友搜索功能：
开启或关闭Android 和IOS  App 的好友搜索（做UI层面的屏蔽）
是否开启普通用户搜索好友：
关闭后，普通用户登录Android和IOS  App，搜索好友功能会被屏蔽（做UI层面的屏蔽）
是否开启普通用户创建群组：
关闭后，普通用户登录Android和IOS  App，创建群组功能会被屏蔽（做UI层面的屏蔽）
是否开启位置相关服务：
开启或者关闭Android和IOS的位置服务（获取定位等）
Android 最新版本号：
用于版本更新时的版本号比对，Android版软件的当前版本号小于这个最新版本号，就会提示用户更新。
Android 最新下载url：
Android 最新的App下载地址，用于版本更新
Android 更新说明：
IOS 更新时弹出示说明信息

 
IOS最新版本号：
用于版本更新，App 当前版本号小于最新版本号，就会提示用户更新。
IOS 最新下载url：
IOS 更新包的下载地址
IOS 更新说明：
IOS 更新时弹出示说明信息
PC 最新版本号：
用于版本更新，PC（Windows）版软件的当前版本号小于这个最新版本号，就会提示用户更新。
PC最新下载url：
PC（Windows）更新包的下载地址
PC更新说明：
PC（Windows）更新时弹出示说明信息
MAC最新版本号：
用于版本更新，如果MaC版当前版本号小于这里设置的最新版本号，就会提示用户更新。
MAC最新下载链接：
MAC 版更新包的下载地址
MAC更新说明:
Android 禁用版本号：
填写后，Android低于该版本号的版本将无法使用。
IOS 禁用版本号：
填写后，苹果版低于该版本号的版本将无法使用。
PC禁用版本号：
填写后，PC版低于该版本号的版本将无法使用。
MAC禁用版本号：
填写后，MAC版低于该版本号的版本将无法使用。
3.	创建管理员账号
在管理后台左侧菜单栏找到角色管理，展开后点击管理员
 

在打开的页面点击新增管理员按钮，然后根据提示填写相关信息

 
4.	用户管理
5.	群组管理
七、服务器管理、维护
1.修改调整服务器最大连接数

使用以下命令查看当前最大连接数：（默认为1024，需要改大）
[root@shiku ~]# ulimit -n
1024

修改以下配置文件：
编辑 /etc/security/limits.conf

[root@shiku~]# vim  /etc/security/limits.conf 

*       soft     nofile    204800 
*       hard    nofile    204800 
*       soft     nproc    204800 
*       hard    nproc    204800 
在配置文件中添加以上内容

编辑 /etc/pam.d/login

[root@shiku~]# vim /etc/pam.d/login 

session required pam_limits.so 
在配置文件中添加以上内容

将以上保存好，然后重启服务器，再使用ulimit -n 

[root@shiku~]#  ulimit -n
204800



2.查看监听的端口

当某个服务出现不能正常访问，使用如下命令会查看对应的端口是否处于监听状态
#[root@shiku~]#netstat -lntp 
 

各个服务对应端口说明：
Spring-boot-imapi    数据接口服务       默认使用端口: 8092   
Tigase-server        xmpp 通讯服务     客户端使用5222，Web版使用5280端口
upload       文件上传服务         默认使用 8088 端口

mongodb    数据库               默认使用 28018 端口

Redis        缓存服务        默认使用6379 端口   

Nginx       主要用于文件访问时的目录映射   使用8089端口

fastDfs     tracker 服务使用 22122 端口  Storage 服务使用23000端口

3.设置防火墙
对外暴露尽量少的端口---我们建议使用域名配置 im的相关服务，不推荐使用ip ，使用域名配置的话就只用对外暴露很少的端口。

7.1 Iptables
 
#开放80端口(HTTP) 
[root@shiku~]# iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#开放443端口(HTTPS) 
[root@shiku~]# iptables -A INPUT -p tcp --dport 443 -j ACCEPT

3.2 firewall

Centos 7.0 自带的防火墙 为 firewall

安装：yum install firewalld
 
firewalld的基本使用
启动：[root@shiku~]# systemctl start firewalld
查看状态：[root@shiku~]# systemctl status firewalld 
禁用，禁止开机启动：[root@shiku~]# systemctl disable firewalld
停止运行：[root@shiku~]# systemctl stop firewalld
 
firewall开启和关闭端口
以下都是指在public的zone下的操作，不同的Zone只要改变Zone后面的值就可以
添加：
[root@shiku~]#firewall-cmd --zone=public --add-port=80/tcp --permanent   
注：（--permanent永久生效，没有此参数重启后失效）
重新载入：
[root@shiku~]#firewall-cmd --reload
查看：
[root@shiku~]#firewall-cmd --zone=public --query-port=80/tcp
删除：
[root@shiku~]#firewall-cmd --zone=public --remove-port=80/tcp --permanent
firewall 端口绑定ip 
用于将某个端口给特定的ip访问，如 mongodb 的数据库端口
指定IP与端口
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="5432" accept"
重新载入，使配置生效
[root@shiku~]#firewall-cmd --reload
查看配置结果
[root@shiku~]#firewall-cmd --list-all

4.查看日志
使用 tail  -f 命令可以打印实时日志

如现在需要查看 spring-boot-imapi 的日志
首先进入spring-boot-imapi 服务的目录
[root@shiku~]# cd   /opt/spring-boot-imapi   
[root@shiku~]# tail  -f  log.log

注：如在查看日志后，需要回到命令行 按 Ctrl + C
5.服务的启动与关闭
Im 相关服务的目录下都会有一个 start 文件和一个 stop 文件 
当需要停止这个服务时，在服务的目录下 执行 sh  stop命令即可，
同理 启动则在 对应目录下执行 sh start 命令

如： 现在需要停止 spring-boot-imapi 服务
[root@shiku~]#  cd  /opt/spring-boot-imapi 
[root@shiku~]#  sh  stop     

#启动 
[root@shiku~]# sh start


6.通过nginx 做端口转发

注： Nginx 需要1.9 以上版本

nginx在1.9版本之后可以充当端口转发的作用，即：访问该服务器的指定端口，nginx就可以充当端口转发的作用将流量导向另一个服务器，同时获取目标服务器的返回数据并返回给请求者。nginx的TCP代理功能跟nginx的反向代理不同的是：请求该端口的所有流量都会转发到目标服务器，而在反向代理中可以细化哪些请求分发给哪些服务器；另一个不同的是，nginx做TCP代理并不仅仅局限于WEB的URL请求，还可以转发如memcached、MySQL等点到点的请求



./configure  --prefix=/opt/nginx1.9.11  --add-module=/opt/ngx_cache_purge-2.3 
--with-stream

--add-module=/opt/ngx_cache_purge-2.3  ：  这里是添加 ngx_cache_purge-2.3 用于清理缓存可根据需要选择是否使用
--with-stream   ： with-stream 是用于tigse端口转发、集群时所需的



在 /opt/nginx1.9.11/conf/nginx.conf 配置文件中添加如下配置
注意： stream {...}  要和 http{...} 同级

stream {                                  
  	upstream xmpp{                       
     	hash $remote_addr consistent;
     	server 47.91.255.114:5222;
  	}
  	server {
         		listen 5222;
         		proxy_connect_timeout 5s;
         		proxy_timeout 5s;
         		proxy_pass xmpp;
  	}
}

如下图所示：
 


7.通过(iptables /fireWall)做端口转发

 Centos 6.X
 1) 开启服务器的ip 转发功能，默认是关闭的。
临时修改：修改过后就马上生效，但如果系统重启后则又恢复为默认值0。
[root@shiku ~]# echo  1  >/proc/sys/net/ipv4/ip_forward 
永久修改：
[root@shiku ~]# vim /etc/sysctl.conf   
#找到下面的值并将0改成1 ，默认值0是禁止ip转发，修改为1即开启ip转发功能。
net.ipv4.ip_forward = 1 
 
[root@shiku ~]# sysctl –p（使之立即生效） 
 

2）配置端口转发
说明：将 47.75.89.57 的8088 端口 转至 47.91.255.114 下的 5222 端口

 [root@shiku ~]#iptables -t nat -A PREROUTING --dst 47.75.89.57  -p tcp --dport 8088 -j DNAT --to-destination 47.91.255.114:5222 

[root@shiku ~]#iptables -t nat -A POSTROUTING --dst 47.91.255.114 -p tcp --dport 5222 -j SNAT --to-source 47.75.89.57 

 [root@shiku ~]# service iptables save       保存 iptablea 规则

 [root@shiku ~]# vim /etc/sysconfig/iptables

如下图所示，保存后会在 iptables 中产生如下内容
 
 [root@shiku ~]# service iptables restart     重启iptables 服务
注：在 /etc/sysconfig/iptables 配置中 找到以下配置注释掉，默认是开启的
 


8.windows 系统下的端口映射
通过PortMapping 程序进行端口转发，PortMapping 支持 DNF TCP UDP端口映射


    

 
八、https 配置说明
3.Tigase-server 配置https 

到证书认证机构网站下载根证书 
https://www.digicert.com/digicert-root-certificates.htm#roots
 






将下载得到的根证书上传到服务器
 
使用如下命令将 .crt 格式的根证书转换为 .pem 格式
#openssl x509 -inform DES -in DigiCertGlobalRootCA.crt -out DigicertRoot.pem -text 

 

使用cat 命令合成证书
[root@shiku~]# cat cert_oem.shiku.co.crt  cert_oem.shiku.co.key  DigicertRoot.pem  >  oem.shiku.co.pem 

然后把合成的.pem证书放到tigase-server下的certs目录下





























九、数据库加密、定时备份
1. MongoDB加密说明：
Mongodb 加密：给每个库配置用户名和密码
# cd  /opt/mongodb-3.4.0/bin    
# ./mongo  -port  28018  
//设置imapi的账号密码
> use  imapi   
> db.createUser({user: "root",pwd: "root",roles: [ "readWrite", "dbAdmin" ]})

//执行后出现如下内容，表示设置成功
 
> db.auth("root","root")  //认证测试，出现 1 表示认证成功

//设置tigase的账号密码
> use tigase
> db.createUser({user: "root",pwd: "root",roles: [ "readWrite", "dbAdmin" ]})

# vim mongo.conf    //打开mongodb配置文件

加上     auth = true  保存，然后重启mongoDb
 


打开 application.properties 配置文件，在下图所示的两个地方配置上mongoDb的用户名和密码
 


 
然后打开tigase的配置文件init-mongo.properties 配置文件，修改所有的mongoDb 连接URL，添加上用户名和密码，如下所示
注：第一个root是userName，第二个root是password
 

2. Mongodb 数据备份：
数据备份脚本

3.Redis 加密说明：
# cd  redis-3.0.3/     进入redis目录
# vim redis.conf
找到 requirepass 所在行，去掉行前的注释，并将 “foobared” 修改为所需的密码，保存文件
 

2.	打开 application.properties 配置文件，找到redis的相关配置，
在下图红框位置配置上和第一步对应的密码。
 







十、搭建服务端代码开发环境（以Eclipse为例）
1. 导入mianshi-parent项目
 
 
 

 

导入后此时的项目结构如下：

 



2. 导入 shiku-tigase 和 tigase-server 项目

说明:tigase-mongodb 是mongodb 储存模块的源码
tigase-muc 是群组模块的源码除非进行特别巨大的改动一般这两个模块是不需要改动的可以暂时不用导入到ide中
 


 
 


导入完成后如下图所示： 


3. 导入shiku-push项目
 

 
4. 导入upload项目
 

 


5. 导入下载不到的jar包解决pom报错
导入完成后的项目结构如下图所示：

 

鼠标点击顶部菜单中的 project -----> clean 

 

将下载不到的maven 依赖包拷贝到本地maven仓库对应的目录中

首先在源码文件目录中找到maven_jars.zip 文件解压，找不到的可以通过如下地址下载一份：http://47.75.89.57/maven_jars.zip

 
解压后如下图所示：
 

然后选中其中一个项目鼠标右键点击选择maven----> Update project

 


 

6. 将lifecycle-mapping-metadata.xml 文件导入到eclipse中
鼠标点击顶部菜单栏中的 Window-----> Prefereces
 

然后找到maven---->Lifecycle Mappings
 

 

注意：下面这个是需要导入的文件，右键点击选择  保存到文件
 

7. 给eclipse安装Lombok 插件
Lombok 下载地址：https://projectlombok.org/download
 


 

 

出现这个画面说明lombok安装成功，然后把eclipse重启一下。

 


最后把eclipse 重启一下

8. 编译并运行 mianshi-im-api 项目
○1 首先将 mianshi-parent项目maven clean、maven install   
需要注意要等到maven clean执行完成，控制台打印BUILD SUCCESS后在执行maven install

 



○2 maven clean、maven install   mianshi-service项目等到成功后再 maven clean、maven install   mianshi-imapi 项目
注意：首次导入需要进maven 编译，成功后后面就不需要每次都编译了（打包除外）

 

○3 修改配置文件application.properties 里mongodb 、redis 、tigase 的连接地址 

 

请参照下图标记位置，找到配置文件对应的配置项进行修改

 
 
 

○4 运行项目，找到 mianshi-im-api 项目中的 Application 类，
右键 Debug As ---> java Application 
 


9. 编译并运行tigase相关项目

○1 clean tigase 项目

                

○3 配置开发环境tigase-server运行脚本，如下图所示
 
 
说明：如上2图所示，在Program arguments中添加参数：
--property-file etc/init.properties
在VM arguments中添加参数：
-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
-server -Xms100M -Xmx200M -XX:PermSize=32m -XX:MaxPermSize=256m -XX:MaxDirectMemorySize=128m
-Djava.ext.dirs=E:\Workspace\Java\tigase\tigase-server\jars
-Dtigase-configurator=tigase.shiku.conf.ShikuConfigurator 


○4 修改tigase-server 项目中etc目录下的init-mongo.properties 配置文件，
关于配置文件的修改说明请参看 配置文件示例文件夹中的init-mongo.properties
     
 

○5 修改完毕后，再次clean  tigase-server项目,如下图所示到这里项目应该是正常的，不会存在报错的情况，如仍然存在报错的情况，请根据错误信息稍作调整或者联系我们。
 

○6 clean 完成后，maven clean tigase-server项目

出现BUILD SUCCESS 表示完成
 

然后maven install  tigase-server 项目，首次编译需要下载依赖文件，请耐心等待
 

○7 运行tigase-server项目
 找到XMPPService类，右键 Debug As ---> java Application
  

10. upload 项目
Upload 项目是用来上传文件的，基本不需要改动，关于部署的具体步骤请参看本文档的项目部署部分。





















