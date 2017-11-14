# 2017/11/9 #

## 创建服务器SSL证书 ##
  <input type="hidden" value="https://github.com/vmware/harbor/blob/master/docs/configure_https.md">
 **Step1.**  
   在/root(根)目录下创建cert文件夹`mkdir cert`并进入`cd cert`  
 **Step2.**  
   创建CA证书：`openssl req -newkey rsa:4096 -nodes -sha256 -keyout ca.key -x509 -days 365 -out ca.crt`  
 **Step3.**  
   创建CSR：`openssl req -newkey rsa:4096 -nodes -sha256 -keyout yourdomain.com.key -out yourdomain.com.csr`  
   注意：yourdomain.com可替换为IP或者该服务器的域名，如果替换成域名则要在`/etc/hosts`里追加相应的DNS解析（或许还有其他操作，暂不讨论，此例中一律使用IP）  
 **Step4.**  
   与服务器里的registry host建立联系：  

    echo subjectAltName = IP:192.168.1.101 > extfile.cnf  
    openssl x509 -req -days 365 -in yourdomain.com.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out yourdomain.com.crt
   
   PS：域名版：`openssl x509 -req -days 365 -in yourdomain.com.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out yourdomain.com.crt`  
  **Step5.**  
   至此服务端全部文件配置完成，对于运行docker的客户端只需将服务端产生的ca.crt复制到客户端相应目录里即可(此处假设服务端客户端为同一个服务器)：  
  
       mkdir -p /etc/docker/certs.d/reg.yourdomain.com/
       cp ca.crt /etc/docker/certs.d/reg.yourdomain.com/


## Harbor搭建过程 ##
  **目的**：配合K8s的https访问模式创建一个支持加密访问的私人仓库  
  **Step1.**  
  去官网`https://github.com/vmware/harbor/releases`下载最新版本的离线版，通过winscp放入自建目录中，解压`tar -xzvf harbor-offline-installer-v1.2.2.tgz`  
  **Step2.**  
  进入`harbor`目录，修改配置文件`vim harbor.cfg`，将`hostname`改成服务器`IP`，将`ui_url_protocol`改为`https`，将`ssl_cert`与`ssl_cert_key`改成之前建好的SSL证书所在路径`/cert/yourdomain.name.crt`和`/cert/yourdomain.name.key`,将`secretkey_path`改为`/`，其余暂时不用动，保存退出  
  **Step3.**  
  下载go新版，下载pip组件(阿里云上都做好了)，安装docker-compose`pip install docker-compose`   
  **Step4.**  
   若第一次使用，则运行`./install.sh`等待系统自动编译完成；如果harbor已经运行，则先执行`./prepare`，然后`docker-compose down`,最后 `docker-compose up -d`即可  
  **Step5.**  
   此时可根据提示在浏览器中输入`https://IP`即可访问harbor界面，当然要先添加安全证书，也可以在命令行中登录，即`docker login IP`，输入默认的用户名`admin`和密码`Harbor12345`即能成功登陆。  
  **Step6.**  
   若想更改harbor配置，先运行`docker-compose down -v`停止harbor运行,接着修改配置`vim harbor.cfg`，然后`./prepare`重新构建，最后运行`docker-compose up -d`  

  **Step7.**  
   使用命令`docker-compose down`可卸载harbor组件，然后进入`/data`目录发现还有残余，通过`rm -f /data/database`以及`rm -f /data/registry`进行清除