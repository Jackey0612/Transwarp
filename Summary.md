### 



## Docker相关概念

- docker的存在是为了更方便快捷地去部署一个应用。
- 两个重要的概念，镜像和容器。镜像它的作用有点像maven工程的pom文件，只要写好了pom文件，IDEA就会自动去下载这些需要的依赖，镜像会把所有需要用到的依赖打包，与pom不同的是，镜像还包含着应用程序。
- 镜像和容器的关系，相似于类和对象的关系，镜像是类，容器是镜像的一个实例化。一个镜像可以实例化出好多容器。



## Docker常用指令

```shell
# 从docker hub拉取镜像
docker pull <image-name>:<tag>

# 查看镜像列表
docker images

# 查看正在运行的容器
docker ps

# 查看所有的容器，包括已经停止的容器
docker ps -a

# 查看容器的详细信息
docker inspect <容器名称|容器ID>

# 运行容器，命令语法：docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# 常用的命令选项如下（需要注意的是，选项参数要放在 run 和 镜像名称 中间）：
# -d: 后台运行容器，并返回容器ID；
# -i: 以交互模式运行容器，通常与 -t 同时使用；
# -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
# -P: 随机端口映射，容器内部端口随机映射到主机的端口
# -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
# --name="nginx-lb": 为容器指定一个名称；
# -e username="ritchie": 设置环境变量；
# --volume , -v: 绑定一个卷
# 
# 使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。
docker run --name mynginx -d nginx:latest
# 使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。
docker run -P -d nginx:latest
# 使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。
docker run -p 80:80 -v /data:/data -d nginx:latest
# 绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。
docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
# 使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
docker run -it nginx:latest /bin/bash

# 查看容器日志
docker logs <容器名称|容器ID>

# 停止容器
docker stop <容器名称|容器ID>

# 删除容器，并且只能删除已经停止的容器
docker rm <容器名称|容器ID>
# 清除全部已停止的容器
docker container prune

```

```shell
# 命令规律总结
# docker run:
docker run +各种参数(-i -t -d -p -P -v --name) + 镜像名 + 创建容器后的操作

# -p参数
-p + 主机ip:port：容器port
eg.  docker run -p 127.0.0.1:80:8080/tcp ubantu bash

# -v参数
-v + 本地路径 : 容器路径
eg.  docker run -v /host/data:/volume/data ubantu bash


```



## Docker Volumn概念

- Docker Image 可以理解成多个只读文件的叠加，Docker Image是只读的
- 在将Image运行起来后，相当于在Image外包裹了一层读写层，变成了Container
- 当删除Container后，再次使用同个Image创建新的Container，此时的只读层还和以前一样，但读写层做过的修改全部丢失了
- 那么问题来了，如果想要持久化在读写层的操作 怎么办
- 主角登场，Docker Volumn实现数据的持久化。不仅如此，Volumn还能帮Container和Container之间，Container和Host之间共享数据。



## Kubernetes相关背景知识

### 编排与调度

- 在docker中，web容器和db容器之间通过建立“link”将db容器中的信息注入到web容器中，从而实现通信和交互，缺乏普适性，随着系统规模的扩大，容器间的“link”关系会愈发庞大复杂


- 以统一的方式定义容器间的各种关系，按照用户的意愿和系统的规则，完全自动化的处理好容器间的各种关系，此之谓“**编排**”


- （这里提一下比较熟悉的Yarn，Yarn的工作称为“**调度**”，它是聚焦于把一个容器按规则放置在一个最佳节点上去运行，重点强调最大化利用集群资源，而k8s重点在于保障集群的稳定和平）


- 容器的本质上是进程，故，对容器进行编排，就是对进程进行处理，因而也称k8s是“**云操作系统**”

  

### 如何处理容器间的关系

- #### 亲密型关系

  - 频繁交互、访问的一组容器，被划为一个“**Pod**”，在一个pod里可以进行高效信息交换
  - “**pod**”是k8s的最小调度单位

- #### 普通型关系

  - 容器间的普通访问，Kubernets通过定义“**服务对象**”对象来管理。一个service对象负责一系列操作。比如如web容器和db容器的交互，涉及到固定ip地址和端口，就产生了service对象来负责；加密授权等操作，则有“Secrit”对象负责。



### 如何恰当的容器化一个应用

- #### 编排对象

  - 为解决不同场景需求从而衍生出不同的解决方案，基于pod改进后的对象资源，成为编排对象。



