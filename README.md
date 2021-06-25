#k8s流程

###一、安装所需配置及环境
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
