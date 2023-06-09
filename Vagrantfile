Vagrant.require_version ">= 2.0.0"

boxes = [
    {
        :name => "k8s-master",
        :eth1 => "192.168.205.100",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-worker1",
        :eth1 => "192.168.205.101",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "k8s-worker2",
        :eth1 => "192.168.205.102",
        :mem => "1024",
        :cpu => "1"
#    },
#    {
#        :name => "k8s-master2",
#        :eth1 => "192.168.206.100",
#        :mem => "2048",
#        :cpu => "1"
#    },
#    {
#        :name => "k8s-worker21",
#        :eth1 => "192.168.206.101",
#        :mem => "1024",
#        :cpu => "1"
    }
]

Vagrant.configure(2) do |config|
  config.vm.box = "generic/ubuntu2004"

  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]
        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
          v.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          v.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
        end
        config.vm.network :private_network, ip: opts[:eth1]
      end
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo modprobe overlay
    sudo modprobe br_netfilter
    sudo echo 'overlay' > /etc/modules-load.d/containerd.conf
    sudo echo 'br_netfilter' >> /etc/modules-load.d/containerd.conf
    sudo echo 'net.bridge.bridge-nf-call-iptables = 1' > /etc/sysctl.d/kubernetes.conf
    sudo echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.d/kubernetes.conf
    sudo echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.d/kubernetes.conf
    sudo sysctl --system
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
    sudo mv ~/kubernetes.list /etc/apt/sources.list.d
    sudo apt update
    echo "KUBELET_EXTRA_ARGS=--node-ip="$(ip addr show eth1  | awk '$1 == "inet" { print $2 }' | cut -d/ -f1) | sudo tee /etc/default/kubelet
    sudo apt install -y containerd.io
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
    sudo systemctl restart containerd

    # Install etcdctl
    export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest|grep tag_name | cut -d '"' -f 4)
    wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz
    tar xvf etcd-${RELEASE}-linux-amd64.tar.gz
    cd etcd-${RELEASE}-linux-amd64
    sudo mv etcd etcdctl etcdutl /usr/local/bin 

    sudo apt install -y kubelet kubeadm kubectl
    sudo swapoff -a
    sudo sed -i '/ swap / s/^/#/' /etc/fstab

  SHELL

end