### 总结：

- 由“**pod**”产生各类“**编排对象**”，再为解决各类实际问题从而产生Service、Ingress等“**服务对象**”
- 通过编排对象（Job，DaemonSet，Pod）来描述被管理的应用
- 通过服务对象（Service，Secret）来模拟平台对编排对象的管理
- 编排对象和服务对象都是API对象，声明式API，只需要描述期望达到的状态，无需关心底层逻辑是如何实现。



## Kubernets原理

- Kubernets关注状态。期望状态和实际状态。Kubernets对象是纽带，它反映着集群的实际状态，同时将用户的期望状态转达给kubernets，对象被控制器管理。具体的，转达过程如下

- 用户提交YAML文件到Kubernets API，控制器controller分析新期望状态和集群中的实际状态差异。

- 集群的预期状态定义了 运行哪些应用，运行哪些容器，使用哪些资源等等。

- 配置数据以及相关集群状态信息存储与etcd中。它是分布式的，高容错的

- Kubernets会自动管理集群，从而匹配用户期望的状态。

- 我们举个例子来说明 Kubernetes 如何管理预期状态。假设要部署一个预期状态为“3”的应用，这意味着要运行该应用的 3 个副本。

  如果这些容器中有 1 个发生崩溃，Kubernetes 副本集就会看到只有 2 个副本在运行，所以它会再添加 1 个副本以满足预期状态。

  副本集是一种控制器，可确保在特定时间运行指定数量的容器集。

  Kubernetes 部署是管理副本集的首选方式，可以向容器集提供声明性更新，不必自己来手动管理它们。 

  也可以使用 Kubernetes 中的自动扩展功能，来基于用户需求管理服务的扩展。在指定应用或服务的预期状态时，还可以让控制器随着需求增加而提供额外的容器集。

  例如，在繁忙时段，应用的预期状态可能会增加到 10 个副本，而不是平常的 3 个。



## Kubernetes个人总结v1.0

1.  k8s是一个容器编排工具，重点在编排，它的存在是符合客观发展规律的，当docker使用越来越广泛，服务器中的容器也就会越来越多，就面临着管理这些容器的任务，比如我们要部署一个web项目，需要先启动Mysql，再启动tomcat，最后是Ngix，如何去控制容器运行的顺序，k8s应运而生。
2.  k8s中的一个重要概念是pod。pod是k8s的最小单元，每个pod中含有一个或多个container。它就像小组长，寝室长，举一个生活中的例子，本科的时候当班长，老师要统计所有学生的信息，我挨个去问太麻烦了，就交给寝室长去做，我只负责对接寝室长。寝室长就是pod，寝室成员就像是多个container。
3.  k8s把container进一步抽象成pod。通过pod来管理几个相关联的container。
4.  k8s也是主从架构
5.  k8s类似yarn，都是为了管理，管理对象不同，yarn管理的是进程，k8s管理的是容器。
6.  pod只能通过k8s的集群内部ip地址访问，要想让它公开，需要把它暴露为service





## MiniKube



### 1.transwarp集群中配置本地minikube

- ### 安装kubectl

  - ```shell
    mkdir /jackey
    ```

  - ```shell
    curl -Lo kubectl "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

  - ```shell
    chmod +x kubectl
    ```

  - ```shell
    sudo mv kubectl /usr/local/bin/
    ```

- ### 安装minikube

  - ```shell
    curl -Lo minikube "https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.18.1/minikube-linux-amd64"
    ```

  - ```shell
    chmod +x minikube
    ```

  - ```shell
    sudo mv minikube /usr/local/bin/
    ```

- ### 启动minikube

  - ```shell
    minikube start \
      > --image-mirror-country=cn \
      > --registry-mirror='https://docker.mirrors.ustc.edu.cn/' \
      > --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' \
      > --vm-driver=docker \
      > --force
    ```


### 2.用Minikube运行一个应用

```shell
#使用 kubectl create 命令创建管理 Pod 的 Deployment。该 Pod 根据提供的 Docker 镜像运行 Container。

$ kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
deployment.apps/hello-node created

#查看 Deployment：

$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           13s

#查看 Pod：

$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-node-7567d9fdc9-brx28   1/1     Running   0          85s

# 查看集群事件：

