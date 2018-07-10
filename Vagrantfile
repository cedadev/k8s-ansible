# -*- mode: ruby -*-
# vi: set ft=ruby :

N_MINIONS = 2

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"

  # Allow root login with same key as vagrant user
  config.vm.provision :shell, inline: <<-SHELL
    mkdir -p /root/.ssh
    cp ~vagrant/.ssh/authorized_keys /root/.ssh
  SHELL

  # Due to a Vagrant bug, eth1 stays down on boot on CentOS 7
  #   Here, we make sure it starts on each boot
  #   See https://github.com/mitchellh/vagrant/issues/8166
  config.vm.provision :shell, run: "always", inline: "ifup eth1"

  # Make sure Kubernetes service traffic is routed over eth1
  # This is only necessary because eth1 is not the default interface
  # We can't just make eth1 the default interface because it won't route traffic to the internet
  config.vm.provision :shell, inline: <<-SHELL
    my_ip="$(ip addr show eth1 | grep "inet " | awk '{ print $2; }' | cut -d "/" -f 1)"
    echo "10.96.0.0/12 via ${my_ip} dev eth1" > /etc/sysconfig/network-scripts/route-eth1
    systemctl restart network
  SHELL

  # Install python-netaddr
  # This is required for the ipaddr filter, but must be done *before* the playbooks are run
  config.vm.provision :shell, inline: "yum install -y python-netaddr"
    
  # Give each host 2 CPUs and 2GB RAM
  config.vm.provider :virtualbox do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end

  # Use a pre-defined token
  kubeadm_token = "51mhbs.oc6k36jrreq8dy6n"

  config.vm.define "kube-master" do |master|
    master.vm.hostname = "kube-master"
    master.vm.network :private_network, ip: "172.28.128.100"

    master.vm.provision :ansible_local do |ansible|
      ansible.playbook = "kubemaster.yml"
      ansible.become = true
      ansible.extra_vars = {
        "host_ip" => "172.28.128.100",
        "cluster_cidr" => "172.28.128.0/24",
        "userspace_proxy" => true,
        "kubeadm_token" => kubeadm_token,
      }
    end
  end

  (1..N_MINIONS).each do |n|
    host_name = "kube-minion%d" % n
    config.vm.define host_name do |node|
      node.vm.hostname = host_name
      node.vm.network :private_network, ip: "172.28.128.%s" % (100 + n)

      node.vm.provision :ansible_local do |ansible|
        ansible.playbook = "kubeminion.yml"
        ansible.become = true
        ansible.extra_vars = {
          "host_ip" => "172.28.128.%s" % (100 + n),
          "cluster_cidr" => "172.28.128.0/24",
          "kube_master_ip" => "172.28.128.100",
          "kubeadm_token" => kubeadm_token,
        }
      end
    end
  end
end
