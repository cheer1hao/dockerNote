# 2017/10/25 #
  ##  Docker下载与安装(阿里云docker-ce) ##  
   step 1: 更新软件   
     &emsp;`sudo apt-get update`  

   step2：安装https证书  
     &emsp;`sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common`  

   step3：安装GPG证书（用来交换识别加密信息）  
     &emsp;`curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`  

   step4：写入软件源信息  
     &emsp;`sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"`  

   step5：更新并安装 Docker-CE  
     &emsp;`sudo apt-get -y update` (-y：参数，对安装过程中所有的问题自动回答yes)  
     &emsp;`sudo apt-get -y install docker-ce`  

   step6：启动docker服务  
     &emsp;`sudo service docker start`  

   step7：检查docker版本详情  
     &emsp;`sudo docker -v`  
     &emsp;`sudo docker info`(更加详细，包括目前内部的容器数，镜像数等)  
   <p>
   step8：配置加速器  
    <input type="hidden" value="https://cr.console.aliyun.com/?spm=5176.100239.blogcont29941.12.tP4G26#/accelerator">
   &emsp;进入 /etc/docker目录，新建daemon.json，输入内容：  

       {
        "registry-mirrors":["https://jpox4hz9.mirror.aliyuncs.com"]
       }  
   &emsp;保存退出，使用命令`service docker restart`重启docker，完成配置    
  
   step9:检验加速成果  
      &emsp;`ps -ef  | grep dockerd` 如果能看到配置项里有registry-mirror这一项就代表配置成功
   <p>
   PS：卸载docker-ce  
   &emsp; `sudo apt-get purge docker-ce`  
   &emsp; `rm -rf /var/lib/docker`


##  OPENJDK1.7下载与安装 ##
  <input type="hidden" value="http://blog.csdn.net/u013403478/article/details/51012113">
  
  step1：下载jdk、jre  
    &emsp;`sudo apt-get install openjdk-7-jdk`  
    &emsp;`sudo apt-get install openjdk-7-jre`  

  step2：进入/etc目录，修改profile文件，在文件末尾添加环境变量  

    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
	export JRE_HOME=$JAVA_HOME/jre 
	export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
	export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH  
   &emsp;(其中JAVA_HOME为openjdk安装路径，可自行根据需要修改)  
  
  step3：保存退出  
    &emsp;`source /etc/profile`  

  step4：重新登录账户生效，检验  
    &emsp;`java -version`  

  若中途因网速等原因无法下载，可选择从阿里云上下载相关镜像：  
  &emsp; 网址：https://dev.aliyun.com/detail.html?spm=5176.1971733.2.7.cmdtqZ&repoId=1224  
  &emsp; 查找相关镜像，点击进入，得到相关dockerfile文件，复制到本地运行即可    


## TOMCAT7下载运行 ##
  <input type="hidden" value="http://blog.csdn.net/q26335804/article/details/47806105">
  step1：拉取docker hub中的tomcat镜像  
   &emsp;`docker pull tomcat:7`  

  step2：查看特定镜像是否存在  
   &emsp;`docker image tomcat:7`  

  step3：基于tomcat:7这一镜像创建一个新的容器  
   &emsp;`docker create --name dev_tomcat -p 8080:8080 tomcat`

  step4：启动容器  
   &emsp;`docker start dev_tomcat`  

  step5：查看容器是否运行  
   &emsp;`docker ps`  

  此时可以外部浏览器访问  

  step6：进入容器内部查看  
   &emsp;docker exec -it ContainerID /bin/bash  
  &emsp;可查看tomcat日志，webapps内部相关信息 

## 简单创建本地私有仓库 ##
   通过官方提供的registry镜像来简单搭建一套本地私有仓库环境  
1. 使用registry启动私有仓库的容器  
    &emsp; `docker run -d -p 5000:5000 -v /local/my_registry:/tmp/registry registry`  
   &emsp; 若之前没有安装registry容器则会自动下载并启动一个registry容器，创建本地的私有仓库服务。默认情况下，会将仓库创建在容器的/tmp/registry目录下，可以通过 -v 参数来将镜像文件存放在本地的指定路径上(冒号前的参数)  

2. 重新标记一个本地镜像当做私有仓库的版本，以hello-world为例：  
  &emsp;` docker tag hello-world localhost:5000/hello-world`  

3. 运行docker images可得知此时又多了一个hello-world的镜像，两者的imageID相同   

4. 启动registry  
    &emsp;`docker start registryID`  

5. 将本地镜像推送到本地仓库中：  
   &emsp;`docker push localhost:5000/hello-world`  

6. 查看本地仓库中的镜像列表（得到的是json式）:  
   &emsp;`curl -XGET 主机IP:5000/v2/_catalog`  

7. 拉取镜像（实现远程计算机拉取本地镜像）  
   &emsp;`docker pull 主机IP:5000/hello-world`  <p>