$ kubectl get events
LAST SEEN   TYPE     REASON                    OBJECT                             MESSAGE
106s        Normal   Scheduled                 pod/hello-node-7567d9fdc9-brx28    Successfully assigned default/hello-node-7567d9fdc9-brx28 to minikube
104s        Normal   Pulling                   pod/hello-node-7567d9fdc9-brx28    Pulling image "k8s.gcr.io/echoserver:1.4"
104s        Normal   Pulled                    pod/hello-node-7567d9fdc9-brx28    Successfully pulled image "k8s.gcr.io/echoserver:1.4" in 552.292333ms
104s        Normal   Created                   pod/hello-node-7567d9fdc9-brx28    Created container echoserver
103s        Normal   Started                   pod/hello-node-7567d9fdc9-brx28    Started container echoserver
106s        Normal   SuccessfulCreate          replicaset/hello-node-7567d9fdc9   Created pod: hello-node-7567d9fdc9-brx28
106s        Normal   ScalingReplicaSet         deployment/hello-node              Scaled up replica set hello-node-7567d9fdc9 to 1
6m39s       Normal   NodeHasSufficientMemory   node/minikube                      Node minikube status is now: NodeHasSufficientMemory
6m39s       Normal   NodeHasNoDiskPressure     node/minikube                      Node minikube status is now: NodeHasNoDiskPressure
6m39s       Normal   NodeHasSufficientPID      node/minikube                      Node minikube status is now: NodeHasSufficientPID
6m14s       Normal   Starting                  node/minikube                      Starting kubelet.
6m14s       Normal   NodeHasSufficientMemory   node/minikube                      Node minikube status is now: NodeHasSufficientMemory
6m14s       Normal   NodeHasNoDiskPressure     node/minikube                      Node minikube status is now: NodeHasNoDiskPressure
6m14s       Normal   NodeHasSufficientPID      node/minikube                      Node minikube status is now: NodeHasSufficientPID
6m13s       Normal   NodeNotReady              node/minikube                      Node minikube status is now: NodeNotReady
6m13s       Normal   NodeAllocatableEnforced   node/minikube                      Updated Node Allocatable limit across pods
6m6s        Normal   RegisteredNode            node/minikube                      Node minikube event: Registered Node minikube in Controller
6m6s        Normal   NodeReady                 node/minikube                      Node minikube status is now: NodeReady
6m2s        Normal   Starting                  node/minikube                      Starting kube-proxy.

# 查看 kubectl 配置：

$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /root/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Wed, 17 Nov 2021 06:30:03 UTC
        provider: minikube.sigs.k8s.io
        version: v1.18.0
      name: cluster_info
    server: https://172.17.0.30:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Wed, 17 Nov 2021 06:30:03 UTC
        provider: minikube.sigs.k8s.io
        version: v1.18.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /root/.minikube/profiles/minikube/client.crt
    client-key: /root/.minikube/profiles/minikube/client.key
    
# 创建service，默认情况下，Pod 只能通过 Kubernetes 集群中的内部 IP 地址访问。 要使得 hello-node 容器可以从 Kubernetes 虚拟网络的外部访问，你必须将 Pod 暴露为 Kubernetes Service。使用 kubectl expose 命令将 Pod 暴露给公网：
$ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
service/hello-node exposed

# 查看你创建的 Service：
$ kubectl get services
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.97.152.31   <pending>     8080:30527/TCP   32s
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP          9m22s

# 使服务生效
$ minikube service hello-node
|-----------|------------|-------------|--------------------------|
| NAMESPACE |    NAME    | TARGET PORT |           URL            |
|-----------|------------|-------------|--------------------------|
| default   | hello-node |        8080 | http://172.17.0.30:30527 |
|-----------|------------|-------------|--------------------------|
* Opening service default/hello-node in default browser...
Minikube Dashboard is not supported via the interactive terminal experience.

Please click the 'Preview Port 30000' link above to access the dashboard.
This will now exit. Please continue with the rest of the tutorial.

X Exiting due to HOST_BROWSER: exit status 1
* 
* If the above advice does not help, please let us know: 
  - https://github.com/kubernetes/minikube/issues/new/choose

```







## kubernets初实践

```shell
# 1.准备tosdemo环境，方便隔离对象
# np-demo.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: tosdemo

# 2. 创建第一个对象
# first-pod-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: first-pod-nginx
  namespace: tosdemo
  # 标签，便于检索筛选
  labels:
    app: nginx
