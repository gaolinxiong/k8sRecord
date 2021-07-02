#k8s流程

### 一、安装所需配置及环境
- ubuntu 16.04虚拟机安装（不做过多讲解）
先关闭防火墙
```
 sudo swapoff -a
```

- 国内，用阿里云源安装就可以了，速度很快：

```
apt-get update && apt-get install -y apt-transport-https

curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -

vim /etc/apt/sources.list.d/kubernetes.list

/* 将下面一行加入到vim的文件中，没有就创建 */
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main

apt-get update

apt-get install kubernetes-cni=0.6.0-00

apt remove kubelet kubectl kubeadm
apt install kubelet=1.11.3-00
apt install kubectl=1.11.3-00
apt install kubeadm=1.11.3-00

```
- 删除现有docker
```
// 删除某软件,及其安装时自动安装的所有包
sudo apt-get autoremove docker docker-ce docker-engine  docker.io  containerd runc

// 删除docker其他没有没有卸载
dpkg -l | grep docker

dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P # 删除无用的相关的配置文件

// 卸载没有删除的docker相关插件(结合自己电脑的实际情况)
sudo apt-get autoremove docker-ce-*

// 删除docker的相关配置&目录
sudo rm -rf /etc/systemd/system/docker.service.d
sudo rm -rf /var/lib/docker

// 确定docker卸载完毕
docker --version
```
- 安装匹配版本的docker
（docker菜鸟教程： https://www.runoob.com/docker/ubuntu-docker-install.html）
```
// 更新 apt 包索引。
apt-get update

// 安装 apt 依赖包，用于通过HTTPS来获取仓库:
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

// 添加 Docker 的官方 GPG 密钥：
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

// 验证您现在是否拥有带有指纹的密钥。
sudo apt-key fingerprint 0EBFCD88

// 使用以下指令设置稳定版仓库
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) \
  stable"

// 更新 apt 包索引。
sudo apt-get update

// 要安装特定版本的 Docker Engine-Community，请在仓库中列出可用版本，然后选择一种安装。列出您的仓库中可用的版本：
apt-cache madison docker-ce

安装想要的指定版本 在这里适配的是17.03.0~ce-0~ubuntu-xenial
sudo apt-get install docker-ce=17.03.0~ce-0~ubuntu-xenial containerd.io

```

## 二、下面开始了正确部署(kubeadm.yaml配置文件)

```
### kubeadm.yaml

apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
controllerManagerExtraArgs:
  horizontal-pod-autoscaler-use-rest-clients: "true"
  horizontal-pod-autoscaler-sync-period: "10s"
  node-monitor-grace-period: "10s"
apiServerExtraArgs:
  runtime-config: "api/all=true"
kubernetesVersion: "v1.11.3"
```
- 在此过程中部分内容要求外网环境，我们需要在此进行换源来配置（如下）
##### （docker.io仓库对google的容器做了镜像，可以通过下列命令下拉取相关镜像：）
```
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.11.3
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.11.3
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.11.3
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.11.3
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd-amd64:3.2.18
docker pull coredns/coredns:1.1.3
```
##### 版本信息需要根据实际情况进行相应的修改。通过docker tag命令来修改镜像的标签：
```
docker tag docker.io/mirrorgooglecontainers/kube-proxy-amd64:v1.11.3 k8s.gcr.io/kube-proxy-amd64:v1.11.3
docker tag docker.io/mirrorgooglecontainers/kube-scheduler-amd64:v1.11.3 k8s.gcr.io/kube-scheduler-amd64:v1.11.3
docker tag docker.io/mirrorgooglecontainers/kube-apiserver-amd64:v1.11.3 k8s.gcr.io/kube-apiserver-amd64:v1.11.3
docker tag docker.io/mirrorgooglecontainers/kube-controller-manager-amd64:v1.11.3 k8s.gcr.io/kube-controller-manager-amd64:v1.11.3
docker tag docker.io/mirrorgooglecontainers/etcd-amd64:3.2.18  k8s.gcr.io/etcd-amd64:3.2.18
docker tag docker.io/mirrorgooglecontainers/pause:3.1  k8s.gcr.io/pause:3.1
docker tag docker.io/coredns/coredns:1.1.3  k8s.gcr.io/coredns:1.1.3
```
##### 使用docker rmi删除不用镜像，通过docker images命令显示，已经有我们需要的镜像文件，可以继续部署工作了：
```
docker images
-----------
k8s.gcr.io/kube-proxy-amd64                                              v1.11.3             be5a6e1ecfa6        10 days ago         97.8 MB
k8s.gcr.io/kube-scheduler-amd64                                          v1.11.3             ca1f38854f74        10 days ago         56.8 MB
k8s.gcr.io/kube-apiserver-amd64                                          v1.11.3             3de571b6587b        10 days ago         187 MB
coredns/coredns                                                          1.1.3               b3b94275d97c        3 months ago        45.6 MB
k8s.gcr.io/coredns                                                       1.1.3               b3b94275d97c        3 months ago        45.6 MB
k8s.gcr.io/etcd-amd64                                                    3.2.18              b8df3b177be2        5 months ago        219 MB
k8s.gcr.io/pause
```
                             
