# 2017/10/26 #
  ## TOMCAT+JDK+WAR生成镜像流程 ##  
    主要内容：hosts可放在run的命令中动态生成；JAVA_OPTS环境变量单独写成文件，run时指定；完整流程  
    未完成部分：ubuntu14.04镜像中下载的docker刚启动就会自动关闭，jdk部分仍用文件代替  


  ** step1**.任选路径编辑Dockerfile文件，目的：获得ubuntu镜像，挂载tomcat，jdk文件到指定的虚拟目录中  

        FROM ubuntu:14.04.4
		WORKDIR /usr/local
		RUN mkdir java
		WORKDIR /usr/local/java
		RUN mkdir jdk
		WORKDIR /usr/local/java/jdk
		ADD jdk7 /usr/local/java/jdk
		WORKDIR /usr/local/java
		RUN mkdir tomcat
		WORKDIR /usr/local/java/tomcat
		ADD apache-tomcat-7.0.68 /usr/local/java/tomcat
		
		ENV JAVA_HOME /usr/local/java/jdk
		ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
		ENV CATALINA_HOME /usr/local/java/tomcat
		ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
		
		EXPOSE 8080
		
		CMD /usr/local/java/tomcat/bin/catalina.sh run
		      
     
   细节之前已经提过，不再重复  
   **step2**.编译  
   &emsp; `docker build -t local/cat .`  
   **step3**.生成容器  
   &emsp; `docker run -d -p 8080:8080 local/cat`   
   此时可通过`IP:8080`成功访问  

   PS：此处可能会出现在步骤3后输入`docker ps`发现生成的容器已停止，此时需要输入`docker -it -p 8085:8080 local/cat /bin/bash`进入容器内部，更改`tomcat/bin`文件夹权限，确保自己能在容器内使用`./startup.sh`命令重启tomcat，然后`ctrl+p+q`退出，重新访问浏览器（IP:8085）即可。
   
   **step4**.另选目录放入web.war，编辑Dockerfile  
   
    FROM local/cat:latest
	COPY ecp.war /usr/local/java/tomcat/webapps
	CMD /usr/local/java/tomcat/bin/catalina.sh run

  注意：因为是最后一层镜像，因此Dockerfile里不必加入环境变量，只需要将war传入tomcat目录下即可

   **step5**.生成jvopts文件，里面写入JAVA_OPTS等其他环境变量

     app_id=ecp-boss
     app_stage=develop

   **step6**.生成镜像  
    &emsp;`docker build local/ecp .`  
   
   **step7**.生成并进入容器  
    &emsp;`docker run -it -p 8081:8080 --add-host=conf.156156.com:120.25.135.172 --env-file=/usr/local/tomjdk/jvopts  local/ecp /bin/bash`

   &emsp;命令解释：`add-host`为向生成的容器的`/etc/hosts`中添加变量，形式为`host:ip`的形式；`env-file`后面为自己写的环境变量文件， `/bin/bash`表示进入容器内部

   **step8**:检验上述命令正确性:`cat /etc/hosts`,得到结果：  
       ![](https://i.imgur.com/Q8ctaHu.png)

   **step9**:进入容器内部后就可以到webapps里检验war包是否存在是否已解压，到bin文件里重启tomcat，到logs文件里查看tomcat启动日志

   step10:浏览器输入`IP:8081/war`包名即可进行访问（此处war包未解压，但tomcat已运行）  
      ![](https://i.imgur.com/1o2WSC4.png)



## 其他 ##

  &emsp;Dockerfile里涉及到的文件路径尽量用绝对路径，`/file`即代表根目录下的file文件，`cd /`即返回到根目录    
      &emsp;docker启动日志路径：/var/log/docker.log  
  &emsp;以守护形态生成container并运行（不进入容器中）： 
   &emsp;&emsp;`sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"`