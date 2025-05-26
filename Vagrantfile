# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

Vagrant.configure("2") do |config|

  $k3s_num_instances ||= 0
  (1..$k3s_num_instances).each do |i|
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
      config.vm.synced_folder '.', '/vagrant', disabled: true
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
    config.vm.synced_folder '.', '/vagrant', disabled: true
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "environments/local/playfile/bootstrap.yml"
      ansible.limit    = "all"
      ansible.groups = {
        "k3s"      => ["k3s-master01", "k3s-node0[1:#{$k3s_num_instances}]"],
        "k3s:vars" => {
          "bootstrap_disable_firewalld" => "true",
          "bootstrap_disable_selinux"   => "true",
          "bootstrap_install_docker"    => "true",
          "bootstrap_install_k3s"       => "true",
        }
      }
      ansible.host_vars = {
        "k3s-master01" => {
          "bootstrap_k3s_server_role" => "master",
        },
      }
    end
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "environments/local/playfile/awx-setup.yml"  
    end
  end

end