PS：在步骤四操作后可能报错connection refuse，解决方案：  
   &emsp;修改Docker配置文件    
	&emsp;  `vim /etc/default/docker`  
   &emsp;增加以下一行  
	 &emsp; `DOCKER_OPTS="$DOCKER_OPTS --insecure-registry=主机IP:5000" `   
   &emsp;重启Docker  
	 &emsp; `sudo service docker restart`


## 构建本地镜像及运行 ##
  <input type="hidden" value="http://www.cnblogs.com/han-1034683568/p/6941337.html?utm_source=tuicool&utm_medium=referral">


## OPENJDK7+TOMCAT7+hosts构建基础环境镜像 ##
  <input type="hidden" value="http://blog.csdn.net/u013469562/article/details/72897289"> 
  准备工作：tomcat7以及openjdk7的文件，放在同一目录下 （openjdk下载地址:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html） <p> 

  在同一目录下编写Dockfile:      
       
        FROM ubuntu:14.04.4      #镜像版本，需从仓库里下载  
		RUN echo "120.25.135.172 conf.156156.com"  >> /etc/hosts  #向hosts里新增的内容,内网用  
		WORKDIR /usr       #切换镜像目录   
		RUN mkdir java     #新建java目录  
		WORKDIR /usr/java  #切换到java目录中  
		RUN mkdir jdk  
		WORKDIR /usr/java/jdk  
		ADD jdk7 /usr/java/jdk  #将jdk文件挂载到镜像目录中  
		WORKDIR /opt  
		RUN mkdir tomcat  
		ADD apache-tomcat-7.0.68 /opt/tomcat   #tomcat同理 
	 
		ENV JAVA_HOME /usr/java/jdk           #环境配置  
		ENV JAVA_BIN /usr/java/jdk/bin  
		ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar  
		ENV CATALINA_HOME /opt/tomcat  
		ENV CATALINA_BASE /opt/tomcat  
		ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin  	

		EXPOSE 8080                          #暴露端口  		
		CMD /opt/tomcat/bin/catalina.sh run  #CMD指令  
   保存退出  

   生成镜像：`sudo docker build -t local/cat .` （注意最后有圆点符号）  
   查看镜像是否存在：`sudo docker images`  

   生成容器并运行：`sudo docker run -d -p 8080:8080 local/cat`  
   （非交互式的，若要交互式则在-d后加参数-it）  
   查看容器是否运行：`sudo docker ps`  
   查看容器启动日志：`docker logs ContainerName `   

   若容器处于运行态，可在浏览器中输入`IP:8080`进行检验  
   若检验不成功，检查之前指令是否有误，dockerfile内容，网络（宿主机和虚拟机互ping）等  

   进入容器内部：`docker exec -it ContainerID /bin/bash`  
   查看hosts是否更改成功：`cat /etc/hosts`  
   切换到tomcat目录查看日志：`cd tomcat/logs`

   至此镜像生成与运行完成  

   退出某一容器（不关闭容器）：`ctrl+p+q`  
   退出某一容器（关闭容器）：`ctrl+d`  
   查看所有容器的详情：`docker ps -a`  
   停止某一容器的运行：`docker stop ContainerID`  
   删除某一容器：`docker rm ContainerID`  
   强制删除某一运行中的容器：`docker rm -f ContainerID`  
   删除某一镜像：`docker rmi ImageName`
   强制删除有容器引用的镜像：`docker rmi -f ImageName`


## WebWar+基础环境镜像构建Web镜像 ##
 选择某一目录，放入需要解析的war包，同时构建Dockerfile

       FROM local/cat:latest  #之前构建的基础镜像
       ENV app_id=ecp-boss   
       ENV app_stage=develop   #java_opts环境变量参数
       COPY ecp.war /opt/tomcat/webapps      #拷入tomcat挂载的虚拟目录中
       CMD /opt/tomcat/bin/catalina.sh run   #运行

 构建Web镜像：`docker build -t ecp-boss-develop . `  
 查看镜像：`docker images`  
 产生容器并进入：`docker run -it -p 8081:8080 ecp-boss-develop /bin/bash`  
 （/bin/bash是进入容器的指令，-d是指后台运行，不进入的情况是`docker run -dit -p 8080:8080 ecp-boss-develop`）  
 检查hosts：`cat /etc/hosts`  
 进入webapps目录查看是否有war包： `cd /tomcat/webapps`    
 进入bin目录重启tomcat：`cd ../bin   ./startup.sh`
 退出容器：`ctrl+p+q`  
 实际在浏览器中访问：`IP:8081/WarName/`

 至此基础的Web镜像构建完成


## 其他 ##
 &emsp;采用桥接模式的虚拟机重启后外网Ping不通，可在`/etc/rc.local`里加上一句`service network start`，然后**重启虚拟机**可暂解，但虚拟机内部仍然Ping不出去，不过外部可连进来。如果无效，可尝试改成`service network restart` 