# install docker 
$install_docker_script = <<-SCRIPT
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
SCRIPT

# add kubernetes repo to system
# for install kubelet, kuber, kubeadm, kubectl 
$init_yum_repo = <<-SCRIPT
sudo cat > /etc/yum.repos.d/kuber.repo << EOF
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
enabled=1
EOF

yum install -y kubelet kubeadm kubectl
SCRIPT

$init_environment = <<-SCRIPT
sudo timedatectl set-timezone Asia/Shanghai
sudo systemctl start chronyd

sudo setenforce 0
sudo cat > /etc/selinux/config << EOF
SELINUX=disabled
SELINUXTYPE=targeted
EOF

sudo systemctl stop firewalld
sudo systemctl disable firewalld

sudo cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo modprobe br_netfilter
sudo sysctl -p /etc/sysctl.d/k8s.conf

yum install -y --setopt=obsoletes=0 docker-ce-18.09.4-3.el7
sudo systemctl enable docker
sudo systemctl start docker

sudo mkdir /etc/docker
sudo touch /etc/docker/daemon.json
sudo cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://kebrbym9.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.provision "shell", inline: $install_docker_script
  config.vm.provision "shell", inline: $init_yum_repo
  config.vm.provision "shell", inline: $init_environment

  # set the virtualbox machine
  # the kubernetes require 2 cpus
  config.vm.provider :libvirt do |lv|
    lv.memory = 2048
    lv.cpus = 4
  end

  # auto set the host in every machines
  # use vagrant-hosts plugin
  config.vm.provision :hosts, sync_hosts: true

  config.vm.define "master" do |master|
    master.vm.network "private_network", ip: "192.168.56.101"
  end

  config.vm.define "node1" do |node1|
    node1.vm.network "private_network", ip: "192.168.56.102"
  end

  config.vm.define "node2" do |node2|
    node2.vm.network "private_network", ip: "192.168.56.103"
  end
end
