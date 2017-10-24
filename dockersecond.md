# 2017/10/24 #
  ## OPENJDK7+TOMCAT7构建本地镜像 ##
   <input type="hidden" value="http://blog.csdn.net/u013469562/article/details/72897289">  
   准备工作：tomcat7以及openjdk7的文件，放在同一目录下 （openjdk下载地址:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html） <p> 

   在同一目录下编写Dockfile:      
       
        FROM ubuntu:14.04.4      #镜像版本，需从仓库里下载  
		RUN echo "120.25.135.172 conf.156156.com"  >> /etc/hosts  #向hosts里新增的内容，内网  
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
   生成镜像：`sudo docker build -t local/cat .`  
   生成并进入活跃的容器：`sudo docker run -t -i local/cat /bin/bash`  
   进入某一存在的容器：`docker exec -it 容器ID /bin/bash`  
   此时可在浏览器中通过`IP:8080`进行访问