## 三、执行kubeadm.yaml文件即可
```
kubeadm init --config kubeadm.yaml
```

## 四、你会得到如下命令(根据提示执行即可)
```
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.3.164:6443 --token 46ncz3.a293cm8jnitg1q50 --discovery-token-ca-cert-hash sha256:1a3ab5a5bf1503dc219c69dd2d799043d646879ff2c5b1805ecccac03f295edd

```
## 五、如果遇到问题 就直接 (kubeadm reset) 大不了推到重来嘛

## 六、！！！一定要记得更改主机名！！！
```
hostname k8s-master

hostname k8s-node1
```

### 七、在此后可通过命令查看当前 k8s-master的机器状态
```
kubectl describe node k8s-master

#下面会得到这样的内容 ------------

Name:               k8s-master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=k8s-master
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket=/var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
CreationTimestamp:  Thu, 01 Jul 2021 05:06:42 -0700
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
Conditions:
  Type                 Status    LastHeartbeatTime                 LastTransitionTime                Reason                    Message
  ----                 ------    -----------------                 ------------------                ------                    -------
  NetworkUnavailable   False     Thu, 01 Jul 2021 06:02:38 -0700   Thu, 01 Jul 2021 06:02:38 -0700   WeaveIsUp                 Weave pod has set this
  OutOfDisk            Unknown   Thu, 01 Jul 2021 06:02:54 -0700   Thu, 01 Jul 2021 06:02:46 -0700   NodeStatusUnknown         Kubelet stopped posting node status.
  MemoryPressure       Unknown   Thu, 01 Jul 2021 06:02:54 -0700   Thu, 01 Jul 2021 06:02:46 -0700   NodeStatusUnknown         Kubelet stopped posting node status.
  DiskPressure         Unknown   Thu, 01 Jul 2021 06:02:54 -0700   Thu, 01 Jul 2021 06:02:46 -0700   NodeStatusUnknown         Kubelet stopped posting node status.
  PIDPressure          False     Thu, 01 Jul 2021 06:02:54 -0700   Thu, 01 Jul 2021 05:06:42 -0700   KubeletHasSufficientPID   kubelet has sufficient PID available
  Ready                Unknown   Thu, 01 Jul 2021 06:02:54 -0700   Thu, 01 Jul 2021 06:02:46 -0700   NodeStatusUnknown         Kubelet stopped posting node status.
Addresses:
  InternalIP:  192.168.3.164
  Hostname:    k8s-master
Capacity:
 cpu:                4
 ephemeral-storage:  102094168Ki
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             8144672Ki
 pods:               110
Allocatable:
 cpu:                4
 ephemeral-storage:  94089985074
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             8042272Ki
 pods:               110
System Info:
 Machine ID:                 66b178f461e84cbc9d57839bd5534487
 System UUID:                42F84D56-27A7-C68B-E187-8CF30ADB8225
 Boot ID:                    98e05520-be34-45ee-84e4-246747b71999
 Kernel Version:             4.15.0-142-generic
 OS Image:                   Ubuntu 16.04.7 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://17.3.0
 Kubelet Version:            v1.11.3
 Kube-Proxy Version:         v1.11.3
Non-terminated Pods:         (8 in total)
  Namespace                  Name                                  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                                  ------------  ----------  ---------------  -------------
  kube-system                coredns-78fcdf6894-f6cbr              100m (2%)     0 (0%)      70Mi (0%)        170Mi (2%)
  kube-system                coredns-78fcdf6894-jtl6c              100m (2%)     0 (0%)      70Mi (0%)        170Mi (2%)
  kube-system                etcd-k8s-master                       0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-apiserver-k8s-master             250m (6%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-controller-manager-k8s-master    200m (5%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-proxy-2m7w5                      0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-scheduler-k8s-master             100m (2%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                weave-net-58tbn                       100m (2%)     0 (0%)      200Mi (2%)       0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource  Requests    Limits
  --------  --------    ------
  cpu       850m (21%)  0 (0%)
  memory    340Mi (4%)  340Mi (4%)
Events:
  Type    Reason                   Age                 From                    Message
  ----    ------                   ----                ----                    -------
  Normal  Starting                 57m                 kubelet, k8s-master     Starting kubelet.
  Normal  NodeAllocatableEnforced  56m                 kubelet, k8s-master     Updated Node Allocatable limit across pods
  Normal  NodeHasSufficientPID     56m (x5 over 56m)   kubelet, k8s-master     Node k8s-master status is now: NodeHasSufficientPID
  Normal  Starting                 56m                 kube-proxy, k8s-master  Starting kube-proxy.
  Normal  NodeHasSufficientDisk    22m (x67 over 56m)  kubelet, k8s-master     Node k8s-master status is now: NodeHasSufficientDisk
  Normal  NodeHasSufficientMemory  16m (x75 over 56m)  kubelet, k8s-master     Node k8s-master status is now: NodeHasSufficientMemory
  Normal  NodeNotReady             6m (x74 over 55m)   kubelet, k8s-master     Node k8s-master status is now: NodeNotReady
  Normal  NodeHasNoDiskPressure    2m (x97 over 56m)   kubelet, k8s-master     Node k8s-master status is now: NodeHasNoDiskPressure


```
## 八、另外,我们还可以通过 kubectl 检查这个节点上各个系统 Pod 的状态
```
kubectl get pods -n kube-system

得到如下内容---------
coredns-78fcdf6894-f6cbr             0/1       Pending   0          37m
coredns-78fcdf6894-jtl6c             0/1       Pending   0          37m
etcd-k8s-master                      1/1       Running   0          37m
kube-apiserver-k8s-master            1/1       Running   0          37m
kube-controller-manager-k8s-master   1/1       Running   0          37m
kube-proxy-2m7w5                     1/1       Running   0          37m
kube-scheduler-k8s-master            1/1       Running   0          37m
```
- 我们会发现其中头两个pod竟然为0/1 通过查找当前master节点状态可知 是没有开通网络导致的 需要执行下面一行命令
```
kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
- 完成后等待2分钟 重新检查一下
```
kubectl get pods -n kube-system

coredns-78fcdf6894-f6cbr             1/1       Running   0          1h
coredns-78fcdf6894-jtl6c             1/1       Running   0          1h
etcd-k8s-master                      1/1       Running   0          1h
kube-apiserver-k8s-master            1/1       Running   0          1h
kube-controller-manager-k8s-master   1/1       Running   0          1h
kube-proxy-2m7w5                     1/1       Running   0          1h
kube-scheduler-k8s-master            1/1       Running   0          1h
weave-net-58tbn                      2/2       Running   1          8m
```

## 九、下面来安装可视化k8s
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc6/aio/deploy/recommended.yaml

kubectl proxy

## 下面这条命令获取kubernetes token

kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token


Name:         namespace-controller-token-56gcv
Type:  kubernetes.io/service-account-token
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi01NmdjdiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImNkZGVmZDhjLWRhNjQtMTFlYi05ZGIzLTAwMGMyOWRiODIyNSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.ZNHBx5N5EzQwg9R2WIi6J0m2x7kAvLKuCuQReAKWvTbIfsDNMmxJmzgIcYOJksLRAY1HSQalcJhSwMKX7uvl9YDCCVTBvUR7Hi_EndzVcDl6x_CgEm68Rzf8LasDszh1NVC0XnMSyUg29Bi9qplrrIQOaemjFt-xsuO-jz7hgexJ10f77g_4Q50wZ38TXMYpNFVpGepR9DB14swugn9WiO79U5Osd1XG9G3PkNjAWsrfTx584zr3kItNogwxIQfXfS-6yqa1NAc_qSRJVtML9AcCyej2vrFrAlDCP4bLDjWN-eCnNVwFf1MEfUbxZY7_KYFEBGQQJKpHzTEZjyDj3g

```
## 十、为kubenetes的node节点设置 roles
```
# k8s-node1为当前节点名称 worker为你想设置的值

kubectl label node k8s-node1  node-role.kubernetes.io/worker=
```

