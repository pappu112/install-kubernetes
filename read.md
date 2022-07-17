Kubernetese Installation Guide Step by Step
--------------------------------------

Kubernetes streamlines application management by automating operational activities associated with container management and providing built-in commands for application deployment, rollout of updates, scaling up and down to accommodate changing requirements, monitoring, and more. 

## Step-1: CRI-O as the container runtime

Change settings for System requirements

```
cat /etc/sysctl.d/99-k8s-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-ip6tables=1
EOF
```
Karnel module for k8

```
echo -e overlay\\nbr_netfilter > /etc/modules-load.d/k8s.conf
```
## Step-2: Switch to iptable-legacy

```
 alternatives --config iptables
```
Enter to keep the current selection[+], or type selection number: 1 (select 1 to choose iptable legacy)

## Step-3: Set Swap Off Settings

```
touch /etc/systemd/zram-generator.conf
```
switch to Cgroup v1 (default is v2)

```
vi /etc/default/grub
```
line 7 : add below line
```
GRUB_CMDLINE_LINUX="systemd.unified_cgroup_hierarchy=0 rd.lvm.lv=fedora_fedora/root console=ttyS0,115200n8"
```
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## Step-4: Desable some settings
Now Desable Firewall

```
systemctl disable --now firewalld
```
disable [systemd-resolved] (enabled by default)
```
systemctl disable --now systemd-resolved
```
## Step-5: Edit Network Manager config file

```
vi /etc/NetworkManager/NetworkManager.conf
```
add into [main] section
[main]
dns=default

Now remove resolve.conf file & create a new resolve

```
unlink /etc/resolv.conf
```
create

```
touch /etc/resolv.conf
```
Now Restart your machine to apply changes

```
reboot
```

## Step-6: Install required packages

```
dnf module -y install cri-o:1.20/default
```

Enable container runtime enviroment

```
systemctl enable --now cri-o
```
## Step-7: Installing kubeadm, kubelet and kubectl

```
dnf -y install kubernetes-kubeadm kubernetes-node kubernetes-client cri-tools iproute-tc container-selinux
```

edit kubelet

```
vi /etc/kubernetes/kubelet
```
line 5 : change
```
KUBELET_ADDRESS="--address=0.0.0.0"
```
Line 8 : uncomment
```
KUBELET_PORT="--port=10250"
```
line 11 : change to your hostname
```
KUBELET_HOSTNAME="--hostname-override=<hostname>"
```
change in kubeadm.conf
```
vi /etc/systemd/system/kubelet.service.d/kubeadm.conf
```
line 7 : add
```
Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=systemd --container-runtime=remote --container-runtime-endpoint=unix:///var/run/crio/crio.sock"
```
## step-8: init cluster

```
kubeadm init
```
then you have to configure the below massage on the terminal


  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

## Step-9: Check nodes and pods

to check the node and its status

```
kubectl get node
```
check pods

```
kubectl get pod
```
Now You will see your node is Ready that means kubernetes installed sucessfully on your fedora 35.
