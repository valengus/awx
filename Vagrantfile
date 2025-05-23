# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

$prepare= <<-SCRIPT
echo 'root:root' | sudo chpasswd
dnf install -y tar git make nano curl bash-completion sshpass dnf-plugins-core
systemctl disable firewalld --now
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config ; setenforce 0
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce docker-ce-cli containerd.io
systemctl enable docker --now
SCRIPT

Vagrant.configure("2") do |config|

  $num_instances ||= 1
  (1..$num_instances).each do |i|
    config.vm.define "k3s-node0#{i}" do |config|
      config.vm.hostname = "k3s-node0#{i}"
      config.vm.box      = "generic/oracle9"
      config.vm.provider :libvirt do |libvirt|
        libvirt.cpus                      = 2
        libvirt.memory                    = 4 * 1024
        libvirt.qemu_use_session          = false
        libvirt.default_prefix            = 'vagrant_'
        libvirt.management_network_name   = 'vagrant'
        libvirt.management_network_domain = 'local'
      end
      config.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_version: 4, nfs_udp: false
      config.vm.provision "shell", inline: $prepare
    end
  end

  config.vm.define "k3s-master01" do |config|
    config.vm.hostname = "k3s-master01"
    config.vm.box      = "generic/oracle9"
    config.vm.provider :libvirt do |libvirt|
      libvirt.driver                     = "kvm"
      libvirt.qemu_use_session           = false
      libvirt.cpus                       = 2
      libvirt.memory                     = 4 * 1024
      libvirt.default_prefix             = 'vagrant_'
      libvirt.management_network_name    = 'vagrant'
      libvirt.management_network_domain  = 'local'
    end
    config.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_version: 4, nfs_udp: false
    config.vm.provision "shell", inline: $prepare
    config.vm.provision "shell", inline: <<-SHELL
    curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE='644' sh -s - --docker --disable=traefik
    echo 'source <(kubectl completion bash)' >> /home/vagrant/.bashrc
    curl -sSLo - https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v5.0.1/kustomize_v5.0.1_linux_amd64.tar.gz | tar xzf - -C /bin/
    SHELL
    (1..$num_instances).each do |i|
      config.vm.provision "shell", inline: <<-SHELL
      NODETOKEN=$(cat /var/lib/rancher/k3s/server/node-token)
      sshpass -p "root" ssh -o "StrictHostKeyChecking no" root@k3s-node0#{i} "curl -sfL https://get.k3s.io | K3S_URL=https://k3s-master01:6443 K3S_TOKEN=$NODETOKEN sh -s - --docker "
      SHELL
    end
  end

  config.vm.define "oracle9" do |config|
    config.vm.hostname = "oracle9"
    config.vm.box      = "generic/oracle9"
    config.vm.provider :libvirt do |libvirt|
      libvirt.cpus                      = 2
      libvirt.memory                    = 4 * 1024
      libvirt.qemu_use_session          = false
      libvirt.default_prefix            = 'vagrant_'
      libvirt.management_network_name   = 'vagrant'
      libvirt.management_network_domain = 'local'
    end
    config.vm.synced_folder '.', '/vagrant', disabled: true
  end

end
