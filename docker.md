# 2017/10/23 #

  ## 构建本地mysql镜像并运行 ##    
 <input type="hidden" value="http://www.cnblogs.com/han-1034683568/p/6941337.html?utm_source=tuicool&utm_medium=referral">
1. 本地选择一个空文件夹，创建Dockerfile,内容：

        FROM mysql:5.7
    	#设置免密登录
    	ENV MYSQL_ALLOW_EMPTY_PASSWORD yes
    	#将所需文件放到容器中
    	COPY setup.sh /mysql/setup.sh
    	COPY schema.sql /mysql/schema.sql
    	COPY privileges.sql /mysql/privileges.sql
    	#设置容器启动时执行的命令
    	CMD ["sh", "/mysql/setup.sh"]
2. 写入setup.sh（运行文件，包括启动mysql服务以及加载两个sql文件）
3. 写入schema.sql设置几张表和相关字段，用来检测执行成功与否
4. 写入privileges.sql用来设置登入时的用户权限，密码
5. 创建镜像：`docker build -t mydocker-mysql .`  
    &emsp;docker build 为创建镜像命令，名称为mydocker-mysql，'.'表示当前目录，即Dockerfile文件所在的目录
6. `docker images`可查看镜像，`docker run -d -p 13306:3306 mydocker-mysql`运行镜像，`docker logs 镜像ID` 查看镜像启动的相关信息。
7. 进入镜像内部：`docker exec -it 镜像ID /bin/bash`
8. 下面就是mysql常规登陆步骤，输入mysql -u privileges.sql中设置的user -p，输入密码，use 设置的DB 即可查看具体信息，也支持navicat远程连接获得结果



## 创建本地私有仓库##
  通过官方提供的registry镜像来简单搭建一套本地私有仓库环境  
1. 使用registry启动私有仓库的容器  
   &emsp; `docker run -d -p 5000:5000 -v /local/my_registry:/tmp/registry registry`  
   &nbsp;&emsp;若之前没有安装registry容器则会自动下载并启动一个registry容器，创建本地的私有仓库服务。默认情况下，会将仓库创建在容器的/tmp/registry目录下，可以通过 -v 参数来将镜像文件存放在本地的指定路径上(冒号前的参数)  <p>
2. 重新标记一个本地镜像当做私有仓库的版本，以hello-world为例：  
  &emsp;` docker tag hello-world localhost:5000/hello-world`  
3. 运行docker images可得知此时又多了一个hello-world的镜像，两者的imageID相同   
4. 启动registry:`docker start registryID`  
5. 将本地镜像推送到本地仓库中：  
   &emsp;`docker push localhost:5000/hello-world`  
6. 查看本地仓库中的镜像列表（得到的是json式）:  
   &emsp;`curl -XGET 主机IP:5000/v2/_catalog`  
7. 拉取镜像（实现远程计算机拉取本地镜像）  
   &emsp;`docker pull 主机IP:5000/hello-world`  <p>
PS：在步骤四操作后可能报错connection refuse，解决方案：  
     &emsp; &emsp;修改Docker配置文件  
	 &emsp; &emsp;vim /etc/default/docker  
	 &emsp; &emsp;增加以下一行  
	 &emsp; &emsp;DOCKER_OPTS="$DOCKER_OPTS --insecure-registry=主机IP:5000"  
	 &emsp; &emsp;重启Docker  
	 &emsp; &emsp;sudo service docker restart


## 其他 ##
查看各个镜像ID：`sudo docker ps -a`  
查看本地镜像有哪些：`docker exec -it registryID ls /var/lib/registry/docker/registry/v2/repositories`  
`ctrl+d` ：退出容器且关闭, 通过`docker ps `查看的结果没有相关记录   
`ctrl+p+q`：退出容器但不关闭, `docker ps`查看的结果有相关记录 