spec:
  containers:
  - name: nginx  
    # 指定端口
    image: nginx:alpine
    # 暴露指定端口
    ports:
    - containerPort: 80
    
# 3. 部署环境和对象
[root@ceph-02 gjk]# kubectl apply -f np-demo.yaml
namespace/tosdemo created

[root@ceph-02 gjk]# kubectl apply -f first-pod-nginx.yaml
pod/first-pod-nginx created

# 4. 验证部署
# 查看集群内 namespace
$ kubectl get namespace | grep tos  
# 查看集群内 tosdemo Namespace 下的 Pod 并宽行显示

[root@ceph-02 gjk]# kubectl get namespace
NAME                           STATUS   AGE
default                        Active   77d
harbor                         Active   75d
kbench-statefulset-namespace   Active   12d
kube-node-lease                Active   77d
kube-public                    Active   77d
kube-system                    Active   77d
mig-test                       Active   68d
monitor                        Active   76d
qospolicy                      Active   32d
tosdemo                        Active   86s

[root@ceph-02 gjk]# kubectl get pod -n tosdemo -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP             NODE      NOMINATED NODE   READINESS GATES
first-pod-nginx   1/1     Running   0          55s   10.224.0.217   ceph-02   <none>           <none>


```

### # 这里抛个锚点，去掉 -o wide后展示信息变少，盲猜去掉 -n tosdemo后会出现类似上面去掉 |grep tos的情况，显示出所有命名空间，但去掉后找不到刚刚创建的pod对象，推测不加-n 参数，默认会进入某个指定的命名空间，所以找不到first-nginx.pod



```shell
# 5.展开更多关于pod的描述信息
[root@ceph-02 gjk]# kubectl describe po first-pod-nginx -ntosdemo
Name:                 first-pod-nginx
Namespace:            tosdemo
Priority:             0
Priority Class Name:  low-priority
Node:                 ceph-02/172.16.0.12
Start Time:           Wed, 01 Dec 2021 11:31:24 +0800
Labels:               app=nginx
Annotations:          <none>
Status:               Running
IP:                   10.224.0.217
IPs:
  IP:  10.224.0.217
Containers:
  nginx:
    Container ID:   docker://2e9a3de1d9bd8659dc5a466ebc50e8070b2727e53f697e72b24f2960fbd6e378
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:12aa12ec4a8ca049537dd486044b966b0ba6cd8890c4c900ccb5e7e630e03df0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 01 Dec 2021 11:31:39 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-bmrrj (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-bmrrj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-bmrrj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 1800s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 1800s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  5m51s  default-scheduler  Successfully assigned tosdemo/first-pod-nginx to ceph-02
  Normal  Pulling    5m51s  kubelet            Pulling image "nginx:alpine"
  Normal  Pulled     5m37s  kubelet            Successfully pulled image "nginx:alpine" in 14.333159305s
  Normal  Created    5m37s  kubelet            Created container nginx
  Normal  Started    5m37s  kubelet            Started container nginx
```

​	



## Kubernets架构

- ![image-20211201142829698](C:\Users\dn\AppData\Roaming\Typora\typora-user-images\image-20211201142829698.png)

  

- 也是主从架构。和hadoop不一样的是，我们通过shell操作kubernets，我们的计算机相当于是node，master在遥远的云端。

- Master控制节点：

  - **kube-api-server**：操作资源的唯一入口，集群控制的入口进程，负责API服务
  - **kube-scheduler**：负责pod的调度
  - **kube-controller-manager**：负责容器编排，资源对象的自动化控制中心
  - **etcd**：整个集群的持久化数据。由kube-apiserver处理后保存在此。
  - ***<u>回顾：容器container是docker提出的概念，pod是k8s提出的概念，把紧密联系、交互的一些容器封装在一起。处理容器间的各种关系成为编排，也就是kube-controller-manager干的事；pod是k8s的最小调度资源，调度pod是kube-scheduler干的事。个人感觉controller-manager业务范围更广，它处理所有容器间的关系，它其中的一个子集是pod，kube-scheduler是对这个子集进行调度的。</u>***

- **Node**计算节点：

  - 运行**kubelet**：负责pod的创建、启停，实现集群管理的基本功能
  - **kube-proxy**:实现service的通信和负载均衡机制









## Pod深探

### a）Pod是什么

- Pod，**一组共享了某些资源的容器**。一个**逻辑概念**。
- Pod里面所有的容器，共享的是同一个 Network **Namespace**，共享一个Volume
- Pod类似于虚拟机，容器是虚拟机里运行的用户

### b）为什么需要Pod

- 容器的本质是进程，在操作系统中，进程并不是单打独斗的，而是有原则的组织在一起，紧密协作，形成**进程组**
- Kubernetes所作的，就是把**进程组**的概念迁移到**容器**技术中。一组容器间也会紧密协作，共同完成某个目标。
- 在调度时，这样的一组容器必须要放在一起，保证在用一个节点上，他们需要被作为一个**整体**来调度，因此，k8s将这样一组容器定义为**pod**，并把**pod**作为最小调度单位。

### c）Pod模型

- 在docker中，容器B和A共享网络和Volumn靠下面的指令

- ````shell
  docker run --net =B --volumns-from = B --name =A image -A ...
  ````

- 这样做的话，容器B就必须比A先启动，放在同一个pod里面，容器间就不是**对等关系**，而是**拓扑关系**了。就类似于没有充电头，只能用电脑给手机充电的时候，需要电脑先充上电，然后手机才能通过数据线插到电脑上充电，实际上用的都是墙上插头里面的电，但有个先后顺序

- 因此，Kubernetes的做法是：加一个插排。电脑接在插排上，插排的开关直接决定着电脑和手机能否充电。即：

- Pod 的实现需要使用一个中间容器，这个容器叫作 Infra 容器。在这个 Pod 中，Infra 容器永远都是第一个被创建的容器，而其他用户定义的容器，则通过 Join Network Namespace 的方式，与 Infra 容器关联在一起。这样的设计使得Pod的生命周期只跟 infra 容器有关，而与用户容器 A 和 B 无关。这也是为什么在 k8s 内，允许用户去单独更新 Pod 里的某一个镜像。即：做这个操作，整个 Pod 既不会重建，也不会重启。





## 容器编排控制器模式（Controller pattern）

### 提交一个YAML文件，K8s怎么把实际状态变成理想状态的？

### a）控制器模式概述

- ```Go
  for {
    实际状态 := 获取集群中对象 X 的实际状态（Actual State）
    期望状态 := 获取集群中对象 X 的期望状态（Desired State）
    if 实际状态 == 期望状态{
      什么都不做
    } else {
      执行编排动作，将实际状态调整为期望状态
    }
  }
  
  ```

- 实际状态指K8s集群本身的状态，期望状态指用户提交的YAML文件

- 通俗来讲，就是在这个死循环内，不停对比，调整

- 这个编排动作叫“调谐（Reconcile）”

- 这个过程叫**“Reconcile Loop”**

### b）动手实践，部署一个 Deployment 控制器。Deployment 会去找带有 “app: nginx” 标签的 Pod，然后统计它们的数量再去 “Replicas” 字段期望的副本数进行对比，有偏差再做调谐。	

- 1） 准备环境

  - 编写 nginx-deployment.yaml 文件

  - ```shell
    # nginx-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      namespace: tosdemo
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 2
      # 以上为控制面字段定义
      # ------------------------------------ # 
      # 以下是 被控制的对象 的模板字段信息
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:alpine
            ports:
            - containerPort: 80
    
    ```

- 2）部署deployment

  - ```shell
    [root@ceph-02 ~]# kubectl apply -f nginx-deployment.yaml
    deployment.apps/nginx-deployment created
    [root@ceph-02 ~]# kubectl get deployment -ntosdemo
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2/2     2            2           27s
    [root@ceph-02 ~]# kubectl get deploy -ntosdemo
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2/2     2            2           41s
    ```

- 3）验证deployment维持的副本数

  - ```shell
    # 先查看当前tosdemo空间内的所有pod
    [root@ceph-02 gjk]# kubectl get po -ntosdemo
    NAME                                READY   STATUS    RESTARTS   AGE
    first-pod-nginx                     1/1     Running   0          4h48m
    nginx-deployment-7fb7fd49b4-brvzs   1/1     Running   0          4m12s
    nginx-deployment-7fb7fd49b4-rpkhc   1/1     Running   0          4m12s
    
    # 删除某个nginx-deployment的副本
    [root@ceph-02 gjk]# kubectl delete po nginx-deployment-7fb7fd49b4-brvzs
    Error from server (NotFound): pods "nginx-deployment-7fb7fd49b4-brvzs" not found
    [root@ceph-02 gjk]# kubectl delete po nginx-deployment-7fb7fd49b4-brvzs -ntosdemo
    pod "nginx-deployment-7fb7fd49b4-brvzs" deleted
    
    # 再次查看，发现已经自动新生成了一个新副本（从NAME和AGE可以区分出来）
    [root@ceph-02 gjk]# kubectl get po -ntosdemo
    NAME                                READY   STATUS    RESTARTS   AGE
    first-pod-nginx                     1/1     Running   0          4h50m
    nginx-deployment-7fb7fd49b4-95f8p   1/1     Running   0          9s
    nginx-deployment-7fb7fd49b4-rpkhc   1/1     Running   0          6m8s
    [root@ceph-02 gjk]#
    ```





## 核心组件运行机制



- 我们以创建一个编排对象 ReplicaSet 来说明 Kubernetes 核心组件的运行机制。


- 这里提到的 ReplicaSet 属于 Deployment 的一个子集，Deployment 通过控制一个或多个 RS 来维持副本数。除此之外 Deployment 还支持滚动升级和回滚操作。
- ReplicaSet 负责通过“控制器模式”，保证系统中它所控制的 Pod 的个数永远等于指定的个数。
- 各组件协作工作流程如下图所示：
- ![image](D:\chrome下载\image.png)







​		



## 杂记

### 连上wifi，wifi是好的，但电脑打不开浏览器

- 解决方案：进入控制面板，Internet属性，点链接，局域网设置，三个框全取消，解决。



### windows10配置本地minikube环境

- 需要win10专业版OS

- 如果升级，需要手动安装hyper-v，在桌面上记事本新建一个Hyper-v.cmd 文件，内容如下，管理员身份运行它

- ```powershell
  pushd "%~dp0"
  dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
  for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
  del hyper-v.txt
  Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
  ```

  

- 开启hyper-v

- 安装kubectl（命令行工具）

- 安装minikube（官网下载就行）

- 启动minikube，会涉及到一些参数

  - `--driver=***` 从1.5.0版本开始，Minikube缺省使用系统优选的驱动来创建Kubernetes本地环境，比如您已经安装过Docker环境，minikube 将使用 `docker` 驱动
  - `--cpus=2`: 为minikube虚拟机分配CPU核数
  - `--memory=2048mb`: 为minikube虚拟机分配内存数
  - `--registry-mirror=***` 为了提升拉取Docker Hub镜像的稳定性，可以为 Docker daemon 配置镜像加速，参考[阿里云镜像服务](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
  - `--kubernetes-version=***`: minikube 虚拟机将使用的 kubernetes 版本

- 使用hyper-v 需要创建一个外部虚拟交换机

- 使用如下命令创建基于hyper-v的kubernets测试环境

- 

  ```shell
  .\minikube.exe start 
  	--image-mirror-country cn \
      --registry-mirror=https://xxxxxx.mirror.aliyuncs.com \
      --vm-driver="hyperv" \
      --hyperv-virtual-switch="MinikubeSwitch" \
      --memory=4096 
  ```

  

- 参考文章：

- https://developer.aliyun.com/article/221687



### ubantu配置本地minikube环境

- 在vmware虚拟机中安装了ubantu18操作系统

- vmware和windows之间复制粘贴：

  -  如果本身是ios驱动，换成自动检测，反之亦然，此时虚拟机选项下出现 安装vmware Tools
  -  点击后桌面上会出现VMware Tools DVD光驱，点击进入
  -  在桌面新建文件夹gjk，将.tar.gz后缀文件复制到gjk中（一定要复制出来，不然没法解压，会报no enogh free space,不知道为啥，离谱！）
  -  右键提取到当前文件夹，出现VMware Tools-10.3.....文件夹
  -  在文件夹中打开终端运行.pl文件

- linux下 apt换国内源

- 关于vmware中全屏黑边问题：

  - 更新阿里下载源

    ```shell
    sudo apt-get update
    ```

  - 安装open-vm-tools

    ```shell
    sudo apt-get install open-vm-tools
    ```

- 检查系统环境是否支持虚拟化技术

  - ```shell
    grep -E --color'vmx|svm' /proc/cpuinfo
    ```

- 安装VirtualBox

  - 在虚拟机设置中打开虚拟化引擎

  - ```shell
    sudo apt install virtualbox virtualbox-ext-pack
    ```

- 安装kubectl

- 安装minikube









