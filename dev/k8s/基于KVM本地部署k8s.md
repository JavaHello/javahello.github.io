# 基于 KVM 本地部署 K8S

## 安装 KVM 虚拟机相关软件(物理机)

sudo apt install qemu-kvm virt-manager libvirt-daemon-system virtinst libvirt-clients bridge-utils

## 创建 k8s-master 虚拟机(物理机)

1. 下载操作系统文件`ubuntu-22.04.1-live-server-amd64.iso`,
1. 创建虚拟机

   ```sh
   # virt 这里使用 root 用户
   virsh net-start default # 启用默认网络
   virt-install --virt-type kvm \
   --name k8s-master \
   --vcpus 4 --memory 8096 \
   --disk path=/opt/vms/k8s-master.img,bus=virtio,size=200 \
   --network network=default,model=virtio \
   --os-variant=ubuntu22.04 \
   --cdrom ubuntu-22.04.1-live-server-amd64.iso
   ```

1. 使用 virt-manager 图像化界面安装
1. 启用控制台连接`systemctl enable serial-getty@ttyS0.service`

### 配置基本环境(k8s-master 虚拟机)

1. 安装基本依赖
   [k8s 官网步骤](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

   ```sh
   # 更新 apt 包索引并安装使用 Kubernetes apt 仓库所需要的包
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl

   # 下载 Google Cloud 公开签名秘钥：
   sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

   # 添加 Kubernetes apt 仓库
   echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

   # 更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

1. 必须项配置

   ```sh
   swapoff -a # 禁用 swap
   vim /etc/fstab # 注释 swap 开头的行
   # 转发 IPv4 并让 iptables 看到桥接流量
   cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
   overlay
   br_netfilter
   EOF

   sudo modprobe overlay
   sudo modprobe br_netfilter

   # 设置所需的 sysctl 参数，参数在重新启动后保持不变
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-iptables  = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   net.ipv4.ip_forward                 = 1
   EOF

   # 应用 sysctl 参数而不重新启动
   sudo sysctl --system

   # 通过运行以下指令确认 br_netfilter 和 overlay 模块被加载：
   lsmod | grep br_netfilter
   lsmod | grep overlay

   # 通过运行以下指令确认 net.bridge.bridge-nf-call-iptables、net.bridge.bridge-nf-call-ip6tables 和 net.ipv4.ip_forward 系统变量在你的 sysctl 配置中被设置为 1：
   sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
   ```

1. 容器配置
   `vim /etc/containerd/config.toml`

   ```txt
   # 配置 systemd cgroup 驱动
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
     SystemdCgroup = true
   # 重载沙箱（pause）镜像, 这个和需要 kubeadm init 指定的一样
   [plugins."io.containerd.grpc.v1.cri"]
     sandbox_image = "registry.k8s.io/pause:3.2"

   # 使用国内镜像，目前国内网络必须配置
          [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://xxxxx.mirror.aliyuncs.com", "https://registry-1.docker.io"]

   # 或启用代理
   # /lib/systemd/system/containerd.service
   [Service]
   Environment="HTTP_PROXY=http://192.168.50.165:1087"
   Environment="HTTPS_PROXY=http://192.168.50.165:1087"
   Environment="NO_PROXY=localhost,127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.svc,.cluster.local,.ewhisper.cn"
   ```

   重启`systemctl restart containerd`

## clone 虚拟机(物理机)

1. clone 两个节点 k8s-node1, k8s-node2

   ```sh
   virt-clone \
   --original k8s-master \
   --name k8s-node1 \
   --file /opt/vms/k8s-node1.img
   virt-clone \
   --original k8s-master \
   --name k8s-node2 \
   --file /opt/vms/k8s-node2.img
   ```

## 网络配置(物理机)

1. 启动虚拟机查看`mac`地址并记住
   ```sh
   # k8s-master
   cat /sys/class/net/enp1s0/address
   # k8s-node1
   cat /sys/class/net/enp1s0/address
   # k8s-node2
   cat /sys/class/net/enp1s0/address
   ```
1. 关闭虚拟机配置默认网络`virsh net-edit default`
   ```txt
   <dhcp>
     <range start='192.168.122.2' end='192.168.122.254'/>
     <host mac='52:54:00:72:72:b1' name='k8s-master' ip='192.168.122.100'/>
     <host mac='52:54:00:72:72:b2' name='k8s-node1' ip='192.168.122.101'/>
     <host mac='52:54:00:72:72:b3' name='k8s-node2' ip='192.168.122.102'/>
   </dhcp>
   ```
   重启网络`virsh net-destroy default` `virsh net-start default`

## 初始化集群(虚拟机)

1. 启动 3 台虚拟机
1. 进入 k8s-master `virsh console k8s-master`
1. 初始化 k8s-master
   ```sh
   export KUBECONFIG=/etc/kubernetes/admin.conf
   # --image-repository=registry.aliyuncs.com/google_containers 使用国内镜像，目前国内网络必须配置
   kubeadm init --pod-network-cidr=10.244.0.0/16 --image-repository=registry.aliyuncs.com/google_containers
   ```
1. 网络插件配置，这里使用的[flannel](https://github.com/flannel-io/flannel)

   ```txt
   kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
   # 如果 podCIDR 不是 10.244.0.0/16 需要修改配置文件
   # 和 kubeadm init --pod-network-cidr=10.244.0.0/16 保持一致
   ```

1. 查看状态`kubectl get nodes`
   ```txt
   NAME         STATUS   ROLES           AGE   VERSION
   k8s-master   Ready    control-plane   1m   v1.26.1
   ```
1. 进入`k8s-node1`, `k8s-node2` `kubeadm join`
   ```sh
   # k8s-node1, k8s-node2
   kubeadm join 192.168.122.100:6443 --token xxx \
       --discovery-token-ca-cert-hash xxx
   ```
1. 查看状态`kubectl get nodes`
   ```txt
   NAME         STATUS   ROLES           AGE   VERSION
   k8s-master   Ready    control-plane   81m   v1.26.1
   k8s-node1    Ready    <none>          67m   v1.26.1
   k8s-node2    Ready    <none>          50m   v1.26.1
   ```
