# 2017/11/3 #
  ## Kubernetes(k8s)介绍 ##  
   基本概念：  
    K8s是Google开源的，基于容器技术的分布式架构系统。它含有多个节点和多个组件，节点主要包括etcd，master 和 minion，每个节点又有多台机器。其中etcd 作为高性能存储服务，一般独立为一个节点，master 端需要安装 kube-apiserver、kube-controller-manager和kube-scheduler 组件，minion 节点需要部署 kubelet、kube-proxy、docker 组件。
  
   优势：  
    1. 使用Docker对应用程序包装、实例化  
    2. 以集群的方式运行、管理跨机器的容器  
    3. 解决Docker跨机器容器之间的通讯问题  
    4. 智能的自我修复机制
    
   基本架构：
   ![](https://i.imgur.com/q8mnibW.png)
   **部署模型：**  
   **Pod：**   
    是K8s集群里可运行的最小单位，包含一组容器和卷，里面运行着很多container,可以通过Replication Controller（RC）使用Pod模板创建出多份拷贝。同一个Pod里的容器共享同一个网络命名空间，可以使用localhost互相通信,这些container相互合作共同完成一个完整的流程，而这个流程则被pod所监视管理。Pod是短暂的，容易受到网络和机器故障等的原因影响，这需要service和label组件来保障它的稳定运行。   
   **Volumn：**     
    各个pod内部维护的文件系统，与pod生命周期一致，即使pod里的container消亡文件系统仍然保存，为之后新产生的container继续服务  
    **RC：**  
    RC确保任意时间都有指定数量的Pod“副本”在运行，副本数量不够/过多时它会主动增加/减少副本数量，副本无响应时它会主动替换并生成新的替代副本。这个特性在执行滚动升级以及分担各个pod的压力时很有用。当创建RC时需要指定Pod模板（用来批量产生pod）以及label（用来监控特定pod而附加的标签）    
    **Label：**  
    一个Label是标记某一Pod的一对键/值对，标记完后使用可以Selectors选择带有特定Label的Pod，如果同一Pod有多个副本则通过负载均衡策略选择出最合适的一个。   
   **Node：**  
    一台实际运行着的物理或者虚拟机器，通常称为Minion。每个节点都运行着Kubernetes的关键组件。  
   **K8s Master：**  
     负责管理各个节点的主机，用以实现集群交互，外部控制等。  
   **Service：**  
     Serives出现屏蔽掉了pod变动对服务调用方带来的影响，所有请求都会先通过这个组件，然后proxy services调用Label选择器选择合适的pod来处理。  
  
   **代理节点：**  
   **Kubelet：**  
     Kubelet组件运行在特定minion节点上，负责维护在特定主机上运行的一组容器，主要功能就是定时从某个地方获取节点上 pod/container 的期望状态（运行什么容器、运行的副本数量、网络或者存储如何配置等等），并调用对应的容器平台接口达到这个状态。  
   **Kube-Proxy：**
     Kube-proxy是一个简单的网络代理和负载均衡器。它具体实现Service模型，每个Service都会在所有的Kube-proxy节点上体现，负责处理分配外来的各种请求。  

   **服务节点：**  
   **ETCD：**  
     etcd是一个一个高可用、强一致性的键值存储仓库，部分机器故障并不会导致功能失效，用于配置共享和服务发现。它通过Watcher机制保障了在同一个分布式集群中的进程或服务能够精准即时找到对方并建立连接，实现了分布式集群中消息的实时传播与更新。  
   **K8s API Server：**  
     整个集群管理的 API 接口，所有对集群进行的查询和管理都要通过 API 来进行，所有模块之前并不会之间互相调用，而是通过和 API Server 打交道来完成自己那部分的工作。  
   **Scheduler：**  
     根据特定的调度算法将pod调度到指定的工作节点（minion）上


   ## 基于阿里云的安装流程 ##  
    <input type="hidden" value="https://yq.aliyun.com/articles/68921">
   备好两台阿里云ECS，一台作为master一台作为node，系统均为ubutu16.04。首先申请开通各自的access_key_id和access_key_secret，然后关闭两台ECS的防火墙`ufw disable`并暂时禁用setenforce `setenforce 0`（要想永久禁用就修改`/etc/selinux/config`文件将`SELINUX=enforcing`改为`SELINUX=disabled`，随后重启机器即可）  
   **Step1.**  
   在master主机上执行命令  `curl -L 'http://aliacs-k8s.oss-cn-hangzhou.aliyuncs.com/installer/kubemgr.sh' | bash -s nice --node-type master --key-id $ACCESS_KEY_ID --key-secret $ACCESS_KEY_SECRET --region $REGION --discovery token://`  
   注意替换keyId,keySecret以及region（华南区region为深圳）  
  **Step2.**  
   记录输出记录中出现的注意token，`TOKEN=token://xxxxxx:xxxxxxxxxxxxxxxx@12x.2x.24x.21x:989x`  
  **Step3.**  
   同理在子机node上执行命令  `curl -L 'http://aliacs-k8s.oss-cn-hangzhou.aliyuncs.com/installer/kubemgr.sh'| bash -s nice --node-type node --key-id $ACCESS_KEY_ID --key-secret $ACCESS_KEY_SECRET --region $REGION --discovery $TOKEN`  
   此时已经成功地安装的了一个master和一个node节点。可以重复在其他机器上执行安装node操作来添加更多节点。但是要让Kubernetes能正常运行您还需要为集群添加网络支持。  
   **Step4.**  
   在matser节点上输入`kubectl get nodes`可查看到从节点加入后的信息  
   **Step5.**  
   为主机添加网络支持用来打通主从节点间的网络通信。在主节点上输入  `curl -sSL http://aliacs-k8s.oss-cn-hangzhou.aliyuncs.com/conf/flannel-vpc.yml -o flannel-vpc.yml`获得相应的yml文件，接着`vim flannel-vpc.yml`进入文件，将里面出现的`[replace with your id]`按提示替换成自己的accessKeyId和accessKeySecret.(注意：替换时要把外面的中括号一并删掉)  
   **Step6.**    
   主节点上输入`kubectl apply -f flannel-vpc.yml`运行yml脚本，接着输入`kubectl --namespace=kube-system get ds`查看所有kube-system命名空间下的所有daemonsets，如果看见一个名字叫kube-falnnel的ds处于Running状态. 说明网络部署成功。  
   **Step7.**
   主节点上运行命令`kubectl apply -f http://k8s.oss-cn-shanghai.aliyuncs.com/kube/kubernetes-dashboard1.5.0.yaml`即配置好了web ui界面，通过命令`kubectl --namespace=kube-system describe svc kubernetes-dashboard`进行查看得到`NodePort`，然后直接访问`http://IP:NodePort`即可得界面  
   **Step8.**  
   查找错误时先看pod的启动情况：`kubectl --namespace=kube-system get po`，看那些服务没启动，接着查看kube的日志进一步了解`journalctl -u kubelet -f`  
   **Step9.**  
   若在上一步的日志中出现`Error adding network: open /run/flannel/subnet.env` `Error while adding to cni network`类似的错误，同时dashboard的svc未能启动，则先拉镜像`docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/kubernetes-dashboard-amd64:v1.5.0`然后按照指示创建文件`mkdir -p /run/flannel`，然后进入`cd /run/flannel`，接着创建所需文件`vim subnet.env`    
        
         FANNEL_NETWORK=172.16.0.0/16   //跟阿里云->VPC->华南->路由器里的ECS实例一致,一般都一样
		 FLANNEL_SUBNET=172.16.0.1/24
		 FLANNEL_MTU=1500
		 FLANNEL_IPMASQ=true
   然后重启服务`service kubelet restart`，再次查看日志应该就恢复正常了  

   **Step10.** 
   任何时候都可以卸载kubernetes：  `curl -L 'http://aliacs-k8s.oss-cn-hangzhou.aliyuncs.com/installer/kubemgr.sh' | bash -s nice --node-type down`
    
   **提醒(慎用)**：若执行kubectl相关命令时出现无法访问`localhost:8080`的错误提示时要先尝试重启harbor`./install`，若不起效才到`/etc/kubernetes/manifests/kube-apiserver.json`里找到`-insecure-bind-address`参数将后面的`127.0.0.1`修改为`0.0.0.0`

## 部署Web服务(简易) ##
   <input type="hidden" value="http://blog.csdn.net/hty46565/article/details/77100099">
  **Step1.**  
   在阿里云上申请并创建镜像仓库(https://cr.console.aliyun.com/?spm=5176.doc60751.2.3.z7UJ18#/imageList) 创建完后点击管理可进入详情界面  
  **Step2.**  
   按照管理界面里的指引在本地远程登录，给本地镜像打tag并上传    
  **Step3.**  
   编写yaml文件：  

      1 apiVersion: v1      //描述RC对象的版本是v1
	  2 kind: ReplicationController    //声明RC对象
	  3 metadata:     //metadata中的是对此RC对象描述信息
	  4   name: myweb      //此RC对象在default命名空间中名为myweb，同一个命名空间中的命名不同
	  5 spec:     //spec中是对RC对象的具体描述
	  6   replicas: 2    //创建副本数，单位是pod
	  7   selector:      //选择器，用来选择对象的
	  8     app: myweb    //选择了标签为app: myweb的pod
	  9   template:     //模版，以下用来描述创建的pod的模版
	 10     metadata:   //对pod模版描述的元数据
	 11       labels:      //给以下的东西打上标签，以让selector来选择
	 12         app: myweb   //给pod模版打上app: myweb这样的标签
	 13     spec:             //对pod模版的具体描述
	 14       containers:         //以下就是要放入pod模版中的容器了
	 15       - image: registry.cn-shanghai.aliyuncs.com/yf_registry/local-cat  //选择远程仓库里的某一镜像
                imagePullPolicy: IfNotPresent  //优先从本地拉取镜像
	 16         name: myweb         //容器名
	 17         resources:           //给该容器分配的资源大小
	 18           limits:
	 19             cpu: 500m     //比1还要小的单位
	 20             memory: 600Mi  //注意控制大小
	 21         ports:         //容器端口号
	 22         - containerPort: 8080           
   **Step4.**  
    运行：`kubectl create -f web-rc.yaml`,通过命令`kubectl get pods`可看出已生成一个pod  
   **Step5.**  
    编写web-svc.yaml  
   
      1 apiVersion: v1
	  2 kind: Service   //对象是Service
	  3 metadata:
	  4   name: myweb   //Service的名字，可随意取
	  5 spec:  
	  6   ports:
	  7   - name: myweb-svc         //端口名称，Service是必须指定端口名称的
	  8     port: 8080          //Service的端口号
	  9     targetPort: 8080        //容器暴露的端口号
	 10     nodePort: 31111       //node的真实端口号
	 11   selector:
	 12     app: myweb     //Service选择了标签为app: myweb的pod
	 13   type: NodePort  
   **Step6.**  
   运行：`kubectl create -f web-svc.yaml`,通过指令`kubectl describe pod myweb`得到集群给这个pod随机分配的外网IP，然后浏览器输入`外网IP:31111`即可访问  
   **Step7.**  
   删除pod：先`kubectl delete rc myweb`然后 `kubectl  delete svc myweb `即可。
   
  


## 其他： ##
   `kubectl get pods`：查看部署了哪些实例  
   `kubectl create -f xxx.yaml`： 创建pod实例  
   `kubectl get rc` ： 查看负责控制pod的各个RC状态  
   `kubectl get svc(service)`  
   `kubectl get deployment`  
   `kubectl delete pods/rc/svc/depolyment xxx`：删除命令（先删rc，再svc，pod就自动删了，既没rc又没svc就找depolyment）  
   `kubectl describe rc/svc/depolyment xxx`(查看某一服务的详情，类似于log功能)  
   `service kubectl restart`：重启服务  
   `kubectl get svc/pods/deployment --all-namespaces -o wide`：查看所有域名下的各类服务  
   `kubectl delete deployment/svc kubernetes-dashboard --namespace=kube-system`：删除特定域名下的服务 
   