## 十一、当看到kube-proxy有问题或者关于下面节点有问题时 可以采用如下命令查看pods情况，迅速找到问题原因
```
# 先通过此命令查看一下
kubectl get pod --namespace=kube-system

NAME                                 READY     STATUS              RESTARTS   AGE
coredns-78fcdf6894-dzq5m             0/1       Pending             0          9m
coredns-78fcdf6894-tfhxd             0/1       Pending             0          9m
etcd-k8s-master                      1/1       Running             0          7m
kube-apiserver-k8s-master            1/1       Running             1          9m
kube-controller-manager-k8s-master   1/1       Running             0          8m
kube-proxy-m69ws                     1/1       Running             0          9m
kube-proxy-n4q6r                     0/1       ContainerCreating   0          6m
kube-scheduler-k8s-master            1/1       Running             0          8m

# 发现 kube-proxy-n4q6r 有问题，则直接查找关于此pod的问题
kubectl describe pod kube-proxy-n4q6r --namespace=kube-system

Name:               kube-proxy-n4q6r
Namespace:          kube-system
Priority:           2000001000
PriorityClassName:  system-node-critical
Node:               k8s-node1/192.168.2.133
Start Time:         Thu, 01 Jul 2021 23:24:22 -0700
Labels:             controller-revision-hash=458499861
                    k8s-app=kube-proxy
                    pod-template-generation=1
Annotations:        scheduler.alpha.kubernetes.io/critical-pod=
Status:             Pending
IP:                 192.168.2.133
Controlled By:      DaemonSet/kube-proxy
Containers:
  kube-proxy:
    Container ID:
    Image:         k8s.gcr.io/kube-proxy-amd64:v1.11.3
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
      /var/lib/kube-proxy from kube-proxy (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-proxy-token-9sn4v (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-proxy:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-proxy
    Optional:  false
  xtables-lock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/xtables.lock
    HostPathType:  FileOrCreate
  lib-modules:
    Type:          HostPath (bare host directory volume)
    Path:          /lib/modules
    HostPathType:
  kube-proxy-token-9sn4v:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  kube-proxy-token-9sn4v
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  beta.kubernetes.io/arch=amd64
Tolerations:
                 CriticalAddonsOnly
                 node.kubernetes.io/disk-pressure:NoSchedule
                 node.kubernetes.io/memory-pressure:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute
                 node.kubernetes.io/unreachable:NoExecute
Events:
  Type     Reason                  Age               From                Message
  ----     ------                  ----              ----                -------
  Warning  FailedCreatePodSandBox  13s (x7 over 6m)  kubelet, k8s-node1  Failed create pod sandbox: rpc error: code = Unknown desc = failed pulling image "k8s.gcr.io/pause:3.1": Error response from daemon: Get https://k8s.gcr.io/v1/_ping: dial tcp 108.177.97.82:443: i/o timeout

# 通过查看原因 可知 是因为国内网络不能直接拉去pause:3.1
# 所以我们的解决办法是 在docker上拉去 aliyun的 hangzhou 包源
# （registry.cn-hangzhou.aliyuncs.com/google_containers）这个包源是真的稳啊～

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
```
## 十二、部署容器存储插件
```
最新的部署ceph cluster需要多一条部署命令才可以部署成功：
1. kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/common.yaml
2. kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml
3. kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml

```
