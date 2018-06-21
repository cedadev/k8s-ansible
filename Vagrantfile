# -*- mode: ruby -*-
# vi: set ft=ruby :

N_NODES = 2


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

  config.vm.define "kube-proxy" do |proxy|
    proxy.vm.hostname = "kube-proxy"
    proxy.vm.network "private_network", ip: "172.28.128.100"

    proxy.vm.provider :virtualbox do |vb|
      vb.cpus = 1
      vb.memory = 512
    end
  end

  (0..N_NODES-1).each do |n|
    node_name = "kube-node%d" % n
    node_ip = "172.28.128.%s" % (101 + n)
    config.vm.define node_name do |node|
      node.vm.hostname = node_name
      node.vm.network "private_network", ip: node_ip

      node.vm.provider :virtualbox do |vb|
        vb.cpus = 2
        vb.memory = 3072
      end

      # Make sure Kubernetes service traffic is routed over eth1
      # This is necessary only because cluster traffic should not use the default route
      node.vm.provision :shell,
        run: "always",
        inline: "ip route add 10.96.0.0/12 via %s dev eth1" % node_ip

      if n == (N_NODES-1)
        # On the final node (i.e. when all the machines in the cluster have started)
        # we run the playbook
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "cluster.yml"
          # ansible.verbose = "vvvv"
          ansible.limit = "all"
          ansible.force_remote_user = false
          ansible.groups = {
            "proxy_hosts" => ["kube-proxy"],
            "kube_masters"  => ["kube-node0"],
            "kube_nodes" => (0..N_NODES-1).map { |n| "kube-node%d" % n },
            "kube_hosts:children" => ["kube_masters", "kube_nodes"],
            "vagrant_hosts:children" => ["proxy_hosts", "kube_hosts"],
            "vagrant_hosts:vars" => [
              "cluster_interface=eth1",
              "kube_master_ip=172.28.128.101",
              "use_userspace_proxy=1",
            ]
          }
        end
      end
    end
  end
end
