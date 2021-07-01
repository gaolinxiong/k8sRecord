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

  kubeadm join 192.168.3.164:6443 --token 28vla6.n9rno3fus3djd0vh --discovery-token-ca-cert-hash sha256:32557bdfb07766da42c166df056009435f2dec97d87a38b0bd3abe1757f41c03

